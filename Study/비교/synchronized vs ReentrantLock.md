## synchronized

멀티스레드 환경에서는 여러 스레드가 동시에 같은 자원에 접근할 경우, 예상치 못한 문제(데이터 손상, 충돌 등)가 발생할 수 있다. 이를 방지하기 위해 동기화(synchronized) 기법이 필요하다.

### 특징
- 스레드가 synchronized 키워드가 붙은 메서드에 진입하려면 해당 객체의 lock을 획득해야 한다.
- lock을 획득하지 못한 스레드는 **RUNNABLE** 상태에서 **BLOCKED** 상태로 전환된다. lock을 획득할 때까지 대기하며, 이 동안 CPU 실행 스케줄링에서 제외된다.
- 여러 스레드가 대기 중일 경우, lock 획득 순서는 보장되지 않는다.
- synchronized 블록 안에서는 변수의 메모리 가시성 문제가 자동으로 해결되므로, 별도의 volatile 선언이 필요하지 않다.

### 단점
#### 무한 대기
BLOCKED 상태의 스레드는 lock이 불릴 때까지 무한 대기를 하며 synchronized는 interrupt, timeout을 지원하지 않는다.
#### 공정성 문제
lock이 돌아왔을 때, BLOCKED 상태의 여러 스레드 중에 어떤 스레드가 lock을 획득할지 알 수 없다.

### 스레드의 상태
- NEW: 스레드가 생성되었지만 아직 시작되지 않은 상태
- RUNNABLE: 실행 중이거나 실행 준비가 완료된 상태
- BLOCKED: 동기화 락이 풀리기를 기다리는 상태 (synchronized 사용 시 발생)
- WAITING: 다른 스레드의 특정 작업이 완료되기를 무한정 기다리는 상태 (`wait()`, `join()` 호출 시)
- TIMED_WAITING: 특정 시간동안 대기하는 상태 (`sleep()`, `wait(timeout)`, `join(timeout)` 호출 시)
- TERMINATED: 스레드의 실행이 완료된 상태

BLOCKED & WAITING 은 모두 스레드가 대기하며 CPU 실행 스케줄링에 들어가지 않기 때문에, CPU에서 보면 실행하지 않는 비슷한 상태이지만 큰 차이가 있다.

- BLOCKED: 인터럽트가 걸려도 여전히 BLOCKED
	- synchronized에서 lock을 획득하기 위해 대기할 때 사용하는 특별한 상태이다.
- WAITING(TIME_WAITING): interrupt가 걸리면 RUNNABLE
	- 스레드가 특정 조건이나 시간 동안 대기할 때 발생하는 상태이다.

### synchronized vs static synchronized
- synchronized 키워드는 인스턴스마다 고유한 모니터 락을 사용한다.
	- 동일 인스턴스에서 동기화된 메서드를 호출할 경우, 스레드는 lock을 얻기 위해 BLOCKED 상태로 대기한다.
- static synchronized 키워드는 클래스 레벨의 모니터 락을 사용한다.
	- 서로 다른 인스턴스라도 동일한 lock을 공유하여, 한 스레드가 lock을 점유하면 다른 스레드는 대기해야 한다.