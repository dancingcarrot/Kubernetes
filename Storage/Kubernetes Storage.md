1. [Storage](#1)<br>
  1.1. [Kubernetes Stroage](#1.1)<br>
  1.1.1. [Kubernetes Stroage 구조 및 개념](#1.1.1)<br>




<br>

## <div id='1'> 1. Storage

### <div id='1.1'> 1.1. Kubernetes Stroage

#### <div id='1.1.1'> 1.1.1. Kubernetes Stroage 구조 및 개념

> 볼륨
- 데이터를 포함하고 있는 디렉터리이며, 파드의 컨테이너에서 접근 가능하다.

<details>
<summary> 볼륨 유형 </summary>

- cephfs
  - cephfs 볼륨은 기존 CephFS 볼륨을 파드에 마운트 할 수 있다. 
  - 파드를 제거할 때 지워지는 emptyDir 과는 다르게 cepfhs 볼륨의 내용은 유지되고, 볼륨은 그저 마운트 해제만 된다.
  - cephfs 볼륨에 데이터를 미리 채울 수 있으며, 해당 데이터는 파드 간에 공유될 수 있다.
  - cephfs 볼륨은 여러 작성자가 동시에 마운트 할 수 있다.

- 컨피그맵(configMap)
  - 구성 데이터를 파드에 주입하는 방법을 제공한다.
  - 컨피그맵에 저장된 데이터는 configmap 유형의 볼륨에서 참조되고 그런 다음에 파드에서 실행되는 컨테이너화된 애플리케이션이 소비한다.
  - 컨피그맵을 참조할 때, 볼륨에 컨피그맵의 이름을 제공한다.
  - 컨피그맵의 특정 항목에 사용할 경로를 사용자 정의할 수 있다.

- emptyDir
  - 파드가 노드에 할당될 때 처음 생성되며, 해당 노드에서 파드가 실행되는 동안에만 존재한다.
  - emptyDir 볼륨은 처음에는 비어있다.
  - 파드 내 모든 컨테이너는 emptyDir 볼륨에서 동일한 파일을 읽고 쓸 수 있지만, 해당 볼륨은 각각의 컨테이너에서 동일하거나 다른 경로에 마운트 될 수 있다. 
  - 노드에서 파드가 제거되면 emptyDir의 데이터가 영구적으로 삭제된다.

   
- hostPath <p>
  - ` < HostPath  볼륨에는 많은 보안 위험이 있으면, 가능하면 HostPath를 사용하지 않는 것이 좋다. HostPath 볼륨을 사용해야하는 경우, 
  필요한 파일 또는 디렉터리로만 범위를 지정하고 ReadOnly로 마운트해야 한다.>`
  - hostPath 볼륨은 호스트 노드의 파일시스템에 있는 파일이나 디렉터리를 파드에 마운트 한다.

- local 
  - local 볼륨은 디스크, 파티션 또는 디렉터리 같은 마운트된 로컬 스토리지 장치를 나타낸다.
  - 로컬 볼륨은 정상적으로 생성된 퍼시스턴트볼륨으로만 사용할 수 있다. 동적으로 프로비저닝된 것은 지원되지 않는다.

- nfs
  - nfs 볼륨을 사용하면 기존 NFS(네트워크 파일 시스템) 볼륨을 파드에 마운트 할 수 있다.
  - 파드를 제거할 때 지워지는 emptyDir과는 다르게 nfs 볼륨의 내용은 유지되고, 볼륨은 그저 마운트 해제만 된다.
  (이 의미는 NFS 볼륨에 데이터를 미리 채울 수 있으며, 파드 간에 데이터를 공유할 수 있다는 뜻이다.)
  - NFS는 여러 작성자가 동시에 마운트 할 수 있다.

- persistentVolumeClaim
  - pvc 볼륨은 퍼시스턴트 볼륨을 파드에 마운트하는데 사용한다.
  - 퍼시스턴트볼륨클레임은 사용자가 특정 클라우드 환경의 세부 내용을 몰라도 내구성이 있는 스토리지를 "클레임" 할 수 있는 방법이다.

- secret
  - secret 볼륨은 암호와 같은 민감한 정보를 파드에 전달하는데 사용된다.
  - 쿠버네티스 API에 시크릿을 저장하고 쿠버네티스에 직접적으로 연결하지 않고도 파드에서 사용할 수 있도록 파일로 마운트 할 수 있다.
  - secret 볼륨은 tmpfs로 지원되기 때문에 비 휘발성 스토리지에 절대 기록되지 않는다.
</details>

<br>

> 스토리지 
  - 스토리지는 파드의 수명과 별개로 데이터를 저장하거나, 파드 간 데이터를 공유할 수 있도록 하는 구조이다.

  - Volume: 파드에 마운트되어 데이터를 저장하는 추상적인 저장소입니다.
  - Persistent Volume (PV): 클러스터에 존재하는 실제 저장소 리소스입니다.
  - Persistent Volume Claim (PVC): 사용자가 필요한 저장 공간을 요청하는 리소스입니다.
  - StorageClass: 동적 볼륨 프로비저닝을 위한 설정 정보를 담고 있습니다.

