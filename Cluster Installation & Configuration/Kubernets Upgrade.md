

## Table of Contents

1. [kubeadm, kubelet, kubectl](#1)<br>
  1.1. [개념](#1.1)<br>
  

2. [Kubernets Upgrade](#2)<br>
  2.1. [Master Node 버전 업그레이드](#2.1)<br>
  2.1.1. [쿠버네티스 패키지 레포지토리 변경](#2.1.1)<br>
  2.1.2. [마이너 버전의 업그레이드 가능한 최신 패치 목록 조회](#2.1.2)<br>
  2.1.3. [kubeadm 업그레이드](#2.1.3)<br>
  2.1.4. [master node drain](#2.1.4)<br>
  2.1.5. [kubelet / kubectl 업그레이드 및 재시작](#2.1.5)<br>
  2.1.6. [master node uncordon](#2.1.6)<br>
  2.1.7. [version upgrade 확인](#2.1.7)<br>
  2.2. [Worker Node 버전 업그레이드](#2.2)<br>
  2.2.1. [worker node 접속](#2.2.1)<br>
  2.2.2. [쿠버네티스 패키지 레포지토리 변경](#2.2.2)<br>
  2.2.3. [업그레이드 가능한 최신 패치 목록 조회](#2.2.3)<br>
  2.2.4. [worker node drain 설정](#2.2.4)<br>
  2.2.5. [kubelet / kubectl 업그레이드 및 재시작](#2.2.5)<br>
  2.2.6. [worker node uncordon](#2.2.6)<br>





# <div id='1'> 1. kubeadm, kubelet, kubectl

### <div id='1.1'> 1.1. 개념

- kubeadm : 쿠버네티스 클러스터를 부트스트랩(초기화)하고 구성하는 도구
- kubelet : 노드의 상태를 관리하고 마스터와 통신을 담당 / 파드의 생성, 실행 및 모니터링에 핵심적인 역할을 수행
- kubectl : 쿠버네티스 클러스터에 명령을 내리는 CLI(Command Line Interface)

<br><br>

# <div id='2'> 2. Kubernets Upgrade
<br>


## <div id='2.1'> 2.1. Master Node 버전 업그레이드 (1.30.x -> 1.31.x)

<br>

### <div id='2.1.1'> 2.1.1. 쿠버네티스 패키지 레포지토리 변경

```
$ echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/soures.list.d/kubernetes.list
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ sudo apt-get update
$ sudo apt-cache madison kubeadm
```
<details>

<summary> `N: Unable to locate package kubeadm` 로그가 발생했을 경우</summary>

```
ubuntu@ta-task-cluster-1:~$ sudo apt-cache madison kubeadm
N: Unable to locate package kubeadm
```
위와 같은 로그가 나왔을 경우 하단의 방법으로 레포지토리 변경 후 진행

*  쿠버네티스 패키지 레포지토리 변경
```
$ sudo rm -rf /etc/apt/sources.list.d/kubernetes.list
$  sudo apt-get update
$  sudo apt-get install -y apt-transport-https ca-certificates curl gpg
$  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$  sudo apt-get update
$  sudo apt-get install -y kubelet kubeadm kubectl
$  sudo apt-mark hold kubelet kubeadm kubectl
```

</details>

<br>

### <div id='2.1.2'> 2.1.2. 마이너 버전의 업그레이드 가능한 최신 패치 목록 조회

```
$ sudo apt update
$ sudo apt-cache madison kubeadm

```
```
  ubuntu@ta-task-cluster-1:~$ sudo apt-cache madison kubeadm
   kubeadm | 1.31.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
```


### <div id='2.1.3'> 2.1.3. kubeadm 업그레이드
  ```
  $ sudo apt-mark unhold kubeadm && \
    sudo apt-get update && sudo apt-get install -y kubeadm=1.31.0-1.1 && \
    sudo apt-mark hold kubeadm
  ```
### <div id='2.1.4'> 2.1.4. 파드가 할당되지 않도록 master node drain 설정
```
$ kubectl drain ta-task-cluster-1 --ignore-daemonsets
```
### <div id='2.1.5'> 2.1.5. kubelet / kubectl 업그레이드 및 재시작
```
$ apt-mark unhold kubelet kubectl && \
    apt-get update && apt-get install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1 && \ 
      apt-mark hold kubelet kubectl
```
kubelet 재시작
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```  

### <div id='2.1.6'> 2.1.6. master node drain해제-> uncordon
```
$ kubectl uncordon ta-task-cluster-1
```


### <div id='2.1.7'> 2.1.7. version upgrade 확인
  ```
ubuntu@ta-task-cluster-1:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE    VERSION
ta-task-cluster-1   Ready    control-plane   6d4h   v1.31.5
ta-task-cluster-2   Ready    <none>          6d4h   v1.30.4
ta-task-cluster-5   Ready    <none>          20h    v1.30.4
```

<br>
<br>

## <div id='2.2'> 2.2. worker node 버전 업그레이드 (1.30.x -> 1.31.x)

<br>

### <div id='2.2.1'> 2.2.1. worker node 접속
```
$ ssh ta-task-cluster-2
```

### <div id='2.2.2'> 2.2.2. 쿠버네티스 패키지 레포지토리 변경

```
$ sudo rm -rf /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl gpg
$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

### <div id='2.2.3'> 2.2.3. 업그레이드 가능한 최신 패치 목록 조회
```
ubuntu@ta-task-cluster-2:~$ sudo apt-cache madison kubeadm
   kubeadm | 1.31.5-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
   kubeadm | 1.31.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
```

### <div id='2.2.4'> 2.2.4. 파드가 할당되지 않도록 worker node drain 설정

```
ubuntu@ta-task-cluster-1:~$ kubectl drain ta-task-cluster-2 --ignore-daemonsets
ubuntu@ta-task-cluster-1:~$ kubectl get nodes
NAME                STATUS                     ROLES           AGE    VERSION
ta-task-cluster-1   Ready                      control-plane   6d4h   v1.31.5
ta-task-cluster-2   Ready,SchedulingDisabled   <none>          6d4h   v1.31.5
ta-task-cluster-5   Ready                      <none>          20h    v1.30.4
```

### <div id='2.2.5'> 2.2.5. kubelet / kubectl 업그레이드 및 재시작
```
$ apt-mark unhold kubelet kubectl && \
    apt-get update && apt-get install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1 && \ 
      apt-mark hold kubelet kubectl
```
kubelet 재시작
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```  

### <div id='2.2.6'> 2.2.6. worker node drain해제-> uncordon
```
ubuntu@ta-task-cluster-1:~$ kubectl uncordon ta-task-cluster-2
node/ta-task-cluster-2 uncordoned
ubuntu@ta-task-cluster-1:~$ kubectl get nodes
NAME                STATUS   ROLES           AGE    VERSION
ta-task-cluster-1   Ready    control-plane   6d4h   v1.31.5
ta-task-cluster-2   Ready    <none>          6d4h   v1.31.5
ta-task-cluster-5   Ready    <none>          20h    v1.30.4
```
