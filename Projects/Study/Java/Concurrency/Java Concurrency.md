
## LockSupport

### synchronized 단점
- 무한 대기: `BLOCKED` 상태의 스레드는 락이 풀릴 때까지 무한 대기한다.
	- 특정 시간까지만 대기하는 타임아웃X
	- 중간에 인터럽트X
- 공정성: 락이 돌아왔을 때 `BLOCKED` 상태의 여러 스레드 중에 어떤 스레드가 락을 획득할 지 알 수 없다. 최악의 경우 특정 스레드가 너무 오랜 기간 락을 획득하지 못할 수 있다.

결국 더 유연하고, 더 세밀한 제어가 가능한 방법들이 필요하게 되었다.
이런 문제를 해결하기 위해 자바 1.5부터 `java.util.concurrent`라는 동시성 문제 해결을 위한 라이브러리 패키지가 추가된다.

이 라이브러리에는 수 많은 클래스가 있지만, 가장 기본이 되는 `LockSupport`에 대해서 먼저 알아보자.
`LockSupport`를 사용하면 `synchronized`의 가장 큰 단점인 무한 대기 문제를 해결할 수 있다.

### 기능
`LockSupport`는 스레드를 `WAITING`상태로 변경한다.
`WAITING`상태는 누가 깨워주기 전까지는 계속 대기한다. 그리고 CPU 실행 스케줄링에 들어가지 않는다.

`LockSupport`의 대표적인 기능은 다음과 같다.
- `park()`: 스레드를 `WAITING`상태로 변경한다.
	- 스레드를 대기 상태로 둔다. 참고로 `park`의 뜻이 "주차하다", "두다"라는 뜻이다.
- `parkNano(nanos)`: 스레드를 나노초 동안만 `TIMED_WAITING`상태로 변경한다.
	- 지정한 나노초가 지나면 `TIMED_WAITING`상태에서 빠져나오고 `RUNNABLE`상태로 변경된다.
- `unpoart(thread)`: WAITING 상태의 대상 스레드를 `RUNNABLE`상태로 변경한다.

![[Pasted image 20240924231713.png]]


## 생산자 소비자 문제
![[Pasted image 20240924232906.png]]
![[Pasted image 20240924232924.png]]
![[Pasted image 20240924232943.png]]
![[Pasted image 20240924232950.png]]