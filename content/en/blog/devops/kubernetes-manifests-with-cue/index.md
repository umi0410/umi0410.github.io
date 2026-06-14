---
title: "Stop Struggling With Helm Charts: Author Kubernetes Manifests with CUE"
date: 2026-06-15T00:00:00+09:00
categories:
  - DevOps
  - Kubernetes
tags:
  - helm
  - cue
  - kubernetes
  - configuration
  - platform-engineering
weight: 13
cover:
  relative: true
  hidden: false
searchHidden: false
ShowWordCount: true
---

> Part 1 of a series on managing Kubernetes configuration at scale.
> Upcoming: comparing CUE and Helm for different use cases, and integrating CUE into CI/CD pipelines and GitOps workflows.

---

I have spent the past four years as a DevOps engineer reading, using, and writing Helm charts — from small internal deployment wrappers to large, multi-team platform charts. I have read through the internals of charts for Istio, CockroachDB, ArgoCD, cert-manager, and Vault, and written enough `_helpers.tpl` macros and validations to last a lifetime.

Over that time, I arrived at a firm personal conclusion: Helm tends to work against teams managing Kubernetes manifests internally — and in my experience, it eventually becomes too cumbersome even for the application developers who have to use it. The team I worked with shared that frustration, and I spent considerable time personally researching alternatives: CUE, KCL, PKL, NixOS modules, and Nickel. A colleague even went as far as developing an open-source project called kubix to address some of these problems.

This post is my personal reflection on that search — specifically, how to author Kubernetes manifests using CUE and why I think it handles this better than Helm. In short, **Helm is too limited for expressing proper configuration abstractions, NixOS modules do it well but carry a steep learning curve, and CUE sits at a practical sweet spot between the two.**

---

## What a Kubernetes Manifest Actually Is

Kubernetes is built on a simple idea: you declare the desired state of your system, and the cluster makes it so. In practice, even getting a single Deployment right requires care.

None of these resources exist in isolation — they influence each other. A Deployment's `selector` must match the Pod's `labels`. When using an HPA, the `replicas` field on the Deployment should be omitted entirely — having both causes the HPA's scaling decisions to be continuously overwritten. Configuration requirements also vary by environment: in production you want zero capacity loss during rollouts; in dev you want the fastest possible deployment.

What started as "just some YAML files" rapidly becomes a complex configuration management problem — and the problems compound when you have dozens of services, multiple environments, ephemeral per-feature namespaces, and multi-cluster setups.

---

## Helm: How the Industry Responded

Helm became the de-facto answer to this problem. The core idea: treat your manifests as templates, separate the variable parts into values files, and render the final YAML by substituting values at deploy time.

A Helm chart for the service above typically looks like this. Critically, the chart itself is shared across services — teams customize it by supplying their own values files:

```text
common-component/
  Chart.yaml
  values.yaml          ← chart default values
  templates/
    deployment.yaml
    service.yaml
    _helpers.tpl       ← named template macros
```

Teams deploy by referencing the shared chart and layering their overrides on top:

```bash
helm upgrade user-api ./charts/common-component \
  -f teams/user-team/api-common.yaml \      # team-wide values for the user-api team
  -f teams/user-team/api-production.yaml    # environment-specific overrides
```

