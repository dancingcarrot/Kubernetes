1. [PV(Persistent Volume) & PVC(Persistent Volume Claim)](#1)<br>
  1.1. [pv & pvc](#1.1)<br>
  1.2. [PersistentVolume 생성](#1.2)<br>
  1.3. [PVC를 사용하는 애플리케이션 Pod 구성](#1.3)<br>


  <br>

## <div id='1'> 1. PV(Persistent Volume) & PVC(Persistent Volume Claim)

### <div id='1.1'> 1.1. pv & pvc

> pv : 데이터를 저장할 볼륨. 볼륨을 생성하고 이를 클러스터에 등록한 것

> PVC: 필요한 저장 공간·RW모드 등 요청사항을 기술한 명세로서 PV에 전달하는 요청.
PV와 바인딩을 하는 목적으로 사용

(https://velog.io/@_zero_/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-PVPVC-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EC%84%A4%EC%A0%95)


### <div id='1.2'> 1.2. PersistentVolume 생성

```
> pv001라는 이름으로 size 1Gi, access mode ReadWriteMany를 사용하여 persistent volume을 생성합니다.
> volume type은 hostPath이고 위치는 /tmp/app-config입니다.
```

1. pv.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /tmp/app-config
```

### <div id='1.3'> 1.3. PVC를 사용하는 애플리케이션 Pod 구성

 다음의 조건에 맞는 새로운 PersistentVolumeClaim 생성하세요.

```
> • Name: pv-volume
> • Class: app-hostpath-sc
> • Capacity: 10Mi
> 생성한 pv-volume PersistentVolumeClaim을 mount하는 Pod 를 생성하세요.
> • Name: web-server-pod
> • Image: nginx
> • Mount path: /usr/share/nginx/html
> Volume에서 ReadWriteMany 액세스 권한을 가지도록 구성합니다.
```

1. pvc 생성

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
  storageClassName: cp-storageclass   # 작업중인 환경에서 사용하고 있는 스토리지로 수정

```

2. pv 생성
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /usr/share/nginx/pvc-test
```

3. pod 생성
```
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
spec:
  containers:
  - name: web
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/pvc-test
      name: pvc-test
  volumes:
  - name: pvc-test
    persistentVolumeClaim:
      claimName: pv-volume

```

4. 확인

```
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl get pods -n subtask
NAME             READY   STATUS    RESTARTS   AGE
web-server-pod   1/1     Running   0          9m27s


ubuntu@qna-cluster-001:~/workspace/sun$ kubectl describe pod web-server-pod -n subtask


...
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from pvc-test (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wggsw (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 

...

Volumes:
  pvc-test:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  pv-volume
    ReadOnly:   false
...

```
