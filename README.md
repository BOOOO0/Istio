# Istio

- Istio는 오픈소스 서비스 메쉬 프레임워크로 서비스 간 네트워크 통신을 제어하고 관리한다. 개인적인 해석으로 조금 더 풀어서 설명하면 쿠버네티스 내 배포된 여러 서비스들을 중앙 관리할 수 있는 툴이다.

- 서비스 메쉬는 어플리케이션 서비스 간 통신을 처리하는 소프트웨어이다.

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