And the deployment template renders the final output:

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "common-component.fullname" . }}
  labels:
    {{- include "common-component.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      {{- include "common-component.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "common-component.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          {{- with .Values.resources }}
          resources:
            {{- toYaml . | nindent 12 }}
          {{- end }}
```

Technically, this works. But as organizations and their services grow, the cracks start to show.

---

## The Pain Points

### 1. String Templating Turns YAML Into a Programmatic Mess

Helm templates are not YAML. They are merely Go templates that happen to produce YAML when rendered.

Properly indenting a multi-line value requires `nindent`. Rendering a dictionary as YAML requires `toYaml`. Handling optional fields requires `{{- with }}` or `{{- if }}` blocks. And crucially, the output is not a plain string — it needs to be formatted, syntactically valid YAML at every indentation level, produced by assembling template fragments that each need to be correctly indented relative to their context. The more conditional your chart, the more intractable this becomes:

```yaml
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicas }}
  {{- end }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          {{- if .Values.env }}
          env:
            {{- range .Values.env }}
            - name: {{ .name }}
              value: {{ .value | quote }}
            {{- end }}
          {{- end }}
          {{- if .Values.envFrom }}
          envFrom:
            {{- toYaml .Values.envFrom | nindent 12 }}
          {{- end }}
```

This is not simple text substitution. It contains programmatic logic — branching, iteration, function calls — embedded in a format that was not designed for it. **The only way to know what the rendered YAML will look like is to run `helm template` and read the output.**

### 2. Leaky Abstractions

Every knob that a user might ever need must be explicitly surfaced in `values.yaml`. This creates a fundamental dilemma for chart authors:

- **Expose everything**: `values.yaml` grows to hundreds of fields that largely replicate the Kubernetes API. At this point, the chart is effectively the same as writing raw Kubernetes manifests — just with an extra layer of indirection on top.
- **Expose only the common cases**: Users who need an uncommon field are stuck. Their options are:
  1. Fork the chart, modify it, set up their own CI, and maintain their own Helm repository.
  2. Use `helm template` and post-process the rendered output — an ad-hoc, fragile approach that breaks compatibility with many GitOps tools.
  3. Write a new chart from scratch.

> **Note on post-rendering:** Helm supports a `--post-renderer` hook that pipes rendered output through an external tool. While technically an escape hatch, it is widely considered fragile, difficult to test, and incompatible with much of the GitOps ecosystem. Its presence in a workflow is generally a signal that the chart's abstraction has already broken down.

### 3. Dynamic Defaults Cannot Be Expressed

In `values.yaml`, every default is static. Consider `replicas`: its sensible default depends on `env` — 1 in dev, 3 in prod. You can bury this logic in a `_helpers.tpl` named template, but that makes the behavior invisible to anyone editing `values.yaml`. Any downstream field that also depends on `replicas` must invoke its own named template in turn. The chain of dependencies is hidden and scattered. **There is no way to express relationships between values in the format itself — every workaround papers over a fundamental limitation.**

### 4. The Golden Path Chart Problem

Platform and DevOps teams often want a single canonical chart that encodes the organization's hard-won operational knowledge — a golden path that teams can adopt without making every infrastructure decision from scratch. This might include internally agreed-upon labels and annotations, `preStop` lifecycle hooks for graceful shutdown, HPA defaults, resource limits, and so on.

On top of the org-level golden path, individual teams should also be able to define their own layer: a chart or module that encodes team-specific conventions and is shared across all their services.

And there is another dimension that makes this harder: teams rarely deploy a single component. A service might consist of a stateless API server, a stateful stream server, a worker and a backoffice component — these distinct Deployments with different replica counts, different resource profiles, and different scaling policies, but sharing most parts of their configuration. Being able to define all of them together as a single unit is a natural and common need.

In Helm, trying to satisfy all of this means the default `values.yaml` of the common chart ends up looking like this:

```yaml
# values.yaml (excerpt from a real in-house golden-path chart)
common:
  env: ""             # dev | stg | prod
  team: ""

deployment:
  replicas: 1
  image:
    repository: ""
    tag: ""
  lifecycle:
    preStop:
      enabled: true      # "required by convention" — but trivially disabled
      sleepSeconds: 5
  topologySpread:
    enabled: true
    maxSkew: 1
  sidecars: []
  extraEnv: []
  extraVolumes: []
  extraVolumeMounts: []

hpa:
  enabled: false
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
# ... and 50 more fields
```

For multi-component services, there are two common approaches:

1. **Parent chart with sub-charts**: Define one golden-path chart per component, then wire them together under a parent chart. The `values.yaml` for the parent ends up with enormous duplication — every shared value must be declared separately for each sub-chart.

2. **Loop-based golden path**: Make the golden path chart itself capable of rendering multiple components by iterating over a list. This requires wrapping `Deployment` and `Service` definitions inside `range` blocks and `define` macros, producing templates that are verbose and difficult to read.

Either way, the template grows to hundreds of lines, `_helpers.tpl` accumulates deeply nested macros.

---

## The Search for Alternatives

**Kustomize** is simple, but provides too little templating flexibility — teams frequently end up reaching for Helm anyway to handle the cases Kustomize can't express. **Jsonnet** and **Dhall** are powerful, but impose significant cognitive overhead for what is ultimately a configuration task; both carry reputations in the Kubernetes community for being painful to work with day to day.

My real focus was on tools designed specifically for configuration at the organizational level — each with a rich type system and composable abstractions: **CUE**, **KCL**, **PKL**, **NixOS modules**, and **Nickel**. All of them address the core problem in meaningfully different ways.

- **KCL**: Natively integrated with ArgoCD and FluxCD, which lowers the bar for adoption in GitOps workflows. However, it lacks fixed-point evaluation, and writing it tends to feel like writing Python — which adds cognitive overhead for a task that is fundamentally about configuration values and constraints.
- **PKL** (Apple): A capable general-purpose configuration language, but it is oriented toward general config rather than Kubernetes manifest authoring specifically. It lacks CUE's constraint and unification model.
- **Nickel**: Borrows Nix's syntax aesthetics but cannot leverage NixOS module's composability guarantees. You inherit the unfamiliar syntax without the main benefit.
- **NixOS modules**: Principled, with a very rich type system and strong guarantees. But the learning curve is steep enough that broad adoption within a mixed-experience team is difficult.

CUE sits at a practical sweet spot: enough expressiveness to model organizational policy correctly, with a learning curve that is manageable for most engineers.

---

## What Is CUE?

CUE (Configure, Unify, Execute) is a language for configuration and data validation. For a thorough introduction to the language — its type system, constraints, comprehensions, and evaluation semantics — the [official CUE documentation](https://cuelang.org/docs/) is the right starting point; this post focuses on practical motivations and patterns rather than language internals.

**Generating the final YAML is just one of CUE's features.** CUE does not produce YAML by substituting values into text strings. Instead, configuration files are always structured values — syntactically valid, schema-checkable, and composable. There are no indentation bugs, no quoting errors, and no structural problems that only surface at render time.

**CUE is also deliberately scoped — it is not a general-purpose language, and the boundaries this places on what you can express are intentional.** There is no mutable state, no side effects, no I/O. CUE has comprehensions for iterating over lists and structs, but there is no imperative control flow. A CUE file is a description of values and their constraints. Evaluation is referentially transparent — given the same inputs, you always get the same output, in the same way a pure function does. The learning curve is far more manageable than a general-purpose language; reading CUE is closer to reading YAML than to reading code.

What CUE has is **unification** and **fixed-point evaluation** — and these two properties together solve most of the problems above.

**Unification** is CUE's underlying evaluation model: all definitions of the same value within a package are automatically merged into a single consistent result. This has a practical implication that is the direct opposite of Helm's leaky abstraction problem. In Helm, a chart must explicitly expose every field a user might want to customize — otherwise there is no way to set it. In CUE, a team can extend any part of the generated output simply by defining it alongside their config. The golden path does not need to anticipate every possible field:

```cue
// apps/user-api/config.cue

_manifests: goldenpath.#Manifests & {
    appConfig: { name: "user-api", env: "dev", ... }
}

