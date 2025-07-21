## kubectl apply
`kubectl apply` 명령어는 kubernetes 리소스의 구성을 선언적으로 관리하는 데 사용된다. 이 명령어는 리소스의 현재 상태를 JSON 또는 YAML 파일 형식으로 지정된 원하는 상태와 비교하여, 필요한 변경사항을 적용한다.

### 주요 특징
- **변경사항만 적용**: 기존 리소스의 설정을 수정하거나 추가할 때 유용하다.
- **선언적 업데이트**: 리소스의 전체 정의를 제공하고 Kubernetes가 필요한 변경을 파악해 적용한다.

### 예시
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-deployment
spec:
  replicas: 3
  ...
```

```bash
kubectl apply -f sample-deployment.yaml
```

## kubectl wait
`kubectl wait` 명령어는 특정 리소스가 특정 조건을 만족할 때까지 대기하는 명령어이다. 이 명령어를 사용하면, 리소스의 상태를 주기적으로 체크하여 지정된 조건이 충족될 때까지 기다릴 수 있다.

### 사용 방법
```
kubectl wait [리소스 종류] [리소스 이름] --for=[조건] [조건 값]
```
#### 옵션
- --for: 대기할 조건을 지정한다.
	- `kubectl wait pod my-pod --for=condition=Ready`
- --timeout: 대기 시간을 지정한다. 이 시간을 초과하면 명령어가 종료된다.
	- `kubectl wait pod my-pod --for=condition=Ready --timeout=60s`
- --namespace: 네임스페이스를 지정한다.
	- `kubectl wait pod my-pod --for=condition=Ready --timeout=60s --namespace=my-namespace`
- etc.

### 예시
```
kubectl wait pod my-pod --for=condition=Ready
```
my-pod 파드가 Ready 상태가 될 때까지 대기할 수 있다.
