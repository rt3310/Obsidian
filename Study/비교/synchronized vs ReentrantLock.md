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

## ReentrantLock

synchronized 키워드는 동기화를 제공하지만, 무한 대기와 비공정성이라는 단점이 있다. 이러한 문제를 해결하기 위해 Java5 부터 ReentrantLock이 도입되었다.
ReentrantLock은 LockSupport를 활용하여 위 단점을 극복한다.
```java
Lock lock = new ReentrantLock();

try {
	lock.lock(); // lock 획득
} finally {
	lock.unlock(); // lock 해제
}
```

### 무한 대기 극복
LockSupport는 concurrent 패키지에 위치하며, 저수준의 객체이다.
LockSupport는 스레드가 lock을 획득하려고 대기할 때, 스레드의 상태를 BLOCKED 대신 WAITING으로 변경해서 interrupt를 허용하게 해준다. 이를 통해 synchronized의 무한 대기 문제를 간단하게 해결한다.

### 공정성 극복
ReentrantLock은 공정 모드와 비공정 모드를 제공하며, 필요에 따라 선택할 수 있다.
#### 공정 모드
```java
Lock lock = new ReentrantLock(true); // 공정 모드 설정
```
**장점**
- 대기 큐에 있는 스레드가 lock을 요청한 순서대로 lock을 획득한다.
- 모든 스레드는 언젠가는 lock을 획득할 수 있다(기아 현상 방지).
**단점**
- 대기 큐 관리로 인해 성능이 저하될 수 있다.
#### 비공정 모드
```java
Lock lock = new ReentrantLock(); // 기본은 비공정 모드
```
- 성능을 중시하는 환경에서 사용된다.
- 비공정 모드이지만, ReentrantLock은 내부적으로 큐로 구현되어 있기 때문에 스레드 간 경합이 크지 않은 이상 순서대로 처리된다.

### 주요 함수
#### `void lock()`
- lock을 획득한다. 만약, 다른 스레드가 이미 lock을 획득했다면 lock이 풀릴 때까지 스레드는 WAITING 상태가 된다. 하지만 interrupt에 응답하지 않는다.
	- 내부 구현에서 interrupt가 발생하면 순간 WAITING 상태를 빠져나와서 RUNNABLE 상태가 되지만, 다시 해당 스레드를 WAITING 상태로 강제로 변경해버린다.
- 이 lock은 모니터 lock이 아니고, Lock 인터페이스와 ReentrantLock이 제공하는 기능이며, 모니터 lock과 BLOCKED 상태는 synchronized 에서만 사용된다.
#### `void lockInterruptibly()`
- lock 획득을 시도하되, 다른 스레드가 interrupt를 발생시켜 WAITING 상태에서 벗어나서 lock 획득을 포기할 수 있다.
#### `boolean tryLock()`
- lock 획득을 시도하고, 즉시 성공 여부를 반환.
#### `boolean tryLock(long time, TimeUnit unit)`
- 주어진 시간 안에 lock을 획득하면 true를 반환, 주어진 시간 안에 획득하지 못하면 false 반환.
- lock 획득 대기 중 interrupt가 발생하면 lock 획득을 포기한다.
#### `void unlock()`
- lock을 획득한 스레드가 호출해야 하며, lock을 해제한다.
#### `Condition newCondition()`
- Condition 객체는 lock과 결합되어 사용되며, 스레드가 특정 조건을 기다리거나 신호를 받는다.
- Object 클래스의 wait, notify와 유사한 역할을 한다.

## synchronized vs ReentrantLock

### 1. 제공 방식
- synchronized는 Java에서 제공하는 키워드로 모든 인스턴스에는 모니터 락이 있다.
- ReentrantLock은 Java의 java.util.concurrent 라이브러리에서 제공하는 클래스이다.
### 2. 스레드 대기 상태
- synchronized를 사용할 때, 스레드가 lock을 기다리는 상태는 BLOCKED이다.
- ReentrantLock을 사용할 때, 스레드가 lock을 기다리는 상태는 WAITING 또는 TIMED_WAITING이다. interrupt를 통해 대기를 중단할 수 있다.
### 3. 생산자-소비자 문제, 한정된 버퍼 문제에서의 효율성
- synchronized를 사용할 경우, 대기 집합(wait set, `wait()`)에서 원하는 스레드만 선택적으로 깨울 수 없어서 비효율적이다.
	- 비효율적이지만, 생산자-소비자 문제와 한정된 버퍼 문제를 해결할 수는 있다.
- ReentrantLock은 Condition 객체를 활용하여 원하는 스레드만 선택적으로 깨울 수 있어 생산자-소비자 문제를 효율적으로 해결할 수 있다.