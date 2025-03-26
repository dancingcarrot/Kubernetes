## Table of Contents

1. [개념](#1)<br>
  1.1. [Rolling Update](#1.1)<br>
  1.2. [Roll back](#1.2)<br>

2. [Rolling Update & Roll back](#2)<br>
  2.1. [Rolling Update/Roll back 을 수행하는 kubectl 명령 예시](#2.1)<br>
  2.2. [Deployment 생성 및 수정](#2.2)<br>
  2.3. [배포 및 확인](#2.3)<br>
  2.4. [rolling update, update record 진행](#2.4)<br>
  2.5. [이미지 버전 업데이트 확인](#2.5)<br>
  2.6. [history 확인](#2.6)<br>
  2.7. [컨테이너 이미지를 이전 버전으로 roll back](#2.7)<br>
  2.8. [roll back 확인](#2.8)<br>

<br>
<br>




# <div id='1'> 1. 개념

### <div id='1.1'> 1.1. Rolling update
> 서비스를 중단하지 않고 업데이트 하는 것

### <div id='1.2'> 1.2. Roll back
> 이전에 배포된 버전으로 되돌리는 작업


# <div id='2'> 2. Rolling Update & Roll back <br>

Deployment를 이용해 nginx 파드를 3개 배포한 다음 컨테이너 이미지 버전을 rolling update하고 update record를 기록합니다. 마지막으로 history 확인 후 컨테이너 이미지를 previous version으로 roll back 합니다.

- name: webserver
- Image : nginx
- Image version: 1.16
- update image version: 1.17
- label: app=payment, environment=production


### <div id='2.1'> 2.1. Rolling Update/Roll back 을 수행하는 kubectl 명령 예시 <br>

> Rolling Update <br>
> 이미지 버전을 업데이트 할 경우 <br>
> kubectl set image deployment webserer nginx=nginx:1.17 

> Roll back <br>
> 이전에 배포된 버전의 deployment로 되돌릴 경우 <br>
> kubectl rollout undo deployment webserver


### <div id='2.2'> 2.2. Deployment 생성 및 수정 <br>

```
ubuntu@qna-cluster-1:~$ kubectl create deployment webserver --image=nginx:1.16 --replicas 3 --dry-run=client -o yaml > deployment-rolling.yml
```

조건에 맞게 deployment 파일 수정
```
ubuntu@qna-cluster-1:~$ vi deployment-rolling.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: payment
    environment: production
  name: webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment
      environment: production
  template:
    metadata:
      labels:
        app: payment
        environment: production
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
```

### <div id='2.3'> 2.3. 배포 및 확인 <br>
```
ubuntu@qna-cluster-1:~$ kubectl apply -f deployment-rolling.yml

ubuntu@qna-cluster-1:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS       AGE
webserver-6855fb78d6-jf5xg             1/1     Running   0              30s
webserver-6855fb78d6-w89jb             1/1     Running   0              30s
webserver-6855fb78d6-z5r4b             1/1     Running   0              30s
```

### <div id='2.4'> 2.4. rolling update, update record 진행 <br>

```
ubuntu@qna-cluster-1:~$ kubectl set image deployment webserver nginx=nginx:1.17 --record
Flag --record has been deprecated, --record will be removed in the future
Warning: would violate PodSecurity "restricted:v1.30": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
deployment.apps/webserver image updated
```

> 특정 버전으로 롤백하려면 --to-revision 옵션을 이용해 원하는 버전을 명시해줌
kubectl rollout undo deployment/{Deployment 명} --to-revision={revision}

### <div id='2.5'> 2.5. 이미지 버전 업데이트 확인 <br>

```
ubuntu@qna-cluster-1:~$ kubectl get pod webserver-696dd55885-wdhb7 -oyaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: 3f2d436cabab3550bf7b67e0c7a04f25cfc798f61c1d3ca64734ae909c2fcbe7
    cni.projectcalico.org/podIP: 10.233.122.152/32
    cni.projectcalico.org/podIPs: 10.233.122.152/32
  creationTimestamp: "2025-03-24T07:23:22Z"
  generateName: webserver-696dd55885-
  labels:
    app: payment
    environment: production
    pod-template-hash: 696dd55885
  name: webserver-696dd55885-wdhb7
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: webserver-696dd55885
    uid: a894012a-bd22-49c5-9f35-c26c5d0f3e4f
  resourceVersion: "18643820"
  uid: 0423357b-1ed6-41bb-b09a-5cacf6e1f534
spec:
  containers:
  - image: nginx:1.17  <이미지 버전 업데이트 확인>
    imagePullPolicy: IfNotPresent
    name: nginx

    (생략)
```



### <div id='2.6'> 2.6. history 확인 <br>

```
ubuntu@qna-cluster-1:~$ kubectl rollout history deployment webserver
deployment.apps/webserver 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment webserver nginx=nginx:1.17 --record=true
```

### <div id='2.7'> 2.7. 컨테이너 이미지를 이전 버전으로 roll back <br>
```
ubuntu@qna-cluster-1:~$ kubectl rollout undo deployment webserver
Warning: would violate PodSecurity "restricted:v1.30": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
deployment.apps/webserver rolled back
```

### <div id='2.8'> 2.8. roll back 확인 <br>
```
ubuntu@qna-cluster-1:~$ kubectl get pods webserver-6855fb78d6-mqmnh -oyaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: d5fc77f722773aeb9f254a7123f2f4186f96872775ecb41b86303c6dc8052e12
    cni.projectcalico.org/podIP: 10.233.98.189/32
    cni.projectcalico.org/podIPs: 10.233.98.189/32
  creationTimestamp: "2025-03-24T07:27:46Z"
  generateName: webserver-6855fb78d6-
  labels:
    app: payment
    environment: production
    pod-template-hash: 6855fb78d6
  name: webserver-6855fb78d6-mqmnh
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: webserver-6855fb78d6
    uid: 5f3b62ac-ccfd-4ee8-88d1-7933d80c1aac
  resourceVersion: "18645663"
  uid: 13327d40-0d0f-4a2b-82c7-ecf78cfb252e
spec:
  containers:
  - image: nginx:1.16       <이미지 버전 확인>
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}

    (생략)
```