// Adding trafficDistribution to the Service — a field not exposed by the golden path.
// No "hole" needed — CUE unifies this with the generated service automatically.
_manifests: services: "user-api": spec: trafficDistribution: "PreferClose"
```

**Fixed-point evaluation** means all constraints in the model are evaluated simultaneously, regardless of declaration order or nesting level. A field deep inside a Deployment spec can reference a field from the top-level app config without any explicit threading, function call, or helper macro. CUE evaluates all constraints together and finds the single consistent result — or tells you exactly why no consistent result exists.

Together, these give you what org-level configuration actually needs: closed enums, value ranges, derived fields, composable schemas — without the overhead of a full programming language.

### Enums and Range Constraints

Unlike rolling update bounds or replica counts (which the Kubernetes API validates anyway), CUE lets you define *org-specific* constraints that Kubernetes knows nothing about:

```cue
#AppConfig: {
    // Only valid environments in our org.
    // "staging", "development", or any typo fails immediately.
    env: "dev" | "stg" | "prod"

    // Replicas must be in a sane range for our infrastructure.
    replicas?: int & >=0 & <=50
}
```

**These are not comments or documentation — CUE enforces them.** Passing `env: "staging"` or `replicas: 200` fails immediately with a clear error pointing to the constraint definition.

When the set of valid values is constrained to a closed enumeration — three possible environments, replicas bounded to a sane range — all the code that consumes those values becomes simpler. Fewer branches, fewer edge cases, less defensive logic. Narrowing the problem space at the boundary eliminates entire categories of bugs downstream.

### Defaults with Chained Dependencies

In Helm, a default that depends on another field requires a named template in `_helpers.tpl`, invoked explicitly wherever that value is used. Any downstream field that also depends on it must invoke its own named template — and so on. The chain of dependencies becomes buried and scattered.

In CUE, all constraints — including defaults — unify into a single consistent model. The `*value | type` syntax marks a default: `*2` is the default, `| (int & >=1)` means any valid integer is still accepted:

```cue
env: "dev" | "stg" | "prod"

