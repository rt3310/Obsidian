## Executors

Executors가 생성해주는 ExecutorService는 **재사용이 가능한 ThreadPool**로 Executors 인터페이스를 확장하여 **Thread의 라이프사이클을 제어**하며 발생할 수 있는 고려사항들을 개발자가 신경쓰지 않도록 편리하게 추상화한 것이다.

## ThreadPool의 종류

### newFixedThreadPool(int, [ThreadFactory])
- 고정된 개수의 스레드를 사용하여 동작하며, 공유된 무한 큐를 사용하여 작업을 처리하는 스레드 풀을을 생성한다.
- 파라미터로 통해 생성된 스레드 개수는 항상 활성화되어 작업을 처리한다.
- 모든 스레드가 활성화 된 상태에서 추가 작업이 있으면 큐에 대기한다.
- 실행 중에 오류로 스레드가 종료되면 새로운 스레드가 생성된다.
### newCachedThreadPool([ThreadFactory])
- 이전에 생성된 스레드를 재사용하는 스레드 풀을 생성한다.
- 사용 가능한 이전 스레드가 없으면 새 스레드를 생성하고 풀에 추가한다.
- 최대 개수는 Integer.MAX_VALUE이다.
- 60초 동안 사용되지 않은 스레드는 종료되고 캐시에서 삭제된다.
- 이미 생성된 스레드를 재사용할 수 있기 때문에 성능 상의 이점이 있을 수 있다.
### newScheduledThreadPool(int, [ThreadFactory])
- 일정 시간 뒤에 실행되는 작업이나, 주기적으로 수행되는 ThreadPool을 인자 개수만큼 생성한다.
### newWorkStealingPool(int parallelism)
- JDK 1.8부터 지원하는 방식으로 시스템에 가용 가능한 만큼 스레드를 활용하는 ExecutorService를 생성한다.
- 주어진 파라미터 수만큼 병렬성을 지원하는 데 필요한 스레드를 유지한다.
- 여러 큐를 사용하여 스레드 풀을 생성한다.
- 실제 스레드 수는 동적으로 움직인다.
### newSingleThreadExecutor([ThreadFactory])
- 단일 스레드인 풀을 생성한다.
- 단일 스레드에서 동작해야 하는 작업을 처리할 때 사용한다.
### unconfigurableExecutorService(ExecutorService)
### unconfigurableExecutorService(ScheduledExecutorService)
- SingleThread 처럼 캐스트를 사용하여 엑세스를 할 수 없다.
- 해당 스레드는 설정을 변환하지 않고 ExecutorService 메서드만 사용할 수 있다.
### defaultThreadFactory()
- 기본 스레드 팩토리를 반환한다.
- 생성되는 모든 스레드를 동일한 ThreadGroup에서 생성한다.
- 새로운 스레드는 pool-N-thread-M의 이름으로 만들어지며, 여기서 N은 이 팩토리의 연속 번호이고 M은 이 팩토리에 의해 생성된 스레드의 연속 번호이다.
- 우선순위가 작은 값으로 생성되며, 비데몬 스레드이다.
### callable
- 작업을 실행하고 결과를 반환하는 Callable 객체를 반환한다.

## 작업을 생성하는 메서드

### execute()
- 작업 처리 결과를 반환하지 않는다
- 작업 처리 도중 예외가 발생하면 스레드가 종료되고 해당 스레드는 ThreadPool에서 제거된다.
- 다른 작업을 처리하기 위해 새로운 스레드를 생성한다.
### submit()
- 작업 처리 결과(future)를 반환한다.
- 결과를 받아야 하기에 Callable을 구현한 Task를 인자로 전달한다.
- 작업 처리 도중 예외가 발생하더라도 스레드는 종료되지 않고 다음 작업을 위해 재사용한다.
- 스레드의 생성 오버헤드를 방지하기 위해서라도 submit()을 가급적 사용한다.
### invokeAny()
- 실행에 성공한 작업 중 하나의 리턴값을 반환한다.
- Task를 Collection에 넣어서 인자로 넘겨줄 수 있다.
### invokeAll()
- 모든 작업의 리턴값을 List<Future<>> 타입으로 반환한다.
- Task를 Collection에 넣어서 인자로 넘겨줄 수 있다.

## 작업을 종료하는 메서드

### shutdown()
- 작업 큐에 남은 작업을 모두 마무리하고 종료한다.
- 오버헤드를 방지하기 위해서 일반적으로 많이 사용한다.
### shutdownNow()
- 작업 큐에 상관없이 강제로 종료한다.
### awaitTermination()
- 이미 수행 중인 작업이 지정된 시간 동안 끝나기를 기다리며, 해당 시간 내에 끝나지 않으면 작업 스레드들을 interrupt() 시키고 false를 반환한다.