## 동기
### Non-pipelined 설계
- single-cycle 구현
	- 사이클 시간은 가장 느린 명령에 따라 달라진다.
	- 모든 명령에는 동일한 시간이 소요된다.
- Multi-cycle 구현
	- 명령어 실행을 여러 단계로 나눈다.
	- 각 명령어는 다양한 단계(클럭 사이클)을 취할 수 있다.
### Pipelined 설계
- 명령어 실행을 여러 단계로 나눈다 (stages).
- 여러 stage에서 서로 다른 명령어 실행을 겹치게 한다.
	- 각 사이클마다 다른 명령이 다른 단계에서 실행된다.
	- 예를 들어, 5-stage pipeline의 경우 (**F**etch-**D**ecode-**R**ead-**E**xecute-**W**rite)
		- 5개의 명령어가 5개의 서로 다른 파이프라인 stage에서 동시에 실행된다.
		- 매 사이클마다 하나의 명령어 실행을 완료한다 (매 5 사이클 대신)
		- 머신의 처리량을 5배 증가시킬 수 있다.

## 데이터 의존성 및 위험
### 데이터 의존성
- 쓰기 후 읽기 (RAW, Read-After-Write) 의존성
	- True dependence
	- 생산자가 데이터를 생산한 후에 데이터를 소비해야 한다.
- 쓰기 후 쓰기 (WAW, Write-After-Write) 의존성
	- Output dependence
	- 나중 명령어의 결과는 이전 명령어로 덮어쓸 수 있다.
- 읽기 후 쓰기 (WAR, Write-After-Read) 의존성
	- Anti dependence
	- 소비자 앞에 있는 값을 덮어쓰면 안된다.
- Notes
	- WAW & WAR은 스토리지 충돌로 인해 발생하는 false dependence라고 한다.
	- 레지스터와 메모리 위치 모두에 대해 세 가지 유형의 종속성이 모두 발생할 수 있다.
	- 프로그램의 특징(머신이 아님)
	- 올바른 출력을 생성하려면 실행 중에 보존되어야 한다.