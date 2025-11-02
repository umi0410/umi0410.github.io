---
title: 'Istio Service Entry: Improved observability about HTTPS requests to external services'
date: 2023-07-08T23:00:00+09:00
weight: 15
categories:
  - Istio
  - DevOps
cover:
  image: preview.png
---

## Introduction

During developing services, there are some cases we need to send HTTPS requests to external services. 

When using Istio, requests based on the hosts that are not registered in `Service registry`
are essentially recognized as a `Cluster` named `Passthrough`, which just operates solely as a TCP proxy.
That is, Envoy simply forwards those TCP packets without performing any additional
functionality on them.
In this case, for requests based on HTTPS which operates at the L7 layer,
Istio is only able to record TCP-level metrics, resulting in limited information available for TCP metrics.

> - _`Service registry` - A registry collecting information about the Services which Service Mesh is aware of. (Reference: [Istio Official Documentations](https://istio.io/latest/docs/concepts/traffic-management/))_
> 
> - _`Passthrough` - A special Cluster used when it is unabled to identify the service based on the service registry.
> Used when the egress mode of Sidecar API is set to `ALLOW_ANY`.
> When the egress mode is `REGISTRY_ONLY`, `Blackhole` Cluster is used instead of `Passthrough` Cluster. (Reference: [Istio Official Blog](https://istio.io/latest/blog/2019/monitoring-external-service-traffic/))_
> - _`Cluster` - While Cluster has diverse meaning, in the context of Istio or Envoy, it primarily refers to a logical group of Endpoints._ 

When sending requests to external services, by utilizing the ServiceEntry feature of Istio and registering the external services in our service registry,
it is possible to prevent them from being identified as the Passthrough Cluster.

However, if requests from application containers to external services are transmitted over HTTPS,
the egress traffic is encrypted by the application containers before passing through Envoy.
As a result, Envoy is unable to decrypt those packets.

Threfore, the observability based on Istio regarding the requests between application containers and external services, when using HTTPS, is reduced.  

In this article, the aim is to address the diminished observability when sending requests to external services over HTTPS by leveraging Istio's `ServiceEntry` and `DestinationRule`.

> - _`ServiceEntry` - A kind of Istio's Custom Resources. It defines entities that are not k8s Services as Services and registers them in the service registry._
> 
> - _`DestinationRule` - A kind of Istio's Custom Resources. It defines the policies to be applied after routing occurs._

After applying ServiceEntry and DestinationRule, application containers send requests over HTTP and Envoy
takes care of TLS origination for the egress traffic.

By doing so, We can maintain the security advantages of HTTPS while allowing Envoy to better understand the packets exchanged between
application containers and external services, resulting in improved observability.

## Demo Environment

If you haven't had the experience of sending requests to external services over HTTPS and monitoring those requests by using Service mesh
You many not fully comprehend the signifcance of the statement and the limited observability it entails.

Let's try a simple demo in order to check the problem, solution and improved result more clearly 
To check the problem, solutio and improved result more clearly, I conducted a simple demo.

For the demo, I have set up a simple K8s cluster with Istio, Prometheus and Kiali installed on my local machine. 

| Name        | Version | Description                                                                                                                             |
|-------------|---------|-----------------------------------------------------------------------------------------------------------------------------------------|
| K8s cluster | 1.23.3  | Installed by using minikube                                                                                                             |
| Istio       | 1.18.0  | Installed simply according to the [official Istio documentation's Helm-based Istio installation guide](https://istio.io/v1.18/docs/setup/install/helm/).<br />Installed without specifying a revision, using the default revision. 
| Prometheus  | 2.41.0  | Installed based on the [official Istio documentation's Prometheus installation guide](https://istio.io/v1.18/docs/ops/integrations/prometheus/).                                 
| Kiali       | 1.67    | Installed based on the [official Istio documentation's Kiali installation guide](https://istio.io/v1.18/docs/ops/integrations/kiali/).                                      

During the demo, I'll be using a K8s Namespace named `test-ext-svc`.
Several K8s resources will be deployed in this Namespace.
The pods in this Namespace have been configured to automatically inject the `istio-proxy` sidecar container on creation.

## Demo 1. Exploring Observability of requests to External Services over HTTPS

Let's examine why the observability is diminished when sending requests to external service over HTTPS.

The external service used for for testing in this article is `https://httpbin.org`.  
Let's create a Deployment that sends requests to the external service over HTTPS.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: curl
  namespace: test-ext-svc
  labels:
    app: curl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: curl
  template:
    metadata:
      labels:
        app: curl
    spec:
      containers:
      - name: curl
        image: curlimages/curl
        command:
        - sh
        - -c
        - |
          while true; do
            curl -vs https://httpbin.org/status/200
            sleep .2
          done;
      terminationGracePeriodSeconds: 0
```

Inside of the created Pod, requests are being sent towards [https://httpbin.org/status/200](https://httpbin.org/status/200).
However, when querying the Istio metrics of the Pod, it can be observed that the standard L7 metrics(Reference: [the official documentation](https://istio.io/latest/docs/reference/config/metrics/)))
such as `istio_requests_total`, `istio_request_duration_milliseconds` are not generated.

As mentioned eariler, only TCP-related metrics are being generated, and upon closer inspection of the labels, it can be observed that
most of label values are set to `unknown`.
The Istio metrics of a specific Pod can be observed the following command. 

```bash
kubectl exec -n test-ext-svc \
  $(kubectl get pod -n test-ext-svc -o name) -- \
  curl -s localhost:15000/stats/prometheus | grep istio_
```

![](tcp-unknown.png)

That is, **the observailty is so poor.**

Indeed, it is clear that L7 metrics related to requests towards external services are being not generated.
Therefore, when querying the L7 Istio metrics on Prometheus, no information is available regarding L7 metrics. 

```promql
sum(
  rate(
    istio_requests_total{reporter="source", source_workload_namespace="test-ext-svc"}[1m]
  )
) by (source_workload, destination_service_name)
```

![](prom-l7-metric.png)

Although there are metrics avaiable for TCP(L4) communication, the detailed information can't be observed due to the Cluster being identified as the Passthrough Cluster.

```promql
sum(
  rate(
    istio_tcp_connections_closed_total{reporter="source", source_workload_namespace="test-ext-svc"}[1m]
  )
) by (source_workload, destination_service_name)
```

![](prom-l4-metric.png)

If there are multiple external services, it would be challenging to differentiate them as they would all be identified as the same Passthrough Cluster. 

The external services are also identified as the Passthrough Cluster on Kiali, and as a result, it becomes impossible to observe the 
destination of the requests and obtain information about the requests, such as status codes.

![](kiali-passthrough.png)

## Demo 2. Making it Identified as a Specific Cluster Instead of the Passthrough Cluster by Adding ServiceEntry

Let's register `httpbin.org` in our service registry by using `ServiveEntry`, which is a type of Istio's Custom Resource.
Then, it would be recognized as a unique Cluster named `httpbin.org` instead of treated as the Passthrough Cluster

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: httpbin
  namespace: test-ext-svc
spec:
  hosts:
    - httpbin.org
  location: MESH_EXTERNAL
  ports:
    - number: 80
      name: http
      protocol: HTTP
    - number: 443
      name: https
      protocol: HTTPS
  resolution: DNS
```

To make the `httpbin.org` host identified as the Cluster named `httpbin.org` instead of the Passthrough Cluster, we can register `httpbin.org` in our service registry.


Requests to the port defined for the HTTP protocol, based on the Host header, and requests to the port defined for the HTTPS protocol, based on the SNI,
will be recognized as requests to the httpbin.org cluster.
The destination for these requests will be the IP address obtained through DNS resolution.

However, it is important to note that although the requests to the httpbin.org cluster can be recognized based on the SNI,
the L7 packets transmitted between the application container and the external service are still encrypted,
and Envoy cannot decrypt them. Therefore, full visibility into the L7 layer cannot be achieved.

Nevertheless, we can observe that in the L7 metrics, the service name is now recognized,
and the requests are no longer treated as the Passthrough Cluster but are processed as the httpbin.org cluster.
This indicates progress towards better identification and routing of the requests.

![](prom-l4-serviceentry.png)

![](kiali-serviceentry.png)

Certainly, information about the httpbin.org cluster can be obtained by querying the proxy config.
By inspecting the proxy configuration, you can verify the settings and configurations related to the httpbin.org cluster

```bash
$ istioctl pc cluster -n test-ext-svc $(kubectl get pod -n test-ext-svc -o name)
SERVICE FQDN                                      PORT      SUBSET     DIRECTION     TYPE             DESTINATION RULE
BlackHoleCluster                                  -         -          -             STATIC
InboundPassthroughClusterIpv4                     -         -          -             ORIGINAL_DST
PassthroughCluster                                -         -          -             ORIGINAL_DST
agent                                             -         -          -             STATIC
httpbin.org                                       80        -          outbound      STRICT_DNS
httpbin.org                                       443       -          outbound      STRICT_DNS
...
```

The endpoints for the respective cluster are obtained by Envoy periodically performing DNS lookups and obtaining the corresponding IP addresses.

```bash
$ istioctl pc endpoint -n test-ext-svc $(kubectl get pod -n test-ext-svc -o name)
ENDPOINT                                                STATUS      OUTLIER CHECK     CLUSTER
...
54.204.94.184:80                                        HEALTHY     OK                outbound|80||httpbin.org
54.204.94.184:443                                       HEALTHY     OK                outbound|443||httpbin.org
54.210.149.139:80                                       HEALTHY     OK                outbound|80||httpbin.org
54.210.149.139:443                                      HEALTHY     OK                outbound|443||httpbin.org
```

## Demo 3: Ensuring L7 Observability While Maintaining Making Requests over HTTPS

If you want to achieve L7 observability while making requests over HTTPS to external services,
you need to ensure that TLS Origination for egress traffic is performed by Envoy instead of the application.

To achieve this, you need to make some modifications to the previously deployed `ServiceEntry` and
apply TLS-related settings through `DestinationRule`. The overall flow can be summarized as follows:

1. The application container sends HTTP requests to the external service in plaintext (e.g., http://httpbin.org:80).
2. The **Istio Sidecar** is responsible for performing server-side TLS communication for egress requests to <Cluster:80>.
3. The Istio Sidecar modifies the Host and Port from `<Cluster>:80` to `<Endpoint>:443` and sends the actual request, following the planned TLS communication from step 2.
4. As a result, the sidecar performs the request to https://httpbin.org:443.

To achieve this, you can add the `targetPort: 443` setting to the ServiceEntry, ensuring that the 80 port of the ServiceEntry is connected to the 443 port of the actual Endpoint.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: httpbin
  namespace: test-ext-svc
spec:
  hosts:
    - httpbin.org
  location: MESH_EXTERNAL
  ports:
    - number: 80
      # The `targetPort` configuration has been added.
      targetPort: 443
      name: http
      protocol: HTTP
    - number: 443
      name: https
      protocol: HTTPS
  resolution: DNS
```

Configure TLS for the 80 port (443 port on the actual Endpoint) of the `httpbin.org` cluster using DestinationRule.

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: httpbin
  namespace: test-ext-svc
spec:
  host: "httpbin.org"
  trafficPolicy:
    portLevelSettings:
      - port:
          number: 80
        tls:
          mode: SIMPLE
```

Finally, the Pod that used to send requests to the external service (httpbin) has been modified to send requests using HTTP instead of HTTPS.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: curl
  namespace: test-ext-svc
  labels:
    app: curl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: curl
  template:
    metadata:
      labels:
        app: curl
    spec:
      containers:
      - name: curl
        image: curlimages/curl
        command:
        - sh
        - -c
        - |
          while true; do
            # The protocol here is updated from `https` to `http`
            curl -vs http://httpbin.org/status/200
            sleep .2
          done;
      terminationGracePeriodSeconds: 0
```

Now, you can verify that the L7 metrics are being generated correctly as follows.

```bash
# Unlike before, istio_requests_total, which is one of the L7 metricss,
# is now being successfully retrieved.
$ kubectl exec -n test-ext-svc $(kubectl get pod -n test-ext-svc -o name) -- curl -s localhost:15000/stats/prometheus | grep istio_requests_total
# TYPE istio_requests_total counter
istio_requests_total{reporter="source",source_workload="curl",source_canonical_service="curl",source_canonical_revision="latest",source_workload_namespace="test-ext-svc",source_principal="unknown",source_app="curl",source_version="",source_cluster="Kubernetes",destination_workload="unknown",destination_workload_namespace="unknown",destination_principal="unknown",destination_app="unknown",destination_version="unknown",destination_service="httpbin.org",destination_canonical_service="unknown",destination_canonical_revision="latest",destination_service_name="httpbin.org",destination_service_namespace="unknown",destination_cluster="unknown",request_protocol="http",response_code="200",grpc_response_status="",response_flags="-",connection_security_policy="unknown"} 54
istio_requests_total{reporter="source",source_workload="curl",source_canonical_service="curl",source_canonical_revision="latest",source_workload_namespace="test-ext-svc",source_principal="unknown",source_app="curl",source_version="",source_cluster="Kubernetes",destination_workload="unknown",destination_workload_namespace="unknown",destination_principal="unknown",destination_app="unknown",destination_version="unknown",destination_service="httpbin.org",destination_canonical_service="unknown",destination_canonical_revision="latest",destination_service_name="httpbin.org",destination_service_namespace="unknown",destination_cluster="unknown",request_protocol="http",response_code="400",grpc_response_status="",response_flags="-",connection_security_policy="unknown"} 106
istio_requests_total{reporter="source",source_workload="curl",source_canonical_service="curl",source_canonical_revision="latest",source_workload_namespace="test-ext-svc",source_principal="unknown",source_app="curl",source_version="",source_cluster="Kubernetes",destination_workload="unknown",destination_workload_namespace="unknown",destination_principal="unknown",destination_app="unknown",destination_version="unknown",destination_service="httpbin.org",destination_canonical_service="unknown",destination_canonical_revision="latest",destination_service_name="httpbin.org",destination_service_namespace="unknown",destination_cluster="unknown",request_protocol="http",response_code="504",grpc_response_status="",response_flags="-",connection_security_policy="unknown"} 3
```

```bash
# Unlike before, ther is significant increase
# in the number of metrics that includes `istio_`.
$ kubectl exec -n test-ext-svc $(kubectl get pod -n test-ext-svc -o name) -- curl -s localhost:15000/stats/prometheus | grep istio_ | wc -l
     207
```

On Kiali, you can also observe that the dashboards are generated based on these metrics, providing **significantly improved observability**.

![](kiali-l7.png)

## Appendix 1. Ensuring Visibility with Various External Services

In the previous sections, it was mentioned that without registering multiple external services in ServiceEntry,
they would all be treated as the same Passthrough cluster,
resulting in reduced visibility.
However, by utilizing ServiceEntry, requests to various external services (e.g., Google, Instagram, ...)
can be recognized as separate clusters, ensuring visibility for each of them.

![](kiali-multiple-external-services.png)

By registering external services in the Service Registry through ServiceEntry,
not only does it improve visibility,
but it also enables the utilization of various Istio features for those external services.
For example, through DestinationRule configuration,
mTLS and advanced load balancing can be applied. VirtualService configuration allows for setting up retries and timeouts.

## Appendix 2. Providing Separate Short Names for External Services

I wanted to explore the possibility of assigning separate short names to external services,
especially for cases where the domain of the external service is too long.
For example, instead of using `foo.bar.baz.example.com`, I wanted to use a shorter name like `foo`.

However, this proved to be challenging based on the current capabilities of Istio.
There are several aspects that need to be addressed. Firstly, the need to configure the sidecar to handle DNS queries
instead of relying on kube-dns or core-dns. Additionally, there is an issue with applying DestinationRule to services
defined with short names. Even if DestinationRule is applied, there is a lack of configuration options for manipulating
the Host header and SNI values. These various challenges made it difficult to provide support for assigning separate
short names to external services within Istio's current functionality.

## In Conclusion

Recently, as part of my work tasks, I've been preparing for Istio upgrades at my company.
Through this process, I learned about the new features that is with each version and have taken the time to catch up on some
articles that I had previously missed.

During my exploration, I devled into concepts that I had rarely used, such as  `Sidecar` API, `PeerAuthentication` or `ServiceEntry`
Among these concepts, I found the section on ServiceEntry covered in this article to be particularly  interesting and useful.

If I have an opportunity, personally, I'd like to share my experiences regarding the challenges caused by breaking changes during Istio upgrades and the convenience brought by new features with each version increase. 

## References

- [https://istio.io/latest/blog/2019/monitoring-external-service-traffic/](https://istio.io/latest/blog/2019/monitoring-external-service-traffic/)
- [https://istio.io/latest/docs/tasks/traffic-management/egress/egress-tls-origination/](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-tls-origination/)
- [https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/](https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/)
