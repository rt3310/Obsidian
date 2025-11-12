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
	- `MTR_LOG_ALL`(default): 데이터 변경 내용을 모두 저장하고 있다는 것을 뜻하는 모드로, 여기에는 redo log 저장 뿐 아니라 Dirty Page도 모두 Flush List에 추가된 모드라는 것을 의미한다.
	- `MTR_LOG_NONE`: Redo Log 저장도 하지 않았고, Dirty Page도 Flush List에 추가하지 않은 모드를 뜻한다.
	- `MTR_LOG_NO_REDO`: Redo Log는 저장하지 않지만, Dirty Page를 Flush List에 추가한 모드라는 것을 뜻한다.
	- `MTR_LOG_SHORT_INSERTS`: 로그 사이즈를 줄여서 사용하는 모드라는 것을 뜻한다.
- `m_state`: MTR의 생명주기 상태 정보를 가진 변수이다. 다음과 같은 4가지 상태로 표현된다.
	- `MTR_STATE_INIT`: 초기 상태를 뜻한다.
	- `MTR_STATE_ACTIVE`: MTR 데이터를 작성하고 있는 상태를 뜻한다.
	- `MTR_STATE_COMMITTING`: 내부 작업 중 `commit()` 함수 내부 수행 중 임을 뜻하는 상태 변수이다.
	- `MTR_STATE_COMMITTED`: `commit()` 함수 작업이 완료된 상태를 뜻한다.
#### MTR 구조체 할당 방식
MTR 구조체는 각 세션에서 트랜잭션을 수행할 때, 각 세션이 할당받아 사용하는 세션 메모리 영역의 Heap 영역에 할당된다.
MTR이 필요한 상황에서 구조체를 호출하는 방식으로 생성되는데 **초기에는 64byte로 받아서 생성**된다. 그리고, **필요한 만큼 Linked List로 연결하여 사용**한다.

`trx_undo_assign_undo` 함수 소스 상의 예제를 보면 다음과 같이 사용하는 것을 볼 수 있다.
```c
trx_undo_assign_undo(
/*=================*/
  trx_t*    trx,    /*!< in: transaction */
  trx_undo_ptr_t* undo_ptr, /*!< in: assign undo log from
          referred rollback segment. */
  ulint   type)   /*!< in: TRX_UNDO_INSERT or
          TRX_UNDO_UPDATE */
{
  trx_rseg_t* rseg;
  trx_undo_t* undo;
  mtr_t   mtr;     /** <------ MTR 선언 부분   **/
  dberr_t   err = DB_SUCCESS;

  ut_ad(trx);

  mtr_start(&mtr);  /** <------ MTR 초기화    **/
  if (&trx->rsegs.m_noredo == undo_ptr) {
    mtr.set_log_mode(MTR_LOG_NO_REDO);;
  } else {
    ut_ad(&trx->rsegs.m_redo == undo_ptr);
  }

  if (trx_sys_is_noredo_rseg_slot(rseg->id)) {
    mtr.set_log_mode(MTR_LOG_NO_REDO);;
    ut_ad(undo_ptr == &trx->rsegs.m_noredo);
  } else {
    ut_ad(undo_ptr == &trx->rsegs.m_redo);
  }

  mutex_enter(&rseg->mutex);

  DBUG_EXECUTE_IF(
    "ib_create_table_fail_too_many_trx",
    err = DB_TOO_MANY_CONCURRENT_TRXS;
    goto func_exit;
  );

  undo = trx_undo_reuse_cached(trx, rseg, type, trx->id, trx->xid,&mtr); /**  <------ MTR 하위 함수로 전달하여 사용 **/
  if (undo == NULL) {
    err = trx_undo_create(trx, rseg, type, trx->id, trx->xid, &undo, &mtr);                                                    /** <------ MTR 하위 함수로 전달하여 사용 **/
    if (err != DB_SUCCESS) {

      goto func_exit;
    }
  }

  if (type == TRX_UNDO_INSERT) {
    UT_LIST_ADD_FIRST(rseg->insert_undo_list, undo);
    ut_ad(undo_ptr->insert_undo == NULL);
    undo_ptr->insert_undo = undo;
  } else {
    UT_LIST_ADD_FIRST(rseg->update_undo_list, undo);
    ut_ad(undo_ptr->update_undo == NULL);
    undo_ptr->update_undo = undo;
  }

  if (trx_get_dict_operation(trx) != TRX_DICT_OP_NONE) {
    trx_undo_mark_as_dict_operation(trx, undo, &mtr); /** <------ MTR 하위 함수로 전달하여 사용 **/
  }

func_exit:
  mutex_exit(&(rseg->mutex));
  mtr_commit(&mtr);  /** <------ MTR 저장완료 후 Commit 진행  **/
  
  return(err);
}
```
#### MTR 생애 주기
MTR은 트랜잭션이 시작하면 **트랜잭션이 수행되면서 발생하는 데이터 변경 내역을 저장하기 위해** 생성된다. 이때 저장하는 정보는 실제 **Page의 내용** 뿐 아니라 **Undo Tablespace에 생성되어 관리하는 페이지 정보**도 같이 저장된다.
MTR은 크게 다음과 같은 순서로 생애 주기가 진행된다.
![[Pasted image 20251112205834.png]]
각 MTR을 기준으로 생애 주기 함수를 기준으로 설명하면 다음과 같이 요약할 수 있다.

