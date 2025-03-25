## Table of Contents

1. [개념](#1)<br>
  1.1. [Label](#1.1)<br>
  1.2. [Selector](#1.2)<br>

2. [Lable & Selector](#2)<br>
  2.1. [node에 label 추가](#2.1)<br>
  2.2. [pod 생성을 위한 yaml파일 생성 및 배포](#2.2)<br>
  2.3. [label이 생성된 노드에 배포 되었는지 확인](#2.3)<br>

3. [Taints & Tolerations](#3)<br>
  3.1. [Taints](#3.1)<br>
  3.2. [Tolerations](#3.2)<br>

<br>
<br>


# <div id='1'> 1. 개념

### <div id='1.1'> 1.1. Label
> key-value 형식의 메타데이터로, 특정 리소스에 부착되어 이를 분류하거나 식별하는데 사용된다.

### <div id='1.2'> 1.2. Selector
> Label을 이용해 쿠버네티스 리소스를 필터링하고 하는데 사용된다.

<br>
<br>

# <div id='2'> 2. Lable & Selector
Label 및 selector 를 활용하여 다음 조건의 POD을 특정 NODE에 배포 될 수 있도록 구성하세요.
<br>
- node label: sub-task-node <br>
- pod name: webserver <br>
- pod image: nginx <br>
- node selector: sub-task-nod


### <div id='2.1'> 2.1. node에 label 추가

```
ubuntu@qna-cluster-1:~$ kubectl label node qna-cluster-3 label=sub-task-node

ubuntu@qna-cluster-1:~$ kubectl get nodes -L label
NAME            STATUS   ROLES           AGE   VERSION   LABEL
qna-cluster-1   Ready    control-plane   33d   v1.30.4   
qna-cluster-2   Ready    control-plane   33d   v1.30.4   
qna-cluster-3   Ready    control-plane   33d   v1.30.4   sub-task-node
qna-cluster-4   Ready    <none>          33d   v1.30.4   
```

### <div id='2.2'> 2.2. pod 생성을 위한 yaml파일 생성 및 배포
```
ubuntu@qna-cluster-1:~$ kubectl run webserver --image=nginx --labels="label=sub-task-node" --dry-run=client -oyaml > subtask-node-nginx.yaml

ubuntu@qna-cluster-1:~$ kubectl apply -f subtask-node-nginx.yaml
```

### <div id='2.3'> 2.3. label이 생성된 노드에 배포 되었는지 확인

```
ubuntu@qna-cluster-1:~$ kubectl get pods -owide
NAME                                   READY   STATUS    RESTARTS       AGE   IP               NODE            NOMINATED NODE   READINESS GATES
webserver                              1/1     Running   0              6s    10.233.122.182   qna-cluster-3   <none>           <none>
```



# <div id='3'> 3. Taints & Tolerations

### <div id='3.1'> 3.1. Taints
> taint의 사전적인 의미는 "얼룩"으로 taint를 설정한 노드에는 pod가 생성되지 않는다.
### <div id='3.2'> 3.2. Tolerations
> toleration은 "용인"이라는 뜻으로 toleration 설정을 가진 pod는 해당 node에 할당 가능하다.



