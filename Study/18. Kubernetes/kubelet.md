https://tech.osci.kr/kubelet/
![[Pasted image 20250721014814.png]]
kubelet은 각 노드에서 실행되는 기본 **Node Agent**이다. 각 node에는 control plane과 통신하는 kubelet이 있다.

즉, kubelet은 노드에서 컨테이너가 동작하도록 관리해주는 핵심 요소이다.
**각 노드에서 pod를 생성하고 정상적으로 동작하는지 관리**하는 역할을 담당하고 있으며, 실제로 우리가 쿠버네티스의 워크로드를 관리하기 위해 내려지는 명령은 kubelet을 통해 수행된다고 볼 수 있다.

## 작동 방식

1. 쿠버네티스 pod를 관리하기 위해 YAML 파일을 작성한다.
2. kubectl 명령어를 통해 해당 명령을 쿠버네티스 클러스터에 적용한다.
3. 이때 YAML의 명령이 kube-apiserver로 전송된 후 kubelet으로 전달된다.
4. kubelet은 이 YAML을 통해 전달된 pod를 생성 혹은 변경하고, 이후 이 YAML에 명시된 컨테이너가 정상적으로 실행되고 있는지 확인한다.

## 구성요소

### CAdvisor
CAdvisor는 kubelet의 하위 프로세스로 컨테이너에 대한 저옵를 수집, 처리 및 전달하는 데몬이다. CAdvisor에서 수집된 정보는 kubelet으로 전달되고 이는 클러스터 상태 모니터링에 사용된다.

### Kubelet APIserver
kubelet APIserver를 통해 Kubelet으로 들어온 요청이 처리된다.

### Kubeconfig
kubelet이 ControlPlane과 통신하는 데 사용하는 인증 정보 파일로, 클러스터의 APIserver 주소, 클러스터 정보 등이 명시되어 있다.

### OOM Monitor
kubelet이 pod의 OOM(Out-Of-Memory)를 처리하는데 사용한다. OOM 발생 시 해당 pod를 중지하거나 다시 시작하여 노드의 리소스 안정성을 보장한다.


## 기능

### Pod 관리
kubelet은 kube-apiserver와 통신하여, pod 정보(PodSpecs)를 전달 받아 그 조건에 맞게 pod를 실행하고, pod의 상태를 주기적으로 모니터링한다.
만약 pod에 문제가 발생할 경우 pod를 다시 시작하거나 스케줄링하여 안정성을 유지한다. 어떤 node에 pod를 배치할지는 kube-scheduler가 결정하지만, 실제로 Container Runtime에 배치를 명령하는 것은 kubelet에서 수행한다.

### Container 관리
kubelet은 Container Runtime을 사용하여 PodSpec에 따라 컨테이너 이미지 pull, 리소스 할당, Container 시작 및 중지 등 Container의 상태를 유지/관리한다.

### 리소스 관리 및 모니터링
kubelet은 노드의 리소스 사용량을 모니터링하여 클러스터의 상태를 파악하고, 이를 Control Plane에 전달한다. 해당 정보는 pod 스케줄링 결정에 사용된다. 또한 pod에 대한 CPU 및 메모리 공급 등의 리소스 할당 프로세스를 처리하여, 공평한 분배를 보장하고 리소스 충돌읠 최소화한다.

### 상태 보고
kubelet은 주기적으로 node와 pod의 상태를 Control Plane에 전달하고 이 정보를 통해 Control Plane은 클러스터 전체 상태 및 node와 pod 상태를 모니터링 할 수 있다.

### node 자동 복구
kubelet은 node가 비정상적으로 종료된 경우 자동 복구를 시도한다. 이를 통해 node의 가용성 및 클러스터 안정성을 유지한다.

### 볼륨 관리
kubelet은 pod 내 볼륨 마운트 조정자로서 pod 재시작 및 재스케줄링 전반에 걸쳐 액세스 연속성을 유지하여 일관된 데이터 스토리지를 보장한다.

### 네트워킹
kubelet은 pod 내의 컨테이너에 대한 네트워크를 설정하여 동일한 노드에 있는 컨테이너 간 및 클러스터의 노드 간에 통신을 가능하게 한다.

### pod 보안 정책
kubelet은 node에서 실행되는 워크로드의 보안을 보장하기 위해 쿠버네티스 관리자가 설정한 pod 보안 정책에 따라 pod를 생성 및 접근할 수 있도록 한다.