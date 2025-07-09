1. [HostPath](#1)<br>
  1.1. [HostPath Volume](#1.1)<br>
  1.2. [HostPath Volume 구성](#1.2)<br>


  <br>

## <div id='1'> 1. HostPath

### <div id='1.1'> 1.1. HostPath Volume 

(Node 단위의 데이터 공유)
> 쿠버네티스 노드의 파일 시스템에 있는 특정 경로를 Pod에 마운트하는 볼륨 타입이다.


### <div id='1.2'> 1.2. HostPath Volume 구성

```
> fluentd.yaml 파일에 다음 조건에 맞게 볼륨 마운트를 설정하시오.
> Worker node의 도커 컨테이너 디렉토리를 동일 디렉토리로 pod에 마운트 하시오.
> Worker node의 /var/log 디렉토리를 fluentd Pod에 동일이름의 디렉토리 마운트하시오.
```

```
# fluentd.yaml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd
```


1. volume mount 설정

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd
        volumeMounts:
        - mountPath: /var/lib/docker/containers
          name: docker-vol
        - mountPath: /var/log
          name: docker-log
      volumes:
      - name: docker-vol
        hostPath:
          path: /var/lib/docker/containers
          type: DirectoryOrCreate
      - name: docker-log
        hostPath:
          path: /var/log
          type: DirectoryOrCreate

```
2. describe 명령어로 마운트 확인
```
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl describe pod fluentd-qds7l -n subtask
```

![alt text](image.png)
![alt text](image-1.png)
