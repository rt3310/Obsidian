 > [!tip] point
 > innodb_autoinc_lock_mode=2 설정과 이를 적용할 때 고려해야 하는 상황에 대해서 이해하는 것이 중요하다.
 
## `innodb_autoinc_lock_mode` 설정에 따른 최적화

- **Auto Increment Lock Mode**: Auto Increment 컬럼을 가진 테이블에 `INSERT` 작업 시 동시성 수준 설정

### INSERT 유형
- **Simple Inserts**(예측 가능한 행 수): 일반 INSERT 문을 말함
- **Bulk Inserts**(예측 불가능한 행 수): e.g INSERT ... SELECT, LOAD DATA, REPLACE ... SELECT

### Auto Increment Lock Mode 종류
- `innodb_autoinc_lock_mode = 0`: 모든 `INSERT` 에 Table Level AUTO-INC Lock 사용
	- 예전 MySQL의 호환성 때문에 존재
- `innodb_autoinc_lock_mode = 1`: Bulk Insert에서만 Table Level AUTO-INC Lock 사용
- `innodb_autoinc_lock_mode = 2`: Table Level의 AUTO_INC Lock 미사용, 가벼운 Mutex 사용
	- MySQL 8.0에서 기본값
	- Auto Increment로 증가하는 값이 연속적으로 증가함을 보장하지 않는다.
		- 때문에 복제 서버와 마스터 서버의 Auto Increment 컬럼 값이 일치하지 않는 경우가 발생할 수 있다.

### **`innodb_autoinc_lock_mode = 2`** 를 통한 최적화
- 주의할 것: statement 기반의 복제가 아닌 Row 기반의 복제를 이용해야 한다.
	- statement 기반의 복제는 마스터에 실행된 sql문을 그대로 복제 서버로 가지고 와서 복제를 하다보니 결과가 달라질 수 있는 것이다.
	- 로우 기반의 복제를 사용한다면 마스터 서버와 복제 서버의 행은 동일하게 복제가 되기 때문에 이런 문제가 발생하지 않는다.
	- MySQL 8.0에서는 기본적으로 복제 모드가 로우 기반이기 때문에 신경쓰지 않아도 된다.