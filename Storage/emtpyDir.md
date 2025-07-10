1. [emptyDir](#1)<br>
  1.1. [emptyDir Volume](#1.1)<br>
  1.2. [emptyDir Volume을 공유하는 multi-pod 구성](#1.2)<br>


  <br>

## <div id='1'> 1. emptyDir

### <div id='1.1'> 1.1. emptyDir Volume

(pod 단위의 데이터 공유)
```
> emptyDir은 파드가 노드에 생성될 때 빈 디렉토리로 생성되며, 파드 내 컨테이너 간에 데이터를 공유할 수 있는 가장 기본적인 볼륨이다.
> 파드가 삭제되면 emptyDir의 데이터도 함꼐 사라진다.
```

> 특징
  - 파드 내 여러 컨테이너 간 데이터 공유 가능
  - 파드 단위로 존재하며, 파드가 삭제되면 데이터도 삭제됨
  - 주로 캐시, 로그, 임시 파일 등에 사용됨

### <div id='1.2'> 1.2. emptyDir Volume을 공유하는 multi-pod 구성

> 다음 조건에 맞춰서 nginx 웹서버 pod가 생성한 로그파일을 받아서 STDOUT으로 출력하는 busybox 컨테이너를 구성 하세요.<p>
(하나의 deployment에 2개의 container로 구성해야 합니다.)

```
> deployment Name: weblog
> Web container:
> • Image: nginx:1.17
> • Volume mount : /var/log/nginx
> • readwrite
> Log container:
> • Image: busybox
> • Command: /bin/sh, -c, "tail -n+1 -f /data/access.log"
> • Volume mount : /data
> • readonly
> emptyDir 볼륨을 통한 데이터 공유가 이루어져야 합니다.
```

1. multi-pod.yaml 작성
```
apiVersion: v1
kind: Pod
metadata:
  name: weblog
spec:
  containers:
  - image: nginx:1.17
    name: web
    volumeMounts:
    - mountPath: /var/log/nginx
      name: log-volume
  - image: busybox
    name: log
    command: [ "/bin/sh", "-c", "tail -n+1 -f /data/access.log" ]
    volumeMounts:
    - mountPath: /data
      name: log-volume
      readOnly: true
  volumes:
  - name: log-volume
    emptyDir: {}
```
2. 적용
```
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl apply -f multi-pod.yaml  -n subtask
Warning: would violate PodSecurity "restricted:v1.31": allowPrivilegeEscalation != false (containers "web", "log" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (containers "web", "log" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or containers "web", "log" must set securityContext.runAsNonRoot=true), seccompProfile (pod or containers "web", "log" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
pod/weblog created

ubuntu@qna-cluster-001:~/workspace/sun$ kubectl get pods -owide -n subtask
NAME     READY   STATUS    RESTARTS   AGE   IP              NODE              NOMINATED NODE   READINESS GATES
weblog   2/2     Running   0          8s    10.233.90.160   qna-cluster-004   <none>           <none>

```

3. 테스트
```
# 초기에는 로그가 없음
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl logs -f weblog -n subtask -c log

# curl 명령어 후 로그 확인
ubuntu@qna-cluster-001:~/workspace/sun$ curl http://10.233.90.160
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

ubuntu@qna-cluster-001:~/workspace/sun$ kubectl logs -f weblog -n subtask -c log
10.233.123.0 - - [07/Jul/2025:05:08:40 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.81.0" "-"

```
3-2. emptyDir 공유 테스트

```
# web pod에 `volume test` 작성

ubuntu@qna-cluster-001:~/workspace/sun$ kubectl exec -it weblog -c web -n subtask -- /bin/sh
# echo volume test >> /var/log/nginx/access.log
# exit


# log pod에서 `volume test` 확인
 
ubuntu@qna-cluster-001:~/workspace/sun$ kubectl exec -it weblog -c log -n subtask -- /bin/sh
/data # cat access.log
10.233.123.0 - - [07/Jul/2025:05:08:40 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.81.0" "-"
volume test

```