replicas: *1 | (int & >=0)

autoScaling: {
    enable:      bool       // no global default — set per env below to avoid conflict
    minReplicas: int & >=1
    maxReplicas: *(minReplicas * 50) | (int & >=1)  // derives from sibling field
}

if env == "dev" {
    autoScaling: enable:      *false | bool
    autoScaling: minReplicas: *1 | (int & >=1)
}
if env == "stg" {
    autoScaling: enable:      *true | bool
    autoScaling: minReplicas: *2 | (int & >=1)
}
if env == "prod" {
    autoScaling: enable:      *true | bool
    autoScaling: minReplicas: *3 | (int & >=1)
}
```

`enable` and `minReplicas` have no global default — a global `*false` would conflict with the env-specific `*true` during unification. `maxReplicas` derives automatically from whichever `minReplicas` is active.

With `env: "stg"`, CUE resolves all constraints to their concrete defaults:

```text
$ cue eval -c .
env:      "stg"
replicas: 1
autoScaling: {
    enable:      true
    minReplicas: 2
    maxReplicas: 100
}
```

*A default is a preferred value within a constraint — not a hardcoded assignment.* The team can override `minReplicas: 1` and set `maxReplicas: 20` explicitly; CUE picks both through unification with no conflict.

---

## Authoring Manifests with CUE: Step by Step

### Setup

```bash
mkdir common-service && cd common-service
cue mod init my-org.dev/common-service

# Import Kubernetes API types as CUE definitions.
# This ensures our schemas always match the real Kubernetes API.
cue get go k8s.io/api/apps/v1
cue get go k8s.io/api/autoscaling/v2
cue get go k8s.io/api/core/v1
```

> **Note:** `cue get go` requires the target packages to be available as Go modules. Run `go get k8s.io/api` and `go mod tidy` in the module root before running these commands if you haven't already.

The directory structure we will build:

```text
common-service/
  cue.mod/
    module.cue
    pkg/                  ← generated k8s type definitions (from cue get go)
  golden-path/
    schema.cue            ← org-level schemas: #ComponentConfig, #AppConfig
    manifests.cue         ← derives k8s manifests from #AppConfig
  apps/
    user/
      common.cue          ← user app: component defaults (all environments)
      production.cue      ← production environment overrides
```

### The Org Schema: What Teams Fill In

Rather than defining Kubernetes resource schemas by hand, we import the authoritative definitions generated from the Kubernetes API itself. What we define is the *org-level abstraction*: the schema for a single deployable component (`#ComponentConfig`), and the schema for an entire application that may consist of multiple components (`#AppConfig`).

