
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
  



  2.4. [Multi contianer pod](#2.3)<br>

  2.5. [Sidecar container pod](#2.5)<br>




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


### <div id='2.3.3'> 2.3.3 Static Pod 생성 조건

-------
> pod name: subtask-static-pod-nginx
> image: busybox
> port: 80
-------

### <div id='2.3.4'> 2.3.4 Static Pod 배포

 ```
 $ kubectl run subtask-static-pod-nginx -oyaml \
    --image=nginx \
    --port=80 \
    --dry-run=client \
    > /etc/kubernetes/manifests/subtask-static-pod-nginx.yaml

 ``` 
### <div id='2.3.5'> 2.3.5 Static Pod 삭제

> Static pod path에서 subtask-static-pod-nginx.yaml파일을 삭제한 후 파드를 삭제한다.

```
root@ta-task-cluster-1:/etc/kubernetes/manifests# rm -rf subtask-static-pod-nginx.yaml 
root@ta-task-cluster-1:/etc/kubernetes/manifests# kubectl delete pod subtask-static-pod-nginx-ta-task-cluster-1
```


## <div id='2.4'> 2.4. Multi container pod



### <div id='2.4.1'> 2.4.1. container와 pod의 차이점
> 파드는 컨테이너로 이루어져 있으며, 컨테이너를 만들고 관리하는 도구가 도커이다.
파드는 1개 이상의 컨테이너로 이루어져 있다.
컨테이너는 개별적인 실행 환경을 갖는데, CPU, 네트워크, 메모리와 같은 시스템 자원을 독자적으로 사용하도록 할당된 환경이다.

### <div id='2.4.2'> 2.4.2. 컨테이너를 동작시키는 pod 를 생성

-------
> pod name : subtask-multi-pod
> pod image : nginx, redis, memcached
-------

- 컨테이너가 하나인 파드 생성
```
$ kubectl run multi --image=nginx --dry-run=client -oyaml > multi.yaml
```
- multi.yaml 파일에 컨테이너 추가하여 편집


```
apiVersion: v1
kind: Pod
metadata:
  name: multi
spec:
  containers:
  - image: nginx
    name: nginx
  - image: redis
    name: redis
  - image: memcached
    name: memcached
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```

- 배포

```
ubuntu@ta-task-cluster-1:~$ kubectl apply -f multi.yaml

ubuntu@ta-task-cluster-1:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS        AGE
multi                                  3/3     Running   0               79s

```


```
ubuntu@ta-task-cluster-1:~$ kubectl describe pod multi

Containers:
  nginx:
    Container ID:   cri-o://4eb2efe8b2f481a1730ce2e1e9d9f477cae723f9c92fd923baea0f8dd0950528
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:088eea90c3d0a540ee5686e7d7471acbd4063b6e97eaf49b5e651665eb7f4dc7
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 10 Feb 2025 00:27:45 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w9s7d (ro)
  redis:
    Container ID:   cri-o://1c50a1fef3494a46113c3adea24671be71a16f3c8a2f85ca199acef18f6637fd
    Image:          redis
    Image ID:       docker.io/library/redis@sha256:93a8d83b707d0d6a1b9186edecca2e37f83722ae0e398aee4eea0ff17c2fad0e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 10 Feb 2025 00:27:53 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w9s7d (ro)
  memcached:
    Container ID:   cri-o://cbdf25dea6ecd6813fb859d8608186f04c8d92d7c6c49e923f7cded74b7a0a42
    Image:          memcached
    Image ID:       docker.io/library/memcached@sha256:a1d3a1c601732fc04245de562a3667db0bc3ae6964f0e830183d4dd5514b1b38
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 10 Feb 2025 00:28:00 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w9s7d (ro)


```


## <div id='2.5'> 2.5. Streaming sidecar container



### <div id='2.5.1'> 2.5.1. Sidecar container개념

  > Sidecar contaienr는 기본 컨테이너의 기능을 확장하거나 보조하는 용도의 컨네이너이다.


### <div id='2.5.2'> 2.5.2. 로그 스트리밍 사이드카 컨테이너 운영

 ```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name:  count
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}

 ```

 ```
 $ kubectl get pod counter -o yaml > subtask-sun2.yaml
 > subtask-sun2.yaml에 sidecar container 추가

          readOnly: true
  - name: count-log-1
    image: busybox
    command: [/bin/sh, -c, 'tail -n+1 -F /var/log/1.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-2
    image: busybox
    command: [/bin/sh, -c, 'tail -n+1 -F /var/log/2.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  dnsPolicy: ClusterFirst

 ```

> 처음 실행했던 counter pod 삭제
```
$ kubectl delete pod counter
```
> sidecar container를 추가한 yaml 적용

```
$ kubectl apply -f pod subtask-sun2.yaml
```

> describe, logs 로 확인

```
$ kubectl describe pod counter

count-log-1:
    Container ID:  cri-o://6d947a7ed9b533ffada7e2dd3f38d14eb6066f005c65cd3dc9003411b9263042
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      tail -n+1 -F /var/log/1.log
    State:          Running
      Started:      Thu, 27 Mar 2025 00:29:24 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log from varlog (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ppthj (ro)
  count-log-2:
    Container ID:  cri-o://cfdfee8e6cbb24ab941be7064bc8214c85c076a74c479d635969b94c835f639a
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:37f7b378a29ceb4c551b1b5582e27747b855bbfaa73fa11914fe0df028dc581f
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      tail -n+1 -F /var/log/2.log
    State:          Running
      Started:      Thu, 27 Mar 2025 00:29:26 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log from varlog (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ppthj (ro)

ubuntu@ta-task-cluster-1:~$ kubectl logs counter
Defaulted container "count" out of: count, count-log-1, count-log-2


ubuntu@qna-cluster-1:~/workspace/sun/2$ kubectl logs counter -c count-log-1 --tail=5
141: Thu Mar 27 00:31:43 UTC 2025
142: Thu Mar 27 00:31:44 UTC 2025
143: Thu Mar 27 00:31:45 UTC 2025
144: Thu Mar 27 00:31:46 UTC 2025
145: Thu Mar 27 00:31:47 UTC 2025


ubuntu@qna-cluster-1:~/workspace/sun/2$ kubectl logs counter -c count-log-2 --tail=5
Thu Mar 27 00:31:28 UTC 2025 INFO 126
Thu Mar 27 00:31:29 UTC 2025 INFO 127
Thu Mar 27 00:31:30 UTC 2025 INFO 128
Thu Mar 27 00:31:31 UTC 2025 INFO 129
Thu Mar 27 00:31:32 UTC 2025 INFO 130


```
