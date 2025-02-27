
## Table of Contents

1. [Pod, Static Pod, Multi container pod, Streaming sidecar container](#1)<br>
  1.1. [개념](#1.1)<br>
  

2. [Pod](#2)<br>
  2.1. [pod 생성 조건](#2.1)<br>
  2.1.1. [test-pod-nginx.yaml 생성](#2.1.1)<br>
  2.1.2. [확인](#2.1.2)<br>
  2.1.3. [pod 생성 조건](#2.1.3)<br>
  2.1.4. [pod 배포](#2.1.4)<br>
  2.1.5. [배포 확인](#2.1.5)<br>
  2.2. [메세지를 포함하는 로그 라인 추출](#2.2)<br>
  2.3. [Static Pod](#2.3)<br>
  2.3.1. [개념](#2.3.1)<br>
  2.3.2. [Static Pod Path 변경 방법](#2.3.2)<br>




# <div id='1'> 1. Pod, Static Pod, Multi container pod, Streaming sidecar container

### <div id='1.1'> 1.1. 개념

> Pod : Pod란 Kubernetes 에서 배포할 수 있는 가장 작은 단위이며, 한개 또는 여러개의 container 들의 묶음이다.
<br>

> Static Pod : 스태틱 파드(Static Pods)는 컨트롤 플레인 컴포넌트인 kube-apiserver의 도움 없이, 특정 노드의 kubelet 데몬에 의해 직접 생성 및 관리되는 파드다.
<br>

> Multi container pod : 멀티 컨테이너 파드(Multi-Container Pod)는 2개 이상의 서로 다른 컨테이너를 포함하고 있는 파드를 의미한다.
<br>

> Streaming sidecar container : 사이드카 패턴은 원래 사용하려고 했던 기본 컨테이너의 기능을 확장하거나 보조하는 용도의 컨테이너를 추가하는 패턴이다.


<br><br>

# <div id='2'> 2. Pod

  <br>

### <div id='2.1'> 2.1. pod 생성 조건

------
> pod name: test-pod-nginx
> image: nginx:1.14
> port: 8080
------

### <div id='2.1.1'> 2.1.1 test-pod-nginx.yaml 생성

```
ubuntu@ta-cluster-1:~$ kubectl run test-pod-nginx --image=nginx:1.14 --port 8080 -oyaml > test-pod-nginx.yaml
Warning: would violate PodSecurity "restricted:v1.30": allowPrivilegeEscalation != false (container "test-pod-nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "test-pod-nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "test-pod-nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "test-pod-nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")

ubuntu@ta-cluster-1:~$ ls
 test-pod-nginx.yaml 


```

### <div id='2.1.2'> 2.1.2 cat 명령어를 통해 확인

```
ubuntu@ta-cluster-1:~$ cat test-pod-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-02-14T01:16:21Z"
  labels:
    run: test-pod-nginx
  name: test-pod-nginx
  namespace: default
  resourceVersion: "4408507"
  uid: 73f88405-06b2-4ee2-994f-7bcebdef3115
spec:
  containers:
  - image: nginx:1.14
    imagePullPolicy: Always
    name: test-pod-nginx
    ports:
    - containerPort: 8080
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-lw6fl
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  imagePullSecrets:
  - name: docker-secret
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-lw6fl
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  phase: Pending
  qosClass: BestEffort

```


### <div id='2.1.3'> 2.1.3. pod 생성 조건

-------
> pod name: test-pod-01
> image: busybox
> 환경변수 : CERT = "test-cert"
> command: /bin/sh
> args: -c "while true; do echo $(CERT); sleep 10;done"
-------


### <div id='2.1.4'> 2.1.4 pod 배포
> pod 배포
$ kubectl run subtask-pod-01 -n subtask \
	--image=busybox \
	--env=CERT="subtask-cert" \
	--command -- /bin/sh \
	--args -c "while true; do echo $(CERT); sleep 10;done" 


### <div id='2.1.5'> 2.1.5 pod 배포 확인

$ kubectl describe pod -n subtask subtask-pod-01

```
Containers:
  subtask-pod-01:
    Container ID:  cri-o://cba138f0314a60fedd060b6257e8e7813bf5ae4aec07d0bd17f7342ac0f8a2a4
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:79c9716db559ffde1170a4faf04910a08d930f511e6904c4899a1f7be2abfb34
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      --args
      -c
      while true; do echo ; sleep 10;done
    State:          Running
      Started:      Fri, 07 Feb 2025 06:21:28 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      CERT:  subtask-cert
```

### <div id='2.2'> 2.2. "kube-system/dns-autoscaler"의 log 중 "dns"메세지를 포함하는 로그 라인 추출

$ kubectl logs dns-autoscaler-6ffb84bd6-jqwmm -n kube-system | grep "dns" > /home/ubuntu/find-dns-autoscaler.log

## <div id='2.3'> 2.3 Static Pod


### <div id='2.3.1'> 2.3.1 개념

> 스태틱 파드(Static Pods)는 컨트롤 플레인 컴포넌트인 kube-apiserver의 도움 없이, 특정 노드의 kubelet 데몬에 의해 직접 생성 및 관리되는 파드다.
특정 경로에 존재하는 yaml 파일에 대해 kubelet이 자동으로 파드로 생성한다.
kube-APIServer에 의하지 않고 kubelet이 파드를 생성 및 관리하는 것이 특징이다.

### <div id='2.3.2'> 2.3.2 Static Pod Path 변경 방법

- /var/lib/kubelet/config.yaml에서 staticPath  부분을 원하는 경로로 수정한다.

```
- staticPodPath: /etc/kubernetes/manifests/
+ staticPodPath: /etc/kubernetes/manifests/sun
```

- kubelet 재시작

$ systemctl restart kubelet














