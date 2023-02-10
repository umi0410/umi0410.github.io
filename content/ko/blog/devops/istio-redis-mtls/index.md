---
title: 'Istio를 이용해 데이터베이스에 mTLS로 인증하기 - Redis 예시'
date: 2023-02-07T01:00:00+09:00
weight: 18
categories:
  - DevOps
  - Istio
  - Network
image: preview.png
toc: true
TableOfContents: true
---
## 시작하며

인증서 방식의 인증 및 암호화는 보안성이 좋고, 경우에 따라 편리하기까지 하다. 경우에 따라인 이유는 인증서 관리가 수작업으로 하기는 참
번거로운 작업이지만 또한 자동화하기 쉬운 작업이기 때문이다.

Istio와 같은 서비스 메쉬 솔루션을 이용하면 인증서를 통해 손쉽게 TLS 혹은 mTLS 통신이 가능하다. 회사에서 업무를 보다보면
애플리케이션 컨테이너가 데이터베이스를 이용할 때 mTLS를 통해 인증과 암호화를 하기 위해 initContainer에서 인증서 관련해 별도의 작업을
해줘야하는 경우가 있었다. initContainer 뿐만 아니라 volume과 volumeMount 설정, 데이터베이스 접속을 위한 URL 환경변수 등등
다양한 설정이 복잡해지면서 Helm chart를 수정할 때 실수를 범하게 되는 경우가 종종 있었다.

'이런 불편을 서비스 메쉬(Istio)로 좀 더 간편하게 해결할 수는 없을까?!' 싶은 생각이 들어 평소 그냥 살펴보기만 했던 Istio의 `DestinationRule`을
이용한 mTLS 관련 내용을 좀 더 살펴봤다. 아쉽게도 팀에서 사용 중인 데이터베이스인 CockroachDB(PostgreSQL 호환) 현재 지원이 되지 않는 상태이지만 오픈소스 컨트리뷰터들께서
열심히 개발 중인 듯하고 내년쯤이면 사용할 수 있지 않을까 싶다. 아무래도 새로운 버전이 나온다고해서 바로 사용할 수 없는 게 현실이다보니 말이다.