```cue
// golden-path/schema.cue
package goldenpath

// #ComponentConfig is the config for a single deployable component.
// env is propagated automatically from the parent #AppConfig.
#ComponentConfig: {
    enabled: *true | bool
    env:     "dev" | "stg" | "prod"
    image:   string

    replicas: *1 | (int & >=0)

    autoScaling: {
        enable:      bool       // no global default — set per env below
        minReplicas: int & >=1
        maxReplicas: *(minReplicas * 50) | (int & >=1)
    }

    if env == "dev" {
        autoScaling: enable:      *false | bool
        autoScaling: minReplicas: *1 | (int & >=1)
    }
    if env == "stg" {
        autoScaling: enable:      *true | bool
        autoScaling: minReplicas: *2 | (int & >=1)
    }
    if env == "prod" {
        autoScaling: enable:      *true | bool
        autoScaling: minReplicas: *3 | (int & >=1)
    }
}

// #AppConfig is the top-level config for an entire application.
// An application may consist of multiple components (api, worker, etc.).
#AppConfig: {
    name:      string
    namespace: string
    team:      string
    env:       "dev" | "stg" | "prod"

    // All components inherit env from the parent AppConfig.
    components: [string]: #ComponentConfig & { env: env }
}

#Labels: {[string]: string | number | bool}
```

In CUE, the `*` default marker must appear inside a disjunction — `*false` alone is not valid syntax. The `| bool` (or `| (int & >=1)`) after it is the type constraint that keeps the field open for team overrides. Without it, the default would also be the only accepted value. The env-level if-block then unifies with the outer type declared in the base struct, so no information is lost.

### The Golden Path: Deriving Manifests from Config

```cue
// golden-path/manifests.cue
package goldenpath

import (
    appsv1        "k8s.io/api/apps/v1"
    autoscalingv2 "k8s.io/api/autoscaling/v2"
)

// #Manifests derives all org-standard Kubernetes resources from an #AppConfig.
#Manifests: {
    appConfig: #AppConfig

    // One Deployment per enabled component.
    // replicas is omitted when HPA is active — the HPA manages it directly.
    deployments: {
        for name, componentConfig in appConfig.components if componentConfig.enabled {
            "\(name)": appsv1.#Deployment & {
                metadata: {
                    name:      name
                    namespace: appConfig.namespace
                    labels: {
                        "app.kubernetes.io/name":       name
                        "app.kubernetes.io/managed-by": "golden-path"
                        "org/team":                     appConfig.team
                        "org/env":                      componentConfig.env
                    }
                    annotations: {
                        "org/datadog-dashboard": "https://app.datadoghq.com/dashboard/\(name)"
                        "org/oncall-slack":      "@\(appConfig.team)-oncall"
                    }
                }
                spec: {
                    selector: matchLabels: "app.kubernetes.io/name": name
                    template: {
                        metadata: labels: "app.kubernetes.io/name": name
                        spec: {
                            containers: [{
                                name:  name
                                image: componentConfig.image
                                lifecycle: preStop: exec: command: ["/bin/sh", "-c", "sleep 5"]
                            }]
                        }
                    }
                    if !componentConfig.autoScaling.enable {
                        replicas: componentConfig.replicas
                    }
                }
            }
        }
    }

    // One HPA per enabled, autoscaling component.
    hpas: {
        for name, componentConfig in appConfig.components
            if componentConfig.enabled && componentConfig.autoScaling.enable {
            "\(name)": autoscalingv2.#HorizontalPodAutoscaler & {
                metadata: {
                    name:      name
                    namespace: appConfig.namespace
                }
                spec: {
                    scaleTargetRef: {
                        apiVersion: "apps/v1"
                        kind:       "Deployment"
                        name:       name
                    }
                    minReplicas: *componentConfig.autoScaling.minReplicas | (int & >=1)
                    maxReplicas: *componentConfig.autoScaling.maxReplicas | (int & >=1)
                }
            }
        }
    }

    // Flat list of all manifests — ready for kubectl apply
    manifests: [
        for _, d in deployments { d },
        for _, h in hpas { h },
    ]
}
```

A few things worth noting:

