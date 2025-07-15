1. [Monitoring](#1)<br>
  1.1. [kube-api로그 모니터링](#1.1)<br>
  1.2. [POD중 CPU를 가장 많이 사용하는 순서대로 출력](#1.2)<br>
  1.3. [NODE중 메모리를 가장 적게 사용하는 순서대로 출력](#1.3)<br>
  1.4. [클러스터에 구성된 모든 PV를 capacity별로 sort하여 출력(json 포멧 활용)](#1.4)<br>
  1.5. [클러스터 내 POD의 전체 레이블을 확인 후 배포된 POD중 레이블을 임의로 선택하고, 선택한 레이블을 사용하는 Pod들 중 CPU 소비율이 가장 높은 Pod의 이름을 찾아서 출력](#1.5)<br>


    <br>

## <div id='1'> 1. Monitoring

### <div id='1.1'> 1.1. kube-api 의 로그 모니터링 후 'error' 오류가 있는 로그 라인 추출(Extract)해서 /subtask/CUSTOM-LOG001 파일에 저장후 출력하세요.


```
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl logs -n kube-system -l component=kube-apiserver --tail=5 | grep -i 'error' > /subtask/CUSTOM-LOG001
```

```
ubuntu@qna-cluster-001:/subtask$ vi CUSTOM-LOG001

http2: server: error reading preface from client 172.16.1.12:44724: read tcp 172.16.1.12:6443->172.16.1.12:44724: read: connection reset by peer
http2: server: error reading preface from client 172.16.1.12:51108: read tcp 172.16.1.12:6443->172.16.1.12:51108: read: connection reset by peer
E0714 04:25:59.509756       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
http2: server: error reading preface from client 172.16.1.12:37930: read tcp 172.16.1.12:6443->172.16.1.12:37930: read: connection reset by peer
http2: server: error reading preface from client 172.16.1.12:51824: read tcp 172.16.1.12:6443->172.16.1.12:51824: read: connection reset by peer
http2: server: error reading preface from client 172.16.1.12:34092: read tcp 172.16.1.12:6443->172.16.1.12:34092: read: connection reset by peer
E0714 04:17:08.656016       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0714 04:17:08.656140       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0714 04:17:08.656202       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0711 05:15:10.814445       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0714 04:17:08.731566       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0714 04:17:08.735201       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0714 04:17:08.894659       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
E0714 04:25:59.551686       1 dispatcher.go:214] "Unhandled Error" err="failed calling webhook \"linkerd-proxy-injector.linkerd.io\": failed to call webhook: Post \"https://linkerd-proxy-injector.linkerd.svc:443/?timeout=10s\": service \"linkerd-proxy-injector\" not found" logger="UnhandledError"
```



### <div id='1.2'> 1.2. kubectl 명령을 사용하여 POD중 CPU를 가장 많이 사용하는 순서대로 출력하세요.


```
ubuntu@qna-cluster-001:~$ kubectl top pod --all-namespaces --sort-by=cpu
NAMESPACE        NAME                                              CPU(cores)   MEMORY(bytes)   
kube-system      kube-apiserver-qna-cluster-003                    23m          507Mi           
kube-system      kube-apiserver-qna-cluster-001                    18m          416Mi           
kube-system      kube-apiserver-qna-cluster-002                    18m          381Mi           
kube-system      calico-node-5g9hz                                 18m          188Mi           
kube-system      calico-node-nxb5n                                 14m          217Mi           
kube-system      calico-node-m5h66                                 13m          186Mi           
kube-system      calico-node-5tg49                                 12m          186Mi           
kube-system      kube-proxy-2h4tl                                  11m          28Mi            
kube-system      kube-controller-manager-qna-cluster-001           6m           76Mi            
kube-system      nodelocaldns-9sjv4                                3m           17Mi            
kube-system      kube-scheduler-qna-cluster-001                    2m           28Mi            
sy-test          open-webui-0                                      2m           938Mi           
metallb-system   speaker-t5nl6                                     2m           36Mi            
kube-system      calico-kube-controllers-5db5978889-ns8z7          2m           49Mi            
metallb-system   speaker-jzkks                                     2m           38Mi            
kube-system      metrics-server-6c8bff4c-cf6cx                     2m           31Mi            
kube-system      kube-scheduler-qna-cluster-003                    2m           40Mi            
kube-system      kube-scheduler-qna-cluster-002                    2m           48Mi            
kube-system      kube-controller-manager-qna-cluster-002           1m           43Mi            
default          nfs-subdir-external-provisioner-8989f7c84-pds7b   1m           20Mi            
kube-system      kube-proxy-h8lv7                                  1m           42Mi            
kube-system      kube-proxy-m9wx9                                  1m           42Mi            
kube-system      dns-autoscaler-5cb4578f5f-rpsvc                   1m           23Mi            
kube-system      kube-controller-manager-qna-cluster-003           1m           38Mi            
kube-system      coredns-d665d669-dzpzt                            1m           19Mi            
metallb-system   speaker-w86df                                     1m           26Mi            
kube-system      nodelocaldns-9mp4d                                1m           23Mi            
kube-system      kube-proxy-5s2fj                                  1m           44Mi            
kube-system      nodelocaldns-wn5ck                                1m           27Mi            
kube-system      nodelocaldns-xkfqm                                1m           25Mi            
metallb-system   controller-74b6dc8f85-n5bg2                       1m           34Mi            
metallb-system   speaker-8xtxw                                     1m           35Mi            
ingress-nginx    ingress-nginx-controller-59ccd79866-ftwdb         1m           70Mi            
kube-system      coredns-d665d669-6hqxx                            1m           35Mi            
ingress          ingress-nginx                                     0m           2Mi             
ingress          app-nginx                                         0m           2Mi     
```



### <div id='1.3'> 1.3. kubectl 명령을 사용하여 NODE중 메모리를 가장 적게 사용하는 순서대로 출력하세요.

1. MEMORY%가 적은 순
```
ubuntu@qna-cluster-001:~$  kubectl top node --sort-by=memory | sort -k5 -n
NAME              CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
qna-cluster-004   42m          3%     2739Mi          38%       
qna-cluster-003   69m          4%     1952Mi          64%       
qna-cluster-001   83m          5%     2019Mi          66%       
qna-cluster-002   95m          6%     2027Mi          66%      
```

2. MEMORY(bytes)가 적은 순
```
ubuntu@qna-cluster-001:~$ kubectl top node --sort-by=memory | tac
qna-cluster-003   82m          5%     1953Mi          64%       
qna-cluster-001   88m          6%     2023Mi          66%       
qna-cluster-002   71m          5%     2037Mi          66%       
qna-cluster-004   57m          4%     2742Mi          38%       
NAME              CPU(cores)   CPU%   MEMORY(bytes)   MEMORY% 
```


 

### <div id='1.4'> 1.4 kubectl 명령을 사용하여 클러스터에 구성된 모든 PV를 capacity별로 sort하여 출력하세요. (json 포멧 활용)

```
ubuntu@qna-cluster-001:~$ kubectl get pv --sort-by=.spec.capacity.storage
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM           STORAGECLASS      VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-e6d36418-497e-492f-9fde-67328981f16a   10Mi       RWX            Retain           Released    yang/pvc-yang   cp-storageclass   <unset>                          47h
pv001                                      1Gi        RWX            Retain           Available                                     <unset>                          28h
pv001-yang                                 1Gi        RWX            Retain           Available                                     <unset>                          47h
ubuntu@qna-cluster-001:~$ 

```


### <div id='1.5'> 1.5 kubectl 명령을 사용하여 클러스터 내 POD의 전체 레이블을 확인 후 배포된 POD중 레이블을 임의로 선택하고, 선택한 레이블을 사용하는 Pod들 중 CPU 소비율이 가장 높은 Pod의 이름을 찾아서 출력하세요.

```
# 레이블 확인
ubuntu@qna-cluster-001:~$ kubectl get pods --all-namespaces --show-labels
NAMESPACE        NAME                                              READY   STATUS    RESTARTS   AGE   LABELS
default          chat-app                                          1/1     Running   0          23h   run=chat-app
default          nfs-subdir-external-provisioner-8989f7c84-pds7b   1/1     Running   0          18d   app=nfs-subdir-external-provisioner,pod-template-hash=8989f7c84,release=nfs-subdir-external-provisioner
ingress-nginx    ingress-nginx-controller-6887c6d764-95r4d         1/1     Running   0          18d   app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,app.kubernetes.io/part-of=ingress-nginx,app.kubernetes.io/version=1.12.0,pod-template-hash=6887c6d764
kube-system      calico-kube-controllers-5db5978889-ns8z7          1/1     Running   7          20d   k8s-app=calico-kube-controllers,pod-template-hash=5db5978889
kube-system      calico-node-5g9hz                                 1/1     Running   4          21d   controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      calico-node-5tg49                                 1/1     Running   8          21d   controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      calico-node-m5h66                                 1/1     Running   8          21d   controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      calico-node-nxb5n                                 1/1     Running   8          21d   controller-revision-hash=65b4898996,k8s-app=calico-node,pod-template-generation=1
kube-system      coredns-d665d669-6hqxx                            1/1     Running   8          21d   k8s-app=kube-dns,pod-template-hash=d665d669
kube-system      coredns-d665d669-dzpzt                            1/1     Running   6          20d   k8s-app=kube-dns,pod-template-hash=d665d669
kube-system      dns-autoscaler-5cb4578f5f-rpsvc                   1/1     Running   8          21d   k8s-app=dns-autoscaler,pod-template-hash=5cb4578f5f
kube-system      kube-apiserver-qna-cluster-001                    1/1     Running   8          21d   component=kube-apiserver,tier=control-plane
kube-system      kube-apiserver-qna-cluster-002                    1/1     Running   8          21d   component=kube-apiserver,tier=control-plane
kube-system      kube-apiserver-qna-cluster-003                    1/1     Running   8          21d   component=kube-apiserver,tier=control-plane
kube-system      kube-controller-manager-qna-cluster-001           1/1     Running   11         21d   component=kube-controller-manager,tier=control-plane
kube-system      kube-controller-manager-qna-cluster-002           1/1     Running   12         21d   component=kube-controller-manager,tier=control-plane
kube-system      kube-controller-manager-qna-cluster-003           1/1     Running   11         21d   component=kube-controller-manager,tier=control-plane
kube-system      kube-proxy-2h4tl                                  1/1     Running   8          21d   controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-proxy-5s2fj                                  1/1     Running   4          21d   controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-proxy-h8lv7                                  1/1     Running   8          21d   controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-proxy-m9wx9                                  1/1     Running   8          21d   controller-revision-hash=d78758f9,k8s-app=kube-proxy,pod-template-generation=1
kube-system      kube-scheduler-qna-cluster-001                    1/1     Running   10         21d   component=kube-scheduler,tier=control-plane
kube-system      kube-scheduler-qna-cluster-002                    1/1     Running   10         21d   component=kube-scheduler,tier=control-plane
kube-system      kube-scheduler-qna-cluster-003                    1/1     Running   11         21d   component=kube-scheduler,tier=control-plane
kube-system      metrics-server-6c8bff4c-cf6cx                     1/1     Running   6          20d   app.kubernetes.io/name=metrics-server,pod-template-hash=6c8bff4c,version=v0.7.0
kube-system      nodelocaldns-9mp4d                                1/1     Running   10         21d   controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
kube-system      nodelocaldns-9sjv4                                1/1     Running   9          21d   controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
kube-system      nodelocaldns-wn5ck                                1/1     Running   4          21d   controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
kube-system      nodelocaldns-xkfqm                                1/1     Running   10         21d   controller-revision-hash=76c95d7d99,k8s-app=node-local-dns,pod-template-generation=1
linkerd-smi      smi-adaptor-78f8cfc865-7br66                      1/1     Running   0          18d   component=smi-adaptor,linkerd.io/extension=smi,pod-template-hash=78f8cfc865
metallb-system   controller-74b6dc8f85-n5bg2                       1/1     Running   0          18d   app=metallb,component=controller,pod-template-hash=74b6dc8f85
metallb-system   speaker-8xtxw                                     1/1     Running   8          21d   app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
metallb-system   speaker-jzkks                                     1/1     Running   8          21d   app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
metallb-system   speaker-t5nl6                                     1/1     Running   4          21d   app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
metallb-system   speaker-w86df                                     1/1     Running   8          21d   app=metallb,component=speaker,controller-revision-hash=87d554745,pod-template-generation=1
sy-subtask       nginx                                             2/2     Running   0          11d   run=nginx

# app=metallb 레이블을 사용하는 pod 중 가장 높은 CPU 소비 찾기

ubuntu@qna-cluster-001:~$ kubectl top pod --all-namespaces --selector=app=metallb --sort-by=cpu 
NAMESPACE        NAME                          CPU(cores)   MEMORY(bytes)   
metallb-system   speaker-8xtxw                 2m           36Mi            
metallb-system   speaker-t5nl6                 2m           18Mi            
metallb-system   speaker-w86df                 2m           23Mi            
metallb-system   controller-74b6dc8f85-n5bg2   1m           18Mi            
metallb-system   speaker-jzkks                 1m           36Mi            


```
