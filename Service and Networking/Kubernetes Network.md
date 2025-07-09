## Table of Contents

1. [Kubernetes Network](#1)<br>
  1.1. [Kubernetes Network 동작 원리](#1.1)<br>
  1.1.1. [Pod Network 구조](#1.1.1)<br>
  1.1.2. [Service Network 구조](#1.1.2)<br>
  1.1.3. [Ingress Network 구조](#1.1.3)<br>
  1.1.4. [Kube-proxy 역할](#1.1.4)<br>

  



<br><br><br>

## <div id='1'> 1. Kubernetes Network

### <div id='1.1'> 1.1. Kubernetes Network 동작 원리

#### <div id='1.1.1'> 1.1.1. Pod Network 구조

- 파드 내 컨테이너들이 동일한 네트워크 네임스페이스를 공유한다.
- Kubernetes에서 각 Pod는 고유한 IP 주소를 가진다. 
  (이 IP주소는 Pod 내의 모든 컨테이너들이 공유한다.)
- Pod 간 통신은 동일 노드뿐 아니라 다른 노드에 있는 Pod 간에도 직접 IP로 통신이 가능해야 한다.
- 이 구조를 실현하기 위해 CNI(Container Network Interface) 플러그인(Calico, Flannel 등)을 사용한다.
- Pod 네트워크는 NAT 없이 평면 IP 구조(Flat Network)를 지향한다.

#### <div id='1.1.2'> 1.1.2. Service Network 구조

-  Service 네트워크는 service가 할당 받는 네트워크 인터페이스이다.
모든 service는 `ClusterIP`라는 IP 주소를 부여받고 클러스터 내부적으로 이 IP주소를 통해 자신이 포워딩 해야 할 Pod들에게 트래픽을 전달한다.
즉, 기본적으로 service는 클러스터 내부적으로만 통신할 수 있게끔 설계 되어 있다.

-> 서비스는 여러가지 타입을 통해 외부통신을 가능하게끔 기능을 제공한다.
(NodePort, LoadBalancer, Ingress)

#### <div id='1.1.3'> 1.1.3. Ingress Network 구조

- Ingress는 클러스터 내부에 있는 여러 서비스로 트래픽을 라우팅하기 위한 규칙을 정의하며, 이를 통해 외부 클라이언트가 단일 IP 주소 또는 DNS 이름을 통해 여러 서비스를 접근할 수 있도록 합니다.
- HTTP 및 HTTPS 트래픽을 클러스터 내부의 서비스로 라우팅하며, 로드 밸런싱, SSL/TLS 종료, 가상 호스팅 등의 기능을 제공한다.
- URI, 호스트네임, 경로 등과 같은 웹 개념을 이해하는 프로토콜-인지형(protocol-aware configuration) 설정 메커니즘을 이용하여 HTTP (혹은 HTTPS) 네트워크 서비스를 사용 가능하게 한다. 인그레스 개념은 쿠버네티스 API를 통해 정의한 규칙에 기반하여 트래픽을 다른 백엔드에 매핑할 수 있게 해준다.


- 전제 조건
  - 인그레스 컨트롤러가 있어야 인그레스를 충족할 수 있다. 인그레스 리소스만 생성한다면 효과가 없다.

  - ingress-nginx와 같은 인그레스 컨트롤러를 배포해야 할 수도 있다. 여러 인그레스 컨트롤러 중에서 선택할 수도 있다.

  - 이상적으로, 모든 인그레스 컨트롤러는 참조 사양이 맞아야 한다. 실제로, 다양한 인그레스 컨트롤러는 조금 다르게 작동한다.

#### <div id='1.1.4'> 1.1.4. kube-proxy 구조

- kube-proxy는 클러스터의 각 노드에서 실행되는 네트워크 프록시로, 쿠버네티스의 서비스 개념의 구현부이다.
- kube-proxy는 노드의 네트워크 규칙을 유지 관리한다. 이 네트워크 규칙이 내부 네트워크 세션이나 클러스터 바깥에서 파드로 네트워크 통신을 할 수 있도록 해준다.
- kube-proxy는 운영 체제에 가용한 패킷 필터링 계층이 있는 경우, 이를 사용한다. 그렇지 않으면, kube-proxy는 트래픽 자체를 포워드(forward)한다.

> 옵션(5가지만 정리) 
```
--add_dir_header : true인 경우 파일 경로를 로그 메시지의 헤더에 추가한다.
--alsologtostderr : 파일과 함께, 표준 에러에도 로그를 출력한다.
--bind-address string     기본값: 0.0.0.0 : 프록시 서버가 서비스할 IP 주소(모든 IPv4 인터페이스의 경우 '0.0.0.0'으로 설정, 모든 IPv6 인터페이스의 경우 '::'로 설정)
--bind-address-hard-fail : 	
true인 경우 kube-proxy는 포트 바인딩 실패를 치명적인 것으로 간주하고 종료한다.
--boot-id-file string     기본값: "/proc/sys/kernel/random/boot_id" : boot-id를 위해 확인할 파일 목록(쉼표로 분리). 가장 먼저 발견되는 항목을 사용한다.
```
(참고 : https://kubernetes.io/ko/docs/reference/command-line-tools-reference/kube-proxy/)