- **`components: [string]: #ComponentConfig`** — `#Manifests` iterates over all components automatically. Adding a new component requires no changes to `#Manifests`.
- **`replicas` is omitted from Deployments when HPA is enabled** — the HPA manages it directly.
- **Teams can add fields freely.** Any field not exposed by the golden path can be defined alongside the component config — CUE unifies both automatically. No fork, no post-processing.

### A Team's Config

`apps/user/common.cue` defines cross-environment defaults for all components of the user service:

```cue
// apps/user/common.cue
package user

import "my-org.dev/common-service/golden-path"

_app: goldenpath.#Manifests & {
    appConfig: {
        name: "user"
        team: "user-team"
        // namespace and env are set per-environment (see production.cue)
    }
}

// user-api: stateless HTTP API server
_app: appConfig: components: "user-api": {
    image: "user-api:latest"
}

// user-worker: background job processor — workers scale differently from API servers
_app: appConfig: components: "user-worker": {
    image:               "user-worker:latest"
    autoScaling: enable: false
}
```

`apps/user/production.cue` adds the concrete production values — CUE unifies both files automatically:

```cue
// apps/user/production.cue
package user

_app: appConfig: {
    namespace: "user-production"
    env:       "prod"
}

_app: appConfig: components: "user-api": {
    image: "user-api:2.1.0"
    // env: "prod" propagates from AppConfig →
    //   autoScaling defaults to enabled (minReplicas: 3, maxReplicas: 150)
}

_app: appConfig: components: "user-worker": {
    image:    "user-worker:2.1.0"
    replicas: 5  // static scaling — autoscaling remains disabled
}

// manifests is defined by #Manifests — no need to write the comprehension here
manifests: _app.manifests
```

For production, the result is:

- **`user-api`**: Deployment (replicas omitted), HPA (`minReplicas: 3, maxReplicas: 150`)
- **`user-worker`**: Deployment (`replicas: 5`), no HPA

### Generating and Applying the Manifests

```bash
# Validate — no output means everything is consistent
cue vet ./apps/user/...

# Export and apply directly
cue export ./apps/user/ --out yaml -e manifests | kubectl apply -f -
```

CUE has no runtime dependency — it is a compile step. The output is plain Kubernetes YAML.

### What Happens When a Policy Is Violated?

```cue
// apps/user/production.cue
_app: appConfig: components: "user-api": {
    env: "staging"   // not in the defined enum: "dev" | "stg" | "prod"
}
```

```text
$ cue vet ./apps/user/...
_app.config.components["user-api"].env: 3 errors in empty disjunction:
    _app.config.components["user-api"].env: conflicting values "dev" and "staging":
        ./golden-path/schema.cue:5:10
        ./apps/user/production.cue:3:10
```

The error names the constraint and the violation site. No cluster interaction, no `helm template` pass, no cryptic API server rejection.

---

## Conclusion

Helm is the right tool for what it was designed for: packaging and distributing software for others to install. If you are publishing an open-source chart, Helm's release management and layered values model are the right foundation.

But for internal platform teams maintaining their own manifests, Helm's model accumulates friction over time. The templates grow verbose and unsafe — a misplaced `nindent` or an unquoted value is invisible until render time. Conventions encoded in `_helpers.tpl` are trivially bypassed. The golden path you spent weeks crafting leaks: teams fork the chart, post-process the output, or simply give up and write raw YAML. And when a team wants to wrap the common chart in their own layer of abstractions — encoding team-specific conventions across multiple components — Helm's tools make the work disproportionately painful.

If you have found yourself `cp -r`-ing a chart into a new directory just to make a small tweak, or spending an afternoon tracing through a 500-line `values.yaml` to understand why a field has the wrong value, these are not skill problems. They are model problems.

CUE addresses them at the root: a rich type system for catching misconfiguration before it reaches a cluster, fixed-point evaluation for expressing the relationships between fields that Helm cannot, and composable modules for building genuine layered abstractions. The result is manifests that are concise, validated, and correct by construction.

If Helm is causing you pain, CUE is worth a serious look.

---

**Next in this series:** comparing CUE and Helm for different use cases, and integrating CUE into CI/CD pipelines and GitOps workflows.