그래서 업무에 올해에는 활용할 수는 없겠지만 CockroachDB가 아닌 다른 데이터베이스로라도 이 기능을 맛본 내용을 정리해보고자 한다.
이번 글에서는 Redis를 이용할 예정이고, v1.16 기준으로 Redis 외에도 Mongo, MySQL 등의 데이터베이스가 사용 가능하다. ([v1.16 문서 참고](https://istio.io/latest/docs/ops/configuration/traffic-management/protocol-selection/))

## 간단하게 원리 살펴보기

L4 TCP 기반의 데이터베이스들이 내부적으로 이용하는 프로토콜을 L7 Application Protocol처럼 사용한다고 볼 수 있다.
envoy의 extension을 통해 트래픽을 해석하고 mTLS 인증, 암호화를 envoy 단에서 수행한다.
아하, 그럼 'MySQL은 사용이 가능한 반면 왜 CockroachDB(PostgreSQL)은 사용이 불가능한가?'가 어느 정도 이해된다. 두 데이터베이스가
TCP 기반으로 각자가 사용하는 프로토콜이 다르기 때문이다.
어쨌든 원리는 이렇다. 서비스메쉬가 알아서 mTLS를 수행하도록 하면 데이터베이스 클라이언트(애플리케이션 서버)는 마치 plain 통신인 것처럼 간단하게
데이터베이스를 이용할 수 있다. 보통은 데이터베이스 서버측의 mTLS까지 Istio로 수행하는 경우는 드물 것 같아 이 경우는 생략하겠다. (레퍼런스도
거의 못 찾아봤던 것 같다.)

그래서 Istio로 이걸 어떻게 정의할 수 있을까? 바로 `DestinationRule`을 가능하다. **DestinationRule의 tls 설정을 통해 client 측 proxy가
어떤 식으로 tls를 이용할 지를 정의할 수 있는데 이때 `mTLS` 관련 설정을 잘 정의해주면 된다.**

## 데모

그럼 간단한 데모를 통해 서비스 메쉬가 어떤 편리함을 가져다주는지 알아보자.
필자의 데모 환경은 다음과 같다.

| Name  | Description                                                                       |
|-------|-----------------------------------------------------------------------------------|
| GKE   | 1.24                                                                              |
| Istio | 1.16.0                                                                            |
| Redis | [Bitnami Helm Chart](https://github.com/bitnami/charts/tree/main/bitnami) v17.7.1 |

수행해보고자하는 건 그냥 `redis-cli`가 사용 가능한 Deployment를 만들어서 redis를 mTLS로 이용해보는 것이다.
그러면서 중간 중간 서비스 메쉬를 이용할 때의 장점에 대해 짚고 넘어가려 한다.

```shell
kubectl create ns istio-redis-mtls
kubens istio-redis-mtls
```

테스트용 Namespace를 만들어줬다.

```yaml
# values.yaml
architecture: standalone
auth:
  enabled: false
```

Redis helm chart를 설치할 때 사용할 values.yaml을 위와 같이 작성해줬다.

* `architecture=standalone` - 그냥 데모용이라 master 한대만인 스탠드얼론 형태로 이용하겠다는 설정 
* `auth.enabled=false` - 패스워드 기반 인증을 안하겠다는 설정

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install foo-redis bitnami/redis \
  --version 17.7.1 -f values.yaml
```

위의 커맨드로 설치해줬다.

```yaml
# redis-client.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-client
  labels:
    app: redis-client
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-client
  template:
    metadata:
      labels:
        app: redis-client
    spec:
      containers:
      - name: redis-client
        image: redis
        command: ['bash', '-c', 'while true; do sleep 3; done;']
      terminationGracePeriodSeconds: 1
```

redis-client를 하나 배포해줬다. 클라이언트는 아직 Istio 사이드카를 inject받지 않는다.

```shell
$ kubectl exec -it $(kubectl get pod -l app=redis-client -o name) -- \
    redis-cli -h foo-redis-master \
    SET foo bar
OK
```

우선은 Plain TCP로 잘 통신이 되는 것을 확인했다.

이제 Redis가 TLS 통신을 하는 서버가 될 수 있도록 인증서를 mount해줄 것이다.
k8s에서는 Cert manger가 아주 유명한 사실상 표준인 인증서 관리/발급 도구이다.
Cert manager 사용법은 생략하고 Self signed CA를 직접 생성해서 인증하는 방법을 적어두기만 하겠다.

```yaml
# cert-manager.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: self-signed-cluster-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: self-signed-ca
spec:
  isCA: true
  commonName: self-signed-ca
  secretName: self-signed-ca-tls
  privateKey:
    algorithm: RSA
  issuerRef:
    name: self-signed-cluster-issuer
    kind: ClusterIssuer
    group: cert-manager.io
  usages:
    - client auth
    - server auth
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: ca-issuer
spec:
  ca:
    secretName: self-signed-ca-tls
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: foo-redis
spec:
  secretName: foo-redis-tls
  duration: 2160h
  renewBefore: 360h
  commonName: foo-redis
  isCA: false
  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048
  usages:
    - client auth
    - server auth
  dnsNames:
    - '*.foo-redis.istio-redis-mtls.svc.cluster.local'
    - foo-redis-master.istio-redis-mtls.svc.cluster.local
    - '*.foo-redis-master.istio-redis-mtls.svc.cluster.local'
    - '*.foo-redis-headless.istio-redis-mtls.svc.cluster.local'
    - foo-redis-headless.istio-redis-mtls.svc.cluster.local
    - 127.0.0.1
    - localhost
    - foo-redis
    - foo-redis-master
    - foo-redis-master.istio-redis-mtls
    - foo-redis-master.istio-redis-mtls.svc.cluster.local
  issuerRef:
    name: ca-issuer
    kind: Issuer
    group: cert-manager.io
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: jinsu
spec:
  secretName: jinsu-tls
  duration: 2160h
  renewBefore: 360h
  commonName: jinsu
  isCA: false
  privateKey:
    algorithm: RSA
    encoding: PKCS1
    size: 2048
  usages:
    - client auth
    - server auth
  issuerRef:
    name: ca-issuer
    kind: Issuer
    group: cert-manager.io
```

다소 긴 Cert manager 커스텀 리소스 관련 매니페스트가 등장하긴 했지만 별로 강조하고 싶은 내용은 아니니 넘어간다. (근데 제대로 설정 안해주면
자꾸 Readiness, Liveness probe가 실패하니 주의해야하긴한다... dnsNames에 저렇게 많은 이름을 적은 이유도 probe 때문이다. probe를 데모 때는 잠깐 빼는 것도 괜찮다.) 
self signed CA를 통해 서버용, 클라이언트용 인증서를 각각 정의했을 뿐이다.

```shell
$ kubectl apply -f cert-manager.yaml
clusterissuer.cert-manager.io/self-signed-cluster-issuer unchanged
certificate.cert-manager.io/self-signed-ca unchanged
issuer.cert-manager.io/ca-issuer created
certificate.cert-manager.io/foo-redis created
certificate.cert-manager.io/jinsu created
```

적용해줬다. 혹시 인증서 발급 방법을 잘 모르겠거나 귀찮다면 그냥 Redis helm chart에서 `tls.autoGenerated` 설정을 이용해
인증서를 자동 발급받는 것도 상관 없다.

```yaml
# values.yaml
architecture: standalone
auth:
  enabled: false
tls:
  enabled: true
  existingSecret: foo-redis-tls
  certFilename: tls.crt
  certKeyFilename: tls.key
  certCAFilename: ca.crt
  authClients: true
```

mTLS를 사용하고자 위와 같이 설정을 해줬다.
`tls.existingSecret`에는 인증서가 담긴 Secret의 이름을 적어주면 되고, `cert*Filename`에는 해당 Secret내의 key를 적어주면 된다.

```shell
helm upgrade foo-redis bitnami/redis \
  --version 17.7.1 -f values.yaml
```

적용해줬다.

```shell
$ kubectl exec -it $(kubectl get pod -l app=redis-client -o name) -- \
    redis-cli -h foo-redis-master --verbose \
    SET foo bar
Error: Connection reset by peer
command terminated with exit code 1
```

이제는 Plain한 통신이 불가능하다. 보안 설정이 엄격해진 것이다!
어떻게 해야 아까 생성해둔 인증서를 통해 mTLS 통신을 할 수 있을까? 일단은 클라이언트 컨테이너가 인증서를 읽을 수 있게 볼륨을 마운트해주든
환경변수를 넣어주든 해야할 것이다. 문제는 그 다음이다.

```shell
$ redis-cli --help
--tls              Establish a secure TLS connection.
--sni <host>       Server name indication for TLS.
--cacert <file>    CA Certificate file to verify with.
--cacertdir <dir>  Directory where trusted CA certificates are stored.
                   If neither cacert nor cacertdir are specified, the default
                   system-wide trusted root certs configuration will apply.
--cert <file>      Client certificate to authenticate with.
--key <file>       Private key file to authenticate with.
...(생략)
```

으윽.. 😵‍💫 **옵션만 봐도 어지러워진다.** 그래도 인증서를 volume으로 mount하고 `--help`가 제시해준 인자들을 잘 적용하면 다음과 같이
mTLS로 통신이 가능하다.

```shell
$ kubectl exec -it  $(kubectl get pod -l app=redis-client -o name) -- \
    redis-cli -h foo-redis-master --sni foo-redis-master --tls \
    --cacert /client-certs/ca.crt \
    --cert /client-certs/tls.crt \
    --key /client-certs/tls.key \
    SET foo bar
OK
```

하지만! **만약 우리가 데모를 벗어나 실제 제품 개발을 할 때는 어떨까?** 지금은 단순 redis-cli 클라이언트였지만 보통은 애플리케이션 서버가 사용하는
데이터베이스 클라이언트 모듈을 설정해줘야할 것이다. 애플리케이션 서버가 Go를 이용하는 경우, JAVA를 이용하는 경우, Node.js를 이용하는 경우 모두
그 클라이언트 모듈의 구현체는 조금씩 다를 수 있고 mTLS 관련 기본값이나 설정 가능한 값이 다를 수 있다. 또한 아예 데이터베이스가 달라질 수도 있다.
이런 경우는 URL이나 인자의 형태 자체가 달라질 수도 있다.

이렇게 **Polyglot하고 다양한 기술이 사용되는 이런 환경에서 서비스 메쉬는 강점을 발휘할 수 있을 것**이다.
클라이언트는 그냥 Plain하게 데이터베이스를 이용하듯 설정하고, 서비스 메쉬가 알아서 mTLS 암호화/인증을 진행할 수 있다.
이렇게 되면 애플리케이션이 어떤 언어를 사용하는지, 어떤 데이터베이스 클라이언트 모듈을 사용하는지에 의존하지 않고
동일하게 동작할 수 있다!

```yaml
# redis-client-istio.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-client-istio
  labels:
    app: redis-cliet-istio
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-client-istio
  template:
    metadata:
      labels:
        app: redis-client-istio
        istio.io/rev: 1-16
      annotations:
        sidecar.istio.io/userVolume: |
          [{
            "name": "client-certs",
            "secret": {
              "secretName": "jinsu-tls",
              "optional": false
            }
          }]
        sidecar.istio.io/userVolumeMount: |
          [{
            "name": "client-certs",
            "mountPath": "/client-certs",
            "readOnly": true
          }]
    spec:
      containers:
      - name: redis-client
        image: redis
        command: ['bash', '-c', 'while true; do sleep 3; done;']
      terminationGracePeriodSeconds: 1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: db-mtls
spec:
  host: foo-redis-master
  workloadSelector:
    matchLabels:
      app: redis-client-istio
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /client-certs/tls.crt
      privateKey: /client-certs/tls.key
      caCertificates: /client-certs/ca.crt
```

인증서는 Sidecar annotation을 통해 istio-proxy 사이드카에 마운트되도록 정의해주고,
trafficPolicy에서 mTLS가 설정된 DestinationRule 정의해줬다. 사실 이것만 보면 '뭐야 이것도 번거로운 거 아니야..?' 싶을 수는 있지만
아까 말했던 것처럼 Polyglot한 환경에서도 이 방식과 **동일한 방식만으로 적용이 가능**하다는 것이 장점이다.

```shell
$ kubectl apply -f redis-client-istio.yaml
deployment.apps/redis-client-istio created
destinationrule.networking.istio.io/db-mtls unchanged
```

자, 그럼 Istio에게 mTLS를 위임한 클라이언트는 어떻게 Redis와 통신할 수 있을지 보자.

```shell
$ kubectl exec -it  $(kubectl get pod -l app=redis-client-istio -o name) -- \
    redis-cli -h foo-redis-master \
    SET foo bar
OK
```

🎉 훨씬 간단해진 것을 볼 수 있다.

아주 간단한 데모였다. 이로써 데모는 끝!

⚠️ envoy proxy가 redis protocol을 해석하는 것을 바탕으로 한 이 기능은 2023.02 기준으로 나름 최신 기능이라고 볼 수 있다.
그렇기에 버전에 따라 해당 기능이 기본적으로 활성화 되어있을 수도, 비활성화 되어있을 수도, 아예 지원되지 않을 수도 있다.
따라서 직접 데모와 유사한 작업을 해보고 싶은 경우 반드시 자신의 Istio 버전을 확인해보는 것이 좋을 것이다.
해당 기능이 활성화 되었는지는 다음과 같이 Envoy proxy의 설정을 조회함으로써 확인이 가능하다.

(문서에서는 기본적으로 비활성화된다고 적혀있던데 필자의 경우 기본적으로 활성화 되어있길래 갸우뚱했다.)
```shell
$ istioctl pc all deployment/redis-client-istio -o yaml | yq | grep "envoy.filters.network.redis_proxy" -C 3
            type_urls:
              - envoy.extensions.filters.network.rbac.v3.RBAC
          - category: envoy.filters.network
            name: envoy.filters.network.redis_proxy
            type_urls:
              - envoy.extensions.filters.network.redis_proxy.v3.RedisProxy
          - category: envoy.filters.network
```

더 자세한 내용은 [protocol selection](https://istio.io/latest/docs/ops/configuration/traffic-management/protocol-selection/) 문서에서
찾아볼 수 있을 것이다.

## 마치며

잘만 동작하면 참 편하지만 암호화라는 게 항상 잘 동작하지 않으면 많은 삽질을 하게 된다. Istio를 이용하는 경우도 이는 마찬가지이다.
혹시 뭔가 계속해서 SSL 혹은 TLS 에러가 난다거나 커넥션이 성공적으로 맺어지지 못한다면 로그를 살펴보는 것을 시작으로 트러블슈팅이 필요하다.
`istioctl pc log --level=debug ...` 와 같은 명령어를 통해 로그 레벨을 낮춰서 로그를 바탕으로 상황을 분석해볼 수도 있고
Protocol selection이 의심된다면 서비스의 port name을 변경해본다거나 appProtocol이라는 필드를 추가해볼 수도 있다.
혹은 envoy configuration을 직접 다운받아 대충 훑어보는 것도 도움이 될 수 있다.

CockroachDB로 이걸 꼭 하고 싶었는데 아직 지원되지 않아서 아쉽다. CockroachDB로 참고 자료들을 찾으니 잘 안나오던데 CockroachDB와 호환되는
PostgreSQL로 찾으니 좀 자료가 나왔다. 반나절을 CockroachDB와 씨름하고 나니 Redis로는 금방할 수 있었다. 결국 서비스메쉬로 추상화 되어있기에
동일한 방법을 이용하기 때문이겠지 🙂 어서 CockorachDB 관련 기능도 출시되면 좋겠다.

## 참고 자료

* [DestinationRule를 이용해 mTLS로 통신하는 Istio 문서](https://istio.io/v1.10/docs/reference/config/networking/destination-rule/#ClientTLSSettings)
* [postgres: Support StartTLS upstream and plain-text downstream](https://github.com/envoyproxy/envoy/issues/19527)
* [postgresql은 startTLS 방식이기 때문에 사용할 수 없다는 코멘트](https://github.com/istio/istio/issues/29761#issuecomment-1065558862)
* [startTLS support in service entry](https://github.com/istio/istio/issues/33345)
* [Register postgres proxy filter API in Istio](https://github.com/istio/istio/issues/42380)
