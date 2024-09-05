# Istio

- Istio는 오픈소스 서비스 메쉬 프레임워크로 서비스 간 네트워크 통신을 제어하고 관리한다. 개인적인 해석으로 조금 더 풀어서 설명하면 쿠버네티스 내 배포된 여러 서비스들을 중앙 관리할 수 있는 툴이다.

- 서비스 메쉬는 어플리케이션 서비스 간 통신을 처리하는 소프트웨어이다.

- Istio의 사용법

- Istio는 클러스터 내 복수의 서비스 리소스에 대한 관리 그러니까 트래픽 라우팅, 그 트래픽 자체의 양에 대한 모니터링, 아마 로그도 가진 서비스 메쉬 프레임워크이다. 불명확한 설명이 오히려 낫다. 모르는 부분이니까.

- Helm 릴리즈가 있어서 Helm을 사용해 설치가 가능하고 values.yaml을 통해 클러스터 내 서비스 리소스를 정의한다. 이게 서비스 배포를 의미하는 건 아니고 생성된 서비스들의 포트 등을 정의하는 yaml 파일이다. **_정정할 부분 values.yaml을 분리해서 gateway의 서비스 부분만 할 수도 있는건지 아무튼 이부분은 내가 직접 설치한 것이 아니고 지금 인프라가 MSA 라이브 서비스를 위한 것이 아니기 때문에 불확실하다._**

```yaml
service:
      type: LoadBalancer               # 서비스 유형 (ClusterIP, NodePort, LoadBalancer)
      ports:
      - name: http                     # HTTP 포트 설정
        port: 80                       # 클러스터 외부에서 접근하는 포트
        targetPort: 8080               # 실제 컨테이너가 리스닝하는 포트
        nodePort: 30080                # NodePort 설정 (LoadBalancer나 NodePort일 때 필요)
      - name: https                    # HTTPS 포트 설정
        port: 443                      # 클러스터 외부에서 HTTPS로 접근하는 포트
        targetPort: 8443               # 실제 컨테이너가 리스닝하는 포트
        nodePort: 30443                # NodePort 설정 (옵션)
```

- 이후 values.yaml이 변경되면(서비스 리소스가 추가, 제거, 변경되면) helm upgrade로 적용을 할 수 있는 것 같은데 이게 무중단으로 적용이 되는 지 확인은 아직 해보지 못했다.

- 설치 후에 Istio가 가지는 커스텀 리소스인 Gateway와 Virtual Service를 사용해서 트래픽 라우팅을 정의한다.

- Gateway는 hosts와 port를 정의해 어떤 트래픽을 외부로부터 들일지를 정의하고

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
spec:
  selector:
    istio: ingressgateway  # Istio Ingress Gateway 컨트롤러에 의해 처리됨
  servers:
  - port:
      number: 80           # 포트 80으로 들어오는 트래픽
      name: http
      protocol: HTTP
    hosts:
    - "example.com"         # 특정 호스트로 들어오는 트래픽만 처리
```

- Vitual Service는 쿠버네티스 Ingress 리소스처럼 uri, port등으로 트래픽을 클러스터 내 어떤 서비스로 라우팅할지 결정한다.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-virtual-service
spec:
  hosts:
  - "example.com"            # Gateway에서 처리된 호스트
  gateways:
  - my-gateway               # Gateway와 연결
  http:
  - match:
      uri:
        prefix: "/api"       # "/api"로 시작하는 트래픽
    route:
    - destination:
        host: my-service     # 트래픽이 라우팅될 내부 서비스
        port:
          number: 8080
```

## Istio 설치

- Helm repo를 사용해서 설치를 할 수 있다.

- 3개의 패키지를 설치해야 하는데 istio/base, istio/istiod, istio/gateway이다.

### Istio/base

- istio/base는 Istio를 구성하는 베이스이다.

- VirtualService, DestinationRule, Gateway같은 핵심 기능을 쿠버네티스의 커스텀 리소스로 정의하는 CRDs(Custom Resource Definitions)를 담고 있고 Istio가 동작하기 위해 필요한 쿠버네티스 API 리소스들을 정의한다.

### Istio/istiod

- Istio/istiod는 Istio의 컨트롤 플레인이다.

- 중앙에서 데이터 플레인 역할인 Envoy 프록시와 XDS 프로토콜을 통해 통신을 하고 클러스터 내 서비스들이 서로를 자동으로 발견하고 연결할 수 있도록 서비스 디스커버리 기능을 제공한다.

- XDS에 대해 조금 더 풀어서 설명하면 DS는 디스커버리 서비스를 의미하는데 Cluster Discovery Service는 클러스터에 대한 정보를 제공하는 것이고 이 XDS API를 통해 Envoy 프록시는 재시작하지 않아도 구성의 변경을 업데이트하고 서비스 간 통신이 동적으로 제어된다.

### Istio/gateway

- Istio/gateway는 Istio Ingress Gateway로 클러스터 외부에서 내부로 들어오는 트래픽을 처리한다.

- 이 트래픽은 데이터 플레인인 Envoy가 처리하고 Ingress Gateway는 Envoy 프록시를 기반으로 동작한다.

## Envoy

- Envoy는 서비스 메쉬의 데이터 플레인으로 트래픽을 직접 제어하는 역할을 한다.

- Envoy 프록시는 서비스의 파드와 함께 사이드카 프록시로 배포되며 해당 서비스로 도달하는 트래픽을 가로채고 직접 제어하는 역할을 한다.