| 주기                 | 동작 내용                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------ |
| `start()`            | 생성된 MTR에 대한 초기화 작업이 이루어지는 단계                                                  |
| `commit()`           | MTR 객체 내용을 Redo Log Buffer로 전달할지 말지 검토한 뒤, Case에 맞게 해당 작업을 수행하는 단계 |
| `release_resource()` | 모든 리소스를 반환하고 종료하는 단계                                                                                                 |
그림으로 표현하면 다음과 같이 간단히 도식화 해볼 수 있다.
![[Pasted image 20251112210027.png]]위 그림은 각 단계 별로 MTR이 가져가는 상태값과 모드 정보를 같이 보여주고 있다.
MTR 생애 주기에 따른 모드와 상태 값을 같이 확인하면서 보면 생애 주기에 따른 동작 방식을 이해하기가 수월할 것이다.
##### MTR Commit
MTR 생애 주기 중 Commit 작업 부분을 좀 더 상세히 알아보자.
전반적인 Flow는 다음과 같이 도식화 할 수 있다.
![[Pasted image 20251112210218.png]]Commit 함수가 호출되면 먼저 해당 작업 내용을 Redo Log Buffer로 남길지 취소 시킬 지 확인하는 단계를 거치게 된다. 이 때 Commit 작업이 취소되지 않는다면 `execute()`함수를 호출하여 다음과 같은 작업을 수행한다.
- `prepare_write()` 함수를 호출하여 MTR의 변경사항을 Redo Log Buffer에 기록하기 위한 준비를 수행한다. 그리고, MTR 작업량을 계산한다.
- `finish_wrte()` 함수를 호출하여 MTR 정보를 memcpy로 복사한다. 
## 트랜잭션 기반의 MTR 동작 방식에 대한 히해

### 트랜잭션 수행 시 MTR 동작 방식
UPDATE 쿼리 하나를 수행하는 트랜잭션을 기준으로 트랜잭션 작업에 대한 작업 내용이 어떻게 저장되는지 확인해보자.
![[Pasted image 20251112224914.png]]
위 그림을 통해 확인할 수 있듯이 쿼리가 수행되면, MTR은 크게 3가지 MTR을 생성한다.
1. Buffer Pool에 저장되는 Page 변경 내용을 담는 MTR(초록색 MTR)
2. Undo TableSpace 영역에 저장되는 Page 변경 내용을 담는 MTR(주황색 MTR)
3. 기타 커밋 작업과 관련된 정보를 저장하는 MTR(노란색 MTR)

