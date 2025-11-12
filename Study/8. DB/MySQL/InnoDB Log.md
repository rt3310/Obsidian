## Redo Log

MySQL의 기본 스토리지 엔진인 InnoDB는 트랜잭션의 무결성과 영속성을 보장하기 위해 Redo Log를 생성하여 관리한다.
Redo Log는 WAL(Write-Ahead Log)의 일종으로, [[WAL]]의 핵심 기능을 제공한다.

다음과 같은 기능을 가지고 있어야 WAL이라고 할 수 있다.
- 데이터 변경 작업이 발생할 때 먼저 WAL에 작성하고 그 다음에 디스크에 순차적으로 기록되게 구현한다.
- 트랜잭션이 성공적으로 Commit 되었다고 판단하는 시점은 **WAL에 기록되고 디스크에 동기화 된 시점**을 의미한다.
- WAL에 기록된 변경 사항은 이후에 백그라운드 프로세스에 의해 실제 데이터 파일에 기록되게 동작한다.
- Instance Crash가 발생했을 때 WAL을 사용하여 Crash Recovery가 진행되어 실제 데이터 파일에 작성되지 않은 변경 내역이 다시 적용되게 동작한다.

WAL로서 Redo Log는 InnoDB 스토리지 엔진의 트랜잭션의 영속성을 보장하기 위한 핵심적인 메커니즘이기 때문에 Redo Log 관리 방식을 이해하고 있는 것은 중요하다.

Redo Log는 앞서 말한 것처럼 **데이터의 영속성을 보장하기 위한 메커니즘**으로 InnoDB 스토리지 엔진이 사용하는 WAL을 말한다. 즉, **데이터 변경 사항을 디스크로 안전하게 기록하는 과정에서 시스템 장애 시 유실 방지를 하기 위한 로그**이다.

## Redo Log 기본 요소

Redo Log는 기본적으로 다음과 같이 Log Buffer와 Log File을 가지고 동작한다.
### Redo Log Buffer
- Redo Log Buffer는 Redo Log 작성 전에 변경 정보가 먼저 저장되는 메모리 영역이다.
- 기본 크기는 16MB이지만, `innodb_log_buffer_size` 시스템 변수를 사용하여 크기를 변경할 수 있다.
- Log Buffer의 내용은 시스템 변수의 설정값에 따라 결정된 Flush 주기에 따라 Redo Log File에 작성된다.
### Redo Log File
- Redo Log File은 데이터 변경 정보를 저장하는 디스크에 생성되는 파일을 말한다. 여기에 저장된 정보를 기반으로 InnoDB 스토리지 엔진은 **Crash Recovery**를 진행한다.
- Redo Log File은 기본적으로 **2개**가 생성되지만, `innodb_log_files_in_group` 시스템 변수를 통해 생성되는 파일 갯수를 변경할 수 있다. 또한, `innodb_log_file_size` 시스템 변수를 통해 파일의 크기 조정도 가능하다.

## Redo Log 기본 동작 방식

### 트랜잭션 수행 시 Redo Log 동작 방식
![[Pasted image 20251112163817.png]]
동작 방식은 다음과 같다.
1. DML 쿼리가 수행되어 Page 내용이 변경되면, Buffer Pool에 Page가 복사되고 그 영역에서 변경 작업이 진행된다.
2. 1번에서 진행된 작업 내용은 MTR 단위로 세션 별 메모리 영역에 기록된다.
3. MTR에 변경 내용이 모두 기록되면 MTR commit()이 수행되며 Redo Log Buffer로 이동하게 된다. 그리고, 변경된 페이지들을 Buffer Pool의 Flush List에 추가한다.
4. Flush 이벤트 발생 시 또는 Buffer Pool에 대한 Checkpoint 발생 시 Redo Log Buffer에 있는 내용이 Redo Log File로 저장된다. (Checkpoint 발생 시 디스크에 Page 내용이 쓰여지기 전에 Redo Log Buffer의 내용이 먼저 쓰이게 된다)
5. Redo Log File 저장 후 Buffer Pool에 있는 Flush List에 있는 Page 정보가 Disk의 데이터 파일로 내려써지게 된다.

위 동작방식은 모든 InnoDB 버전에서 동일하게 진행된다.
### Redo Log Buffer Flush 주기 설정
앞서 말한대로 Redo Log Buffer는 어떤 기준에 다다르거나, 이벤트 발생 시 Redo Log Buffer의 변경 내용을 Flush 한다.
Flush 작업의 주기는 MySQL 성능에 영향을 주는 아주 중요한 요소이다. 모든 Flush 유발 이벤트를 다 제어할 수는 없지만, 트랜잭션 처리와 관련하여 `innodb_flush_log_at_trx_commit` 시스템 변수를 사용하면 DBE(Database Engineer)가 Flush 주기를 제어할 수 있다.

