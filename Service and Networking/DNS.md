1. [DNS](#1)<br>
  1.1. [서비스 및 파드용 DNS](#1.1)<br>
  1.2. [Service and DNS lookup 구성](#1.2)<br>

<br>

## <div id='1'> 1. DNS

### <div id='1.1'> 1.1. 서비스 및 파드용 DNS

> 쿠버네티스는 파드와 서비스를 위한 DNS 레코드를 생성한다. 사용자는 IP 주소 대신에 일관된 DNS네임을 통해서 서비스에 접속할 수 있다.
> 쿠버네티스는 DNS를 설정 하기 위해 사용되는 파드와 서비스에 대한 정보를 발행한다. Kubelete은 실행 중인 컨테이너가 IP가 아닌 이름으로 서비스를 검색할 수 있도록 파드의 DNS를 설정한다.
> 클러스터 내의 서비스에는 DNS 네임이 할당된다. 기본적으로 클라이언트 파드의 DNS 검색 리스트는 파드 자체의 네임스페이스와 클러스터의 기본 도메인을 포함한다.

(https://kubernetes.io/ko/docs/concepts/services-networking/dns-pod-service/)

> Service DNS 형식: <svc-name>.<namespace>.svc.cluster.local
> Pod DNS 형식: <pod-ip>.<namespace>.pod.clouster.local


### <div id='1.2'> 1.2. Service and DNS Lookup 구성

```
> image nginx를 사용하는 resolver pod를 생성하고 resolver-service라는 service를 구성합니다.
> 클러스터 내에서 service와 pod 이름을 조회할 수 있는지 테스트합니다.
> • dns 조회에 사용하는 pod 이미지는 busybox:1.28이고, service와 pod 이름 조회는 nlsookup을 사용합니다.
> • service 조회 결과와 pod name 조회 결과를 추출하세요.
```

1. > image nginx를 사용하는 resolver pod를 생성하고 resolver-service라는 service를 구성
```
# pod 생성
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl run resolver --image=nginx --labels="app=resolver" -n subtask
Warning: would violate PodSecurity "restricted:v1.31": allowPrivilegeEscalation != false (container "resolver" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "resolver" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "resolver" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "resolver" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
pod/resolver created

# service 생성
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl expose pod resolver --port=80 --name=resolver-service -n subtask
service/resolver-service exposed
```

2. 클러스터 내에서 service와 pod 이름을 조회할 수 있는지 테스트합니다.
• dns 조회에 사용하는 pod 이미지는 busybox:1.28이고, service와 pod 이름 조회는 nlsookup을 사용합니다.
```
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl run dns --rm -it --image=busybox:1.28 -n subtask -- /bin/sh 
Warning: would violate PodSecurity "restricted:v1.31": allowPrivilegeEscalation != false (container "dns" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "dns" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "dns" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "dns" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
If you don't see a command prompt, try pressing enter.
/ # nslookup resolver
Server:    169.254.25.10
Address 1: 169.254.25.10

nslookup: can't resolve 'resolver'
/ # nslookup resolver-service
Server:    169.254.25.10
Address 1: 169.254.25.10

Name:      resolver-service
Address 1: 10.233.4.253 resolver-service.subtask.svc.cluster.local
```


