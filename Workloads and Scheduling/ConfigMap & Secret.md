## Table of Contents

1. [ConfigMap & Secret](#1)<br>
  1.1. [Option](#1.1)<br>
  1.1.1. [Configmap](#1.1.1)<br>
  1.1.2. [Secret](#1.1.2)<br>

2. [OpenSSL을 이용하여 TLS 생성 후 Secret 생성](#2)<br>
  2.1. [private key 생성](#2.1)<br>
  2.2. [private key를 이용한 csr 생성](#2.2)<br>
  2.3. [crt 인증서 생성](#2.3)<br>
  2.4. [crt 파일을 pem 파일로 변환](#2.4)<br>
  2.5. [secret 생성 및 확인](#2.5)<br>

3. [configmap 등록](#3)<br>
  3.1. [webserver configmap 생성](#3.1)<br>
  3.2. [Secret 생성](#3.2)<br>
  3.3. [nginx 컨테이너 생성](#3.3)<br>
  3.4. [파일을 수정하여 configmap 등록 후 배포](#3.4)<br>
  3.5. [확인](#3.5)<br>

<br>
<br>


# <div id='1'> 1. ConfigMap & Secret

### <div id='1.1'> 1.1. Option

#### <div id='1.1.1'> 1.1.1. kubectl create Configmap
> kubectl create configmap [configmap 이름] --from-literal=[키]=[값]

#### <div id='1.1.2'> 1.1.2. kubectl create Secret
> kubectl create secret [secret 종류] [secret 이름] [flags][options]

<br>
<br>

# <div id='2'> 2. OpenSSL을 이용하여 TLS 생성 후 Secret 생성

### <div id='2.1'> 2.1. private key 생성

```
ubuntu@qna-cluster-1:~$ openssl genrsa -out subtask.key 2048
```

<details>
<summary> public key 생성 방법 </summary>
$ openssl rsa -in subtask.key -pubout -out publicSubtask.key
</details>

### <div id='2.2'> 2.2. private key를 이용한 csr 생성
```
ubuntu@qna-cluster-1:~$ openssl req -new -key subtask.key -out subtask.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

### <div id='2.3'> 2.3. crt 인증서 생성

```
ubuntu@qna-cluster-1:~$ openssl req -x509 -days 365 -key subtask.key -in subtask.csr -out subtask.crt -days 365
Warning: No -copy_extensions given; ignoring any extensions in the request
```

### <div id='2.4'> 2.4. crt 파일을 pem 파일로 변환
```
ubuntu@qna-cluster-1:~$ openssl x509 -in subtask.crt -out subtask.pem -outform PEM
```


### <div id='2.5'> 2.5. secret 생성 및 확인
```
>> secret 생성
ubuntu@qna-cluster-1:~$ kubectl create secret tls subtask-secret --key subtask.key --cert subtask.crt
secret/subtask-secret created

>> secret 생성 확인
ubuntu@qna-cluster-1:~$ kubectl get secret
NAME             TYPE                DATA   AGE
subtask-secret   kubernetes.io/tls   2      12s

ubuntu@qna-cluster-1:~$ kubectl describe secret subtask-secret
Name:         subtask-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1245 bytes
tls.key:  1704 bytes

```

# <div id='3'> 3. configmap 등록

아래의 변수를 configMap 으로 등록한 후 nginx 컨테이너에 환경변수로 할당하세요. 다음, 컨테이너에 접근하여 configmap 환경변수와 secret 을 출력하세요.
> DBNAME: mysql <br>
> USER: admin  <br>


### <div id='3.1'> 3.1. webserver configmap 생성
``` 
ubuntu@qna-cluster-1:~$ kubectl create configmap webserver --from-literal=DBNAME=mysql --from-literal=USER=admin
configmap/webserver created

ubuntu@qna-cluster-1:~$ kubectl get configmap
NAME                    DATA   AGE
webserver               2      17s

ubuntu@qna-cluster-1:~$ kubectl describe configmap webserver
Name:         webserver
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DBNAME:
----
mysql
USER:
----
admin

BinaryData
====
```
<br>

### <div id='3.2'> 3.2. secret 생성
```
kubectl create secret generic sun-secret --from-literal=PASSWORD=adminsecret
secret/sun-secret created
```


### <div id='3.3'> 3.3. nginx 컨테이너 생성

```
ubuntu@qna-cluster-1:~$ kubectl run webserver-configmap --image=nginx --dry-run=client -o yaml > webserver-configmap.yaml
```

```
ubuntu@qna-cluster-1:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
webserver-configmap                    1/1     Running   0          12s

```
### <div id='3.4'> 3.4. 파일을 수정하여 configmap 등록 후 배포

```
ubuntu@qna-cluster-1:~$ vi webserver-configmap.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
  name: webserver-configmap
spec:
  containers:
  - image: nginx
    name: webserver-configmap
    envFrom:
      - configMapRef:
          name: webserver
    env:
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: sun-secret
          key: PASSWORD

```


### <div id='3.5'> 3.5. 확인
```
ubuntu@qna-cluster-1:~$ kubectl exec -it webserver-configmap -- /bin/bash 
root@webserver-configmap:/# env
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=webserver-configmap
DBNAME=mysql  <DBNAME 확인>
PWD=/
PKG_RELEASE=1~bookworm
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.233.0.1:443
DYNPKG_RELEASE=1~bookworm
PASSWORD=adminsecret    <PASSWORD 확인>
NJS_VERSION=0.8.9
TERM=xterm
USER=admin      <USER 확인>
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.233.0.1
KUBERNETES_SERVICE_HOST=10.233.0.1
KUBERNETES_PORT=tcp://10.233.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NGINX_VERSION=1.27.4
NJS_RELEASE=1~bookworm
_=/usr/bin/env


ubuntu@qna-cluster-1:~$ kubectl exec -it webserver-configmap -- env | grep DBNAME
DBNAME=mysql
ubuntu@qna-cluster-1:~$ kubectl exec -it webserver-configmap -- env | grep USER
USER=admin
ubuntu@qna-cluster-1:~$ kubectl exec -it webserver-configmap -- env | grep PASSWORD
PASSWORD=adminsecret


```