이 시스템 변수의 값은 실행되는 트랜잭션의 안정성과 DB 성능에 크게 영향을 주기 때문에 신중히 고민하여 결정해야 한다. **안정성이 중요할 경우 1로 설정**하여 **커밋 시 마다 디스크에 작업 내용이 Flush** 될 수 있게 해야 하고, **성능이 종요하다면, 0 또는 2**로 선택하여 성능을 높이도록 해야 한다.

`innodb_flush_log_at_trx_commit` 시스템 변수는 MySQL 내구성과 성능 사이의 균형을 맞추는데 중요한 변수이니 따로 상세히 공부하는 것을 추천한다.

## MTR(mini-Transaction)

MTR은 **Redo Log가 내부 작업을 할 때 사용하는 가장 작은 작업 단위를 말한다**. 즉, **단일 Page 또는 소수의 Page에 대한 변경 작업을 Redo Log에 기록하는 가장 작은 단위**이다.

### MTR 특징
MTR은 다음과 같은 특징을 가지고 있다.
- Redo Log 작업의 최소 단위이다. 즉, **실제 동작하는 I/O 작업의 최소단위**이다. 그래서, **사용자가 생성하는 하나의 트랜잭션은 여러 MTR로 분리되어 처리**된다.
- 원자성을 보장한다. 즉, MTR에 포함된 모든 변경 내용이 성공적으로 Redo Log에 기록되거나, 아니면 아무 것도 기록되지 않는다.
- WAL(Write-Ahead Log)로 동작한다. 즉, 작업 내용은 먼저 Redo Log Buffer에 기록하고 그 다음에 스케줄에 따라 디스크의 Redo Log File에 Flush 된다.
  ![[Pasted image 20251112174759.png]]

Redo Log 작성 시 MTR 구조를 사용함으로써 다음과 같은 효과를 얻을 수 있다.
- MTR 단위로 하나의 트랜잭션 작업을 나누면 **여러 내부 작업을 동시에 진행**할 수 있다. 이를 통해 성능을 향상한다.
- **MTR은 물리적인 변경 단위를 기본으로 기록**하기 때문에, 논리적인 SQL보다 훨씬 더 간결하고 효율적인 Redo Log Record를 생성할 수 있다. 이를 통해 Redo Log의 크기를 줄이고, Disk I/O를 최소화하여 성능을 최적화 할 수 있다.

### MTR 구조체
MySQL 소스 안에서 MTR은 `mtr_t` 구조체를 통해 구현된다. `mtr_t` 구조체 구성 요소 중 간단히 정리하면 다음과 같다.
```c
struct mtr_t {
struct Impl { 
/** mtr이 잠금을 사용하기 위한 용도  **/
mtr_buf_t m_memo;

	/** mtr에서 사용하는 mini Transaction 로그 **/
mtr_buf_t m_log;

/** 현재 mtr 로그 모드 **/
mtr_log_t m_log_mode; 

	/** MTR 생명주기 정보 **/
mtr_stat_t m_state;


/** mtr 로그에 기록된 페이지 초기 로그 레코드 수 **/
ib_uint32_t	m_n_log_recs;

	/** 더티 페이지가 생성되고 디스크를 플러시가 필요한 경우 true **/
bool m_made_dirty; 

	/** 버퍼 페이지를 변경한 경우 true **/
bool m_modifications; 

	/** ibuf의 데이터가 변경된 경우 true **/
bool m_inside_ibuf;

 	/** 현재 mtr에 의해 수정된 테이블스페이스 **/
fil_space_t*  m_user_space;
/** 현재 mtr에 의해 수정된 Undo 테이블스페이스 **/
fil_space_t*  m_undo_space;  
/** 현재 mtr에 의해 수정된 System 테이블스페이스 **/
fil_space_t*  m_sys_space;

	/** Flush Observer **/
	/** m_log_moderedo를 쓰지 않겠다는 뜻일 때 더티 페이지의 플러시 여부는 이 파라미터
(인덱스 생성) 로 판단한다. **/
FlushObserver* m_flush_observer; 

	/** 현재 자신의 mini-Transaction **/
mtr_t*  m_mtr;
}
```
#### MTR 구조체 추가 설명
MTR 구조체 내부의 여러 변수 중 몇 개만 좀 더 알아보자.

- `m_memo`: MTR 내부에서 잠금 이용할 때 사용되는 변수이다.
- `m_log`: MTR 로그를 저장하여 관리하는 변수이다.
- `m_log_mode`: 현재 관리하는 MTR Log의 모드를 보여주는 변수로 다음과 같은 값을 가질 수 있다.
	- `MTR_LOG_ALL`(default): 