
## 데드락 감지 이해

### 데드락 감지 동작
`SELECT ... FOR UPDATE`와 같은 Lock을 요구하는 연산은 데드락 감지 스레드를 통해서 데드락을 유발하는지 검사를 주기적으로 받는다.

### 데드락 감지가 성능에 미치는 영향
많은 락을 요구하는 서비스 스레드 처리가 있다면 데드락 감지 스레드 때문에 지연이 발생할 수 있다.

### 데드락 감지 스레드의 동작 방식
데드락 감지 스레드가 검사하는 동안에는 잠금을 가진 서비스들은 잠금을 반납하지 못한다.

## 최적화 전략

### **`innodb_deadlock_detect`** 을 비활성화 시키기
락을 요구하는 작업이 많다면 이를 비활성화 시키는 것
- **비활성화하면 데드락이 발생할 수 있으므로 `innodb_lock_wait_timeout`** 을 적절한 값으로 설정해야 한다. (기본 값: 50)
- 환경에 따라서 Lock 을 기다리는 타임아웃은 조절하는 것이 좋음: (OLTP 환경 vs OLAP 환경)

## 락을 기다리는 트랜잭션의 유무를 확인하는 방법

### **`Performance Schema`**
- **`Wait Event`** 테이블
	- 잠금이나 I/O 작업을 통해서 대기하고 있는 스레드에 대한 정보를 확인할 수 있다
- **`Lock`** 테이블
	- 잠금이 점유되었거나, 잠금을 기다리고 있는 정보를 확인할 수 있다.

### **`InnoDB status`**
InnoDB 내부 상태를 통해서 잠금을 기다리는 트랜잭션을 볼 수 있다. 