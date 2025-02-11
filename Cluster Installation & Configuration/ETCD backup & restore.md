


## Table of Contents

1. [ETCD](#1)<br>
  1.1. [개념](#1.1)<br>
  

2. [ETCD backup & restore](#2)<br>
  2.1. [ETCD backup](#2.1)<br>
  2.1.1. [etcd 백업에 사용 되는 키 찾기](#2.1.1)<br>
  2.1.2. [스냅샷 저장](#2.1.2)<br>
  2.1.3. [임의의 application 배포](#2.1.3)<br>
  2.2. [restore](#2.2)<br>
  2.2.1 [데이터가 복구될 디렉토리를 지정](#2.2.1)<br>
  2.2.2 [기존의 이전 스냅샷을 복원하기 위해 etcd 파일 수정](#2.2.2)<br>
  2.2.3 [etcd 시스템 재시작](#2.2.3)<br>




## <div id='1'> 1. ETCD

### <div id='1.1'> 1.1. 개념
A distributed, reliable key-value store for the most critical data of a distributed system
<br>
<분산시스템에서 중요한 데이터를 저장할 때 사용할 수 있는, 믿을 수 있는 key value 분산 저장소>

- coreOS가 만든 분산 key:value 형태의 데이터 스토리지
- Kubernetes Cluster의 정보를 저장(memory)해서 사용
> etcd가 저장하는 정보들 
 - nodes
 - pods
 - configs
 - secrets
 - accounts
 - roles
 - bindings
<br>

- 모든 etcd 데이터는 etcd 데이터베이스 파일에 보관(/var/lib/etcd)

`kubectl get pod`와 같은 command들은 모두 etcd로부터 받는 정보이다.


<br><br>

## <div id='2'> 2. ETCD backup & restore


### <div id='2.1'> 2.1. ETCD backup

#### <div id='2.1.1'> 2.1.1. etcd 백업에 사용 되는 키 찾기
```
$ sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
    - --etcd-cafile=/etc/ssl/etcd/ssl/ca.pem
    - --etcd-certfile=/etc/ssl/etcd/ssl/node-ta-task-cluster-1.pem
    - --etcd-compaction-interval=5m0s
    - --etcd-keyfile=/etc/ssl/etcd/ssl/node-ta-task-cluster-1-key.pem
    - --etcd-servers=https://10.200.50.84:2379
    - --storage-backend=etcd3
    - mountPath: /etc/ssl/etcd/ssl
      name: etcd-certs-0
      path: /etc/ssl/etcd/ssl
    name: etcd-certs-0

```
<br>

#### <div id='2.1.2'> 2.1.2. 스냅샷 저장

```
$ sudo ETCDCTL_API=3 etcdctl --debug \
  --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/ssl/etcd/ssl/ca.pem \
        --cert=/etc/ssl/etcd/ssl/node-ta-task-cluster-1.pem\
        --key=/etc/ssl/etcd/ssl/node-ta-task-cluster-1-key.pem \
        snapshot save /data/etcd-snapshot.db
.
.
.


Snapshot saved at /data/etcd-snapshot.db

```

#### <div id='2.1.3'> 2.1.3. 임의의 nginx application 배포

```
$ kubectl run nginx --image=nginx 

ubuntu@ta-task-cluster-1:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS       AGE
nfs-pod-provisioner-668f59bd7f-747gt   1/1     Running   4 (4d8h ago)   4d8h
nginx                                  1/1     Running   0              11s

```

### <div id='2.2'> 2.2. ETCD restore

#### <div id='2.2.1'> 2.2.1. 데이터가 복구될 디렉토리를 지정

```
$ sudo ETCDCTL_API=3 etcdctl \
  --data-dir=/var/lib/etcd-restore \
  snapshot restore /data/etcd-snapshot.db




ubuntu@ta-task-cluster-1:~$ sudo ETCDCTL_API=3 etcdctl \
  --data-dir=/var/lib/etcd-restore \
  snapshot restore /data/etcd-snapshot.db
Deprecated: Use `etcdutl snapshot restore` instead.

2025-02-11T08:37:17Z	info	snapshot/v3_snapshot.go:260	restoring snapshot	{"path": "/data/etcd-snapshot.db", "wal-dir": "/var/lib/etcd-restore/member/wal", "data-dir": "/var/lib/etcd-restore", "snap-dir": "/var/lib/etcd-restore/member/snap"}
2025-02-11T08:37:17Z	info	membership/store.go:141	Trimming membership information from the backend...
2025-02-11T08:37:18Z	info	membership/cluster.go:421	added member	{"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"]}
2025-02-11T08:37:18Z	info	snapshot/v3_snapshot.go:287	restored snapshot	{"path": "/data/etcd-snapshot.db", "wal-dir": "/var/lib/etcd-restore/member/wal", "data-dir": "/var/lib/etcd-restore", "snap-dir": "/var/lib/etcd-restore/member/snap"}

```

#### <div id='2.2.2'> 2.2.2. 기존의 이전 스냅샷을 복원하기 위해 etcd 파일 수정

```

$ cd /etc

$ sudo vi etcd.env

 ETCD_DATA_DIR=/var/lib/etcd

해당 부분의 경로를 복구될 디렉토리로 수정해준다.

 ETCD_DATA_DIR=/var/lib/etcd-restore

```

#### <div id='2.2.3'> 2.2.3. etcd 시스템 재시작

```
$ sudo systemctl restart etcd

ubuntu@ta-task-cluster-1:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS       AGE
nfs-pod-provisioner-668f59bd7f-747gt   1/1     Running   4 (4d8h ago)   4d8h

```

> nginx가 배포되기 전 상태로 복구된 것을 확인할 수 있다.
