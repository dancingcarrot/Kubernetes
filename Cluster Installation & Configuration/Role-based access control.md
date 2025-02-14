## Table of Contents

1. [Role-based access control(RBAC)](#1)<br>
  1.1. [개념](#1.1)<br>
  

2. [Role-based access control](#2)<br>
  2.1. [namespace 생성](#2.1)<br>
  2.1.1. [Service Account 생성](#2.1.1)<br>
  2.1.2. [Role 생성](#2.1.2)<br>
  2.1.3. [RoleBinding 생성 후 확인](#2.1.3)<br>
  2.2. [ServiceAccount, ClusterRole, ClusterRoleBinding](#2.2)<br>
  2.2.1. [ClusterRole 생성](#2.2.1)<br>
  2.2.2. [ServiceAccount 생성](#2.2.2)<br>
  2.2.3. [ServiceAccount cicd-token에 바인딩 후 확인](#2.2.3)<br>




# <div id='1'> 1. Role-based access control(RBAC)

### <div id='1.1'> 1.1. 개념

> 역할 기반 접근 제어(Role Based AccessControl, RBAC)

컴퓨터 시스템 보안에서 권한이 있는 사용자들에게 시스템 접근을 통제하는 방법 중 하나이다.


<br><br>

# <div id='2'> 2. Role-based access control

  <br>

### <div id='2.1'> 2.1. namespace가 api-access인 것을 생성

```
ubuntu@ta-cluster-1:~$  kubectl create ns api-access
namespace/api-access created
```

### <div id='2.1.1'> 2.1.1 namespace에 pod-viewer라는 이름의 Service Account를 생성

```
ubuntu@ta-cluster-1:~$ kubectl create serviceaccount pod-view --namespace api-access
serviceaccount/pod-view created
```

### <div id='2.1.2'> 2.1.2 podreader-role이라는 이름의 Role을 생성
```
ubuntu@ta-cluster-1:~$ kubectl create role podreader-role --verb=get --verb=list --verb=watch --resource=pods --namespace=api-access
role.rbac.authorization.k8s.io/podreader-role created

```


### <div id='2.1.3'> 2.1.3 podreader-rolebinding이라는 이름의 RoleBinding을 생성 후 확인

```
ubuntu@ta-cluster-1:~$ kubectl create rolebinding podreader-rolebinding --role=podreader-role --serviceaccount=api-access:pod-viewer --namespace=api-access
rolebinding.rbac.authorization.k8s.io/podreader-rolebinding created



ubuntu@ta-cluster-1:~$ kubectl describe rolebinding podreader-rolebinding --namespace=api-access
Name:         podreader-rolebinding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  podreader-role
Subjects:
  Kind            Name        Namespace
  ----            ----        ---------
  ServiceAccount  pod-viewer  api-access

```  




<br>
<br>

## <div id='2.2'> 2.2. ServiceAccount, ClusterRole, ClusterRoleBinding


### <div id='2.2.1'> 2.2.1. deployment,statefulSet,daemonSet 리소스 타입에서만 create가 허용된 deployment-clusterrole이름을 가진 ClusterRole 생성

```
ubuntu@ta-cluster-1:~$ kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployment,statefulSet,daemonSet
clusterrole.rbac.authorization.k8s.io/deployment-clusterrole created
```

### <div id='2.2.2'> 2.2.2. 미리 생성된 namespace api-access에 cicd-token 이라는 새로운 ServiceAccount를 만듬
```
ubuntu@ta-cluster-1:~$ kubectl create serviceaccount cicd-token --namespace=api-access
serviceaccount/cicd-token created
```

### <div id='2.2.3'> 2.2.3. ClusterRole deployment-clusterrole을 namespace api-access 로 제한된 새 ServiceAccount cicd-token에 바인딩 후 확인

```
ubuntu@ta-cluster-1:~$ kubectl create clusterrolebinding deployment-clusterrolebinding --clusterrole=deployment-clusterrole --serviceaccount=api-access:cic-toekn
clusterrolebinding.rbac.authorization.k8s.io/deployment-clusterrolebinding created


ubuntu@ta-cluster-1:~$ kubectl describe clusterrolebindings deployment-clusterrolebinding
Name:         deployment-clusterrolebinding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  deployment-clusterrole
Subjects:
  Kind            Name       Namespace
  ----            ----       ---------
  ServiceAccount  cic-toekn  api-access



```  



