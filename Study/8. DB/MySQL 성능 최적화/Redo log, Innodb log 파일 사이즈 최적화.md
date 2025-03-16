## Redo Log 동작 이해

### Redo Log 란?
- 데이터 변경 내역을 로그에 저장해서 Durability를 올리기 위해 사용된다.
- 즉, 커밋했지만 아직 디스크에 반영되지 않는 데이터가 있는 상황에서 MySQL이 죽은 경우에 복구할 때 사용한다.
- **`innodb_flush_log_at_trx_commit`** 값에 따라서 플러쉬 지점이 달라진다.

### 트랜잭션 처리와 Redo Log 기록 과정
1. 트랜잭션이 데이터를 변경하면, 해당 변경 내용은 먼저 Log Buffer에 기록된다.
2. 트랜잭션이 커밋을 요청하면, MySQL은 먼저 Log Buffer에 있는 모든 관련 로그 기록을 Redo Log 파일로 플러시 한다.
3. Log Buffer의 내용이 Redo Log 파일로 플러시 된 후, 트랜잭션이 성공적으로 커밋된다. 이로써 데이터 변경 사항이 안정적으로 저장된 것으로 간주된다.

### **`innodb_flush_log_at_trx_commit`** 값에 따른 동작 과정
- 값이 0이면 1초에 한 번씩 플러쉬 된다. 그래서 Durability를 엄격하게 보장하지 않는다.
- 값이 1이면 매 트랜잭션마다 디스크에 플러쉬 된다. 엄격한 Durability를 보장한다.
- 값이 2이면 매 트랜잭션마다 커밋되면 운영체제의 메모리 버퍼에 기록되고 동기화는 1초마다 유지된다.
	- MySQL이 종료되더라도 운영체제가 정상적으로 작동한다면 Durability를 보장, 그렇지 않다면 1초 간의 유실이 생길 수 있다.

> [!note] 체크포인트란?
> 체크포인트는 데이터베이스가 Redo Log의 내용을 정기적으로 디스크의 실제 데이터 파일로 플러시하는 과정

## 최적화 방법

### Redo Log 파일 사이즈 증가
- Redo Log 파일 사이즈만큼 가득 차게되면 Redo Log의 변경된 데이터 내역들이 디스크 데이터에 반영이 되는 체크포인트 과정이 발생한다.
- 만약 Redo Log의 사이즈가 작다면 이런 체크포인트 과정이 자주 발생해서 Disk Write 작업이 많이 발생할 수 있다.
- Write Intensive 한 어플리케이션이라면 Redo Log File 사이즈를 증가시키는 것이 좋다.
- MySQL 8.0.30 부터는 **`innodb_redo_log_capacity`** 로 설정할 수 있고, 이전 버전에서는 **`innodb_log_file_size`** 와 **`innodb_log_files_group`** 을 사용해야 한다.
- 적절한 Redo Log 파일 사이즈를 측정하기 위해서는 Peek Traffic 동안 Redo Log 파일이 쌓이는 사이즈를 측정해서 이를 바탕으로 설정하는 것을 권장한다.

#### Redo Log가 특정 시간 동안 얼마나 쌓였는지 아는 방법