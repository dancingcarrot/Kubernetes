## Table of Contents

1. [개념](#1)<br>
  1.1. [Deployment](#1.1)<br>
  1.2. [Scale-in/Scale-out](#1.2)<br>

2. [Scale-in/Scale-out](#2)<br>
  2.1. [deployment 생성 및 수정](#2.1)<br>
  2.2. [배포 확인](#2.2)<br>
  2.3. [kubectl 명령어를 사용하여 pod의 수를 3개로 확장](#2.3)<br>
  2.4. [확장 후 pod 개수 확인](#2.4)<br>

<br>
<br>


# <div id='1'> 1. 개념

### <div id='1.1'> 1.1. Deployment

> Deployment란 k8s에서 애플리케이션 단위를 관리하는 controller이다.
<br>

### <div id='1.2'> 1.2. Scale-in/Scale-out

> Scale-in : pod 또는 node의 개수를 감소시켜 불필요한 리소스 낭비를 방지하는 것 <br>
> Scale-out : pod 또는 node의 개수를 증가시켜 부하를 분산시키는 것<br>

<br>

# <div id='2'> 2. Scale-in/Scale-out <br>

 kubectl 명령을 통해 webserver 라는 이름으로 yaml 을 생성하고, kubectl 명령을 통해 webserver Deployment의 pod 수를 3개로 확장하세요.

- kind: deployment <br>
- Name: webserver <br>
- 2 replicas <br>
- label: app_env_stage=dev <br>
- container name: webserver <br>
- container image: nginx:1.14 <br>

### <div id='2.1'> 2.1. Deployment 파일 생성 및 수정 <br>

Deployment 파일 생성

```
ubuntu@qna-cluster-1:~$ kubectl create deployment webserver --image=nginx:1.14 --replicas=2 --dry-run=client -o yaml > webserver.yaml
```

 조건에 맞게 deployment 파일 수정
```
ubuntu@qna-cluster-1:~$ vi webserver.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app_env_stage: dev
  template:
    metadata:
      labels:
        app_env_stage: dev
    spec:
      containers:
      - image: nginx:1.14
        name: nginx
```

### <div id='2.2'> 2.2. 배포 확인 <br>
```
ubuntu@qna-cluster-1:~$ kubectl get pods
webserver-5d89946b46-chbgq             1/1     Running   0              9s
webserver-5d89946b46-p9lr5             1/1     Running   0              9s
```

### <div id='2.3'> 2.3. kubectl 명령어를 사용하여 pod의 수를 3개로 확장 <br>
```
ubuntu@qna-cluster-1:~$ kubectl scale --replicas=3 deployment webserver
deployment.apps/webserver scaled
```

### <div id='2.4'> 2.4. 확장 후 pod 개수 확인 <br>
```
ubuntu@qna-cluster-1:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
webserver-5d89946b46-ftxbz             1/1     Running   0          5s
webserver-5d89946b46-mphf4             1/1     Running   0          47s
webserver-5d89946b46-z9dcv             1/1     Running   0          47s
```