트랜잭션이 시작하고 DML 쿼리를 수행할 때, 먼저 세션의 메모리 공간에 MTR을 생성하고 `mtr.start()`를 수행하여 초기화를 진행한다.
그 뒤 실제 쿼리 수행 후 발생하는 변경 정보들을 각각의 MTR에 저장한다. 이 때 발생하는 변경 정보 중 Undo와 관련된 정보는 Undo용 MTR에 저장하고, 일반 Page용 변경 정보는 일반 데이터 변경용 MTR에 저장한다. 그리고 추가적으로 발생하는 커밋용 정보들은 Commit용 MTR에 저장한다. 이 때, 작업량이 많아서 MTR이 더 필요하게 되면 추가 생성하여 기존 MTR에 Linked List로 연결하여 사용한다.
쿼리 수행이 완료되면, `mtr.commit()`이 호출된다. `mtr.commit()`이 호출되면, mtr에 기록된 정보들을 Redo Log Buffer로 전달하게 된다. 그리고 해당 변경 작업으로 인해 flush 되어야 하는 dirty page들은 buffer pool flush list에 추가된다.
작업이 다 완료된 MTR은 `release_resource()`를 호출하여 할당받은 리소스를 모두 해제하고 종료한다.

### 트랜잭션 완료 후 Buffer Pool Flush List 처리 방식
트랜잭션 작업이 완료되고 나면, Buffer Pool Flush List에 추가된 Page들에 대한 Flush 작업이 발생한다. 여기서는 트랜잭션 작업의 마지막 단계라고 할 수 있는 Dirty Page에 대한 Flush가 어떻게 동작하는지 간단히 살펴보자.
![[Pasted image 20251112225725.png]]Flush List로 관리되고 있는 Dirty Page들은 여러 기준에 따라 Flush 작업이 진행된다. Flush 작업은 그림에서 확인할 수 있듯이 동기 방식과 비동기 방식이 모두 동작한다.
비동기 방식으로 동작하는 함수인 `buf_flush_write_block_now()`와 동기 방식으로 동작하는 `buf_dblwr_flush_buffered_writes()`함수가 호출되어 Flush가 진행되고, 마지막에 데이터 파일에 작성할 때는 `file_io()` 함수를 호출하여 진행한다.
### 트랜잭션 롤백 시 동작 방식
트랜잭션 롤백이 수행될 때 다음과 같은 방식으로 롤백이 진행된다.
![[Pasted image 20251112225939.png]]트랜잭션 수행 시 각각 쿼리마다 MTR이 생성되고 `commit()`함수가 수행되어 해당 내용이 Redo Log에 저장된다. 그래서, 트랜잭션을 롤백할 때도 해당 변경 내용이 다시 되돌아가기 위해 Redo Log로 저장되어야 한다.

## MySQL Ver. 5.7 Redo Log 동작 방식

앞에서는 Redo Log 내부 동작 방식을 이해하기 위한 기초 지식을 정리했다. 이제는 그 지식을 기반으로 5.7의 동작 방식을 알아보자.

### Mutex 기반의 동작 방식
MySQL 5.7의 Redo Log 동작 방식은 간단히 얘기하여 Mutex 기반으로 동작한다라고 설명할 수 있다. 각각의 작업 단계에서 작업의 안정성을 보장하기 위해 MySQL 5.7은 Mutex를 사용한다.
그래서, MTR 내부 동작 방식도 다음과 같이 진행된다.
![[Pasted image 20251112230201.png]]트랜잭션 수행 시 MTR이 생성되어 작성되고 나서 내부적으로 MTR `commit()`이 호출되면 먼저 로그 버퍼 쓰기를 위한 Redo Log Buffer의 Mutex를 확보한다.
Mutex를 확보하고 나면 MTR 정보를 Redo Log Buffer에 작성하고, 작성을 완료하면 확보한 Mutex를 해제하여 반환한다.
그리고, 변경된 페이지 정보를 Buffer Pool의 Flush List에 추가하고, MTR 커밋을 위한 마무리로 관련된 리소스를 해제하는 작업을 진행한다.
### 트랜잭션 기반의 Redo Log 동작 방식
간단히 함수로 확인했던 Redo Log 동작 방식을 실제 트랜잭션을 수행하는 예제를 통해 좀 더 상세히 알아보자.
![[Pasted image 20251112230637.png]]