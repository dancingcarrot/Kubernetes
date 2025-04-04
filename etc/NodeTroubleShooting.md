

## Table of Contents

1. [TroubleShooting](#1)<br>
  1.1. [이슈](#1.1)<br>
  1.2. [노드 상태 확인](#1.2)<br>
  1.3. [qna-cluster-2 노드접속](#1.3)<br>
  1.4. [podman 상태 확인](#1.4)<br>
  1.5. [podman 재시작/kubelet enable설정](#1.5)<br>
  1.6. [상태확인](#1.6)<br>
  1.7. [qna-cluster-3 접속](#1.7)<br>
  1.8. [kubelet 상태 확인](#1.8)<br>
  1.9. [kubelet 재시작](#1.9)<br>
  1.10. [상태확인](#1.10)<br>
  1.11. [schedulingdisabled 상태 해제](#1.11)<br>
  1.12. [결과](#1.12)<br>








# <div id='1'> 1. TroubleShooting

### <div id='1.1'> 1.1. 이슈

* Cluster Trouble Shooting
> - qna-cluster-2 와 qna-cluster-3 에 이슈가 발생하였습니다. 
> - 원인을 파악하고 조치해보세요.



<br><br>

### <div id='1.2'> 1.2. 노드 상태 확인
<br>

```
ubuntu@qna-cluster-1:~$ kubectl get nodes
NAME            STATUS                        ROLES           AGE   VERSION
qna-cluster-1   Ready                         control-plane   44d   v1.30.4
qna-cluster-2   Ready, SchedulingDisabled     control-plane   44d   v1.30.4
qna-cluster-3   Ready, SchedulingDisabled     control-plane   44d   v1.30.4
qna-cluster-4   Ready                            <none>       44d   v1.30.4
```

-> 현재 두개의 노드가 SchedulingDisabled 인 것을 확인할 수 있다.


### <div id='1.3'> 1.3. qna-cluster-2 노드접속

```
$ ssh qna-cluster-2
```
해당 명령어를 통해 qna-cluster-2로 접속

### <div id='1.4'> 1.4. podman 상태 확인

```
ubuntu@qna-cluster-2:~$  systemctl status podman
○ podman.service - Podman API Service
     Loaded: loaded (/lib/systemd/system/podman.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ podman.socket
       Docs: man:podman-system-service(1)

Apr 04 00:36:57 qna-cluster-2 systemd[1]: Started Podman API Service.
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="/usr/bin/podman filtering at log level info"
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="Not using native diff for overlay, this may cause degraded performance for building images: kernel has CONFIG_OVERLAY_FS_RE>
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="Found CNI network k8s-pod-network (type=calico) at /etc/cni/net.d/10-calico.conflist"
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="Setting parallel job count to 7"
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="using systemd socket activation to determine API endpoint"
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="using API endpoint: ''"
Apr 04 00:36:57 qna-cluster-2 podman[4058407]: time="2025-04-04T00:36:57Z" level=info msg="API service listening on \"/run/podman/podman.sock\""
Apr 04 00:37:02 qna-cluster-2 systemd[1]: podman.service: Deactivated successfully.


ubuntu@qna-cluster-2:~$ sudo systemctl is-enabled kubelet
disenabled

```
> podman이 inactive 상태인 것을 확인할 수 있다.
> kubelet이 disenabled 상태인 것을 확인할 수 있다.

### <div id='1.5'> 1.5. podman 재시작/kubelet enable설정

```
$ sudo systemctl restart podman

$ sudo systemctl enable kubelet
```

* podman도 disabled인 경우 sudo systemctl enable podman 명령어를 통해서 enable로 변경해준다.

### <div id='1.6'> 1.6. 상태확인

```
ubuntu@qna-cluster-2:~$ sudo systemctl restart podman
ubuntu@qna-cluster-2:~$ systemctl status podman
● podman.service - Podman API Service
     Loaded: loaded (/lib/systemd/system/podman.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-04-04 04:08:54 UTC; 1s ago
TriggeredBy: ● podman.socket
       Docs: man:podman-system-service(1)
   Main PID: 4176541 (podman)
      Tasks: 7 (limit: 9478)
     Memory: 9.7M
        CPU: 116ms
     CGroup: /system.slice/podman.service
             └─4176541 /usr/bin/podman --log-level=info system service

Apr 04 04:08:54 qna-cluster-2 systemd[1]: Starting Podman API Service...
Apr 04 04:08:54 qna-cluster-2 systemd[1]: Started Podman API Service.
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="/usr/bin/podman filtering at log level info"
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="Not using native diff for overlay, this may cause degraded performance for building images: kernel has CONFIG_OVERL>
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="Found CNI network k8s-pod-network (type=calico) at /etc/cni/net.d/10-calico.conflist"
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="Setting parallel job count to 7"
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="using systemd socket activation to determine API endpoint"
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="using API endpoint: ''"
Apr 04 04:08:54 qna-cluster-2 podman[4176541]: time="2025-04-04T04:08:54Z" level=info msg="API service listening on \"/run/podman/podman.sock\""


ubuntu@qna-cluster-2:~$ sudo systemctl is-enabled kubelet
enabled
```

### <div id='1.7'> 1.7. qna-cluster-3 접속

```
$ ssh qna-cluster-2
```

### <div id='1.8'> 1.8. kubelet 상태 확인

```
ubuntu@qna-cluster-3:~$ systemctl status kubelet
○ kubelet.service - Kubernetes Kubelet Server
     Loaded: loaded (/etc/systemd/system/kubelet.service; disabled; vendor preset: enabled)
     Active: inactive (dead) since Fri 2025-04-04 01:10:38 UTC; 7min ago

     ...(생략)
     

```

### <div id='1.9'> 1.9. kubelet 재시작

```
ubuntu@qna-cluster-1:~$ sudo systemctl start kubelet
```

### <div id='1.10'> 1.10. 상태확인

```
ubuntu@qna-cluster-3:~$ systemctl status kubelet
● kubelet.service - Kubernetes Kubelet Server
     Loaded: loaded (/etc/systemd/system/kubelet.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-04-04 01:18:59 UTC; 2h 45min ago

     ...(생략)

```

### <div id='1.11'> 1.11. schedulingdisabled 상태 해제

> master node에서 진행

```
$ kubectl uncordon qna-cluster-2
$ kubectl uncordon qna-cluster-3
```

### <div id='1.12'> 1.12. 결과

```
ubuntu@qna-cluster-1:~$ kubectl get nodes
NAME            STATUS   ROLES           AGE   VERSION
qna-cluster-1   Ready    control-plane   44d   v1.30.4
qna-cluster-2   Ready    control-plane   44d   v1.30.4
qna-cluster-3   Ready    control-plane   44d   v1.30.4
qna-cluster-4   Ready    <none>          44d   v1.30.4
```


