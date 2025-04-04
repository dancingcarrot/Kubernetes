## Table of Contents

1. [Node management](#1)<br>
  1.1. [kubectl drain option](#1.1)<br>
  1.2. [Node SchedulingDisabled](#1.2)<br>
  1.3. [Ready 상태 node 수 기록](#1.3)<br>
  


<br>
<br>


# <div id='1'> 1. Node management

### <div id='1.1'> 1.1. kubectl drain option

> kubectl drain node [노드명]

1. --ignore-daemonsets=true; : daemonsets로 실행한 파드들을 무시하고 drain 설정을 적용할 수 있다.
2. --delete-emptydir-data=false : node가 drain될 때 빈 디렉토리(emptyDir)에 있는 데이터를 삭제할지 여부를 결정한다.
3.  --disable-eviction=false :  강제 Drain이 삭제를 사용하도록 한다. 
 

<br>
<br>

### <div id='1.2'> 1.2. Node SchedulingDisabled


특정 노드를 스케줄링 불가능하게 설정하고, 해당 노드에서 실행 중인 모든 Pod을 다른 node로 reschedule 하세요.
<br>

```
> 스케줄링 설정 전

ubuntu@qna-cluster-1:~$ kubectl get nodes
NAME            STATUS   ROLES           AGE   VERSION
qna-cluster-1   Ready    control-plane   34d   v1.30.4
qna-cluster-2   Ready    control-plane   34d   v1.30.4
qna-cluster-3   Ready    control-plane   34d   v1.30.4
qna-cluster-4   Ready    <none>          34d   v1.30.4
```

```
> 스케줄링 불가능 설정 진행

ubuntu@qna-cluster-1:~$ kubectl drain qna-cluster-4 --ignore-daemonsets --force --delete-emptydir-data
node/qna-cluster-4 cordoned
Warning: ignoring DaemonSet-managed Pods: chaos-mesh/chaos-daemon-2rlgg, kube-system/calico-node-ttgh6, kube-system/kube-proxy-lbnmx, kube-system/nodelocaldns-xbzzh, metallb-system/speaker-6s645
evicting pod mariadb/mariadb-0
evicting pod kyverno/kyverno-cleanup-ephemeral-reports-29047790-m9xzv
evicting pod kyverno/kyverno-cleanup-admission-reports-29047790-psfhz
evicting pod kyverno/kyverno-cleanup-cluster-admission-reports-29047790-cmrzk
evicting pod kyverno/kyverno-cleanup-update-requests-29047790-rqmbk
evicting pod kyverno/kyverno-cleanup-cluster-ephemeral-reports-29047790-88plb
pod/kyverno-cleanup-update-requests-29047790-rqmbk evicted
pod/kyverno-cleanup-cluster-ephemeral-reports-29047790-88plb evicted
pod/kyverno-cleanup-cluster-admission-reports-29047790-cmrzk evicted
pod/kyverno-cleanup-admission-reports-29047790-psfhz evicted
pod/kyverno-cleanup-ephemeral-reports-29047790-m9xzv evicted
pod/mariadb-0 evicted
node/qna-cluster-4 drained
```

```
> 확인
ubuntu@qna-cluster-1:~$ kubectl get nodes
NAME            STATUS                     ROLES           AGE   VERSION
qna-cluster-1   Ready                      control-plane   34d   v1.30.4
qna-cluster-2   Ready                      control-plane   34d   v1.30.4
qna-cluster-3   Ready                      control-plane   34d   v1.30.4
qna-cluster-4   Ready,SchedulingDisabled   <none>          34d   v1.30.4

```

### <div id='1.3'> 1.3. Ready 상태 node 수 기록

Ready 상태(NoSchedule로 taint된 node는 제외)인 node를 찾아 그 수를 notaint_ready_node.log 에 기록.

```
root@qna-cluster-1:/home/ubuntu/workspace/sun# echo 3 > /var/subtask/notaint_ready_node.log
root@qna-cluster-1:/home/ubuntu/workspace/sun# cat /var/subtask/notaint_ready_node.log
3
```


