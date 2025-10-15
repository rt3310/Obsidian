## ICP(Index Condition Pushdown)

ICP(Index Condition Pushdown)은 MySQL 5.6부터 도입된 인덱스 최적화 기법으로, 인덱스를 사용할 때 데이터를 더 적게 읽기 위해 WHERE 조건의 일부를 인덱스 레벨에서 미리 평가하는 기능이다.

즉, 스토리지 엔진(ex, InnoDB)이 단순히 인덱스 키만 읽는 게 아니라, 가능한 한 `WHERE` 조건을 인덱스 스캔 도중에 평가해 불필요한 row 접근을 줄이는 방식이다.

## 기본 개념

보통 인덱스를 사용할 때 MySQL의 내부 동작은 다음과 같다.
1. 스토리지 엔진이 인덱스 스캔을 통해 후보 레코드들의 row id(or primary key)를 가져온다.
2. MySQL 서버 레이어가 그 row id를 사용해 실제 테이블 데이터를 읽는다.
3. WHERE 절을 평가해서 최종적으로 조건에 맞는 row만 반환한다.

즉, 모든 후보 row에 대해 실제 데이터 페이지를 읽어야 하므로 비효율적일 수 있다.

ICP는 이 중간 단계를 최적화한다.
> WHERE 조건 중 인덱스 컬럼에 대한 조건은 스토리지 엔진 단계에서 미리 평가

즉, 스토리지 엔진이 인덱스를 스캔하면서 인덱스 엔트리만 보고도 조건을 만족하지 않는 레코드를 건너뛸 수 있게 해준다.
따라서 서버 레이어로 전달되는 후보 row수가 줄어든다.


## 사용

### 사전 작업
먼저 테스트로 사용할 MySQL (or MariaDB)에서 옵티마이저 스위치 값을 조정하여 인덱스 컨디션 푸시다운 옵션을 off 하자.
```sql
set optimizer_switch = 'index_condition_pushdown=off';
```

정상적으로 되었는지 확인하자
```sql
show variables like 'optimizer_switch';
```

```
...index_condition_pushdown=off;...
```
여러 옵션이 나오는데 위와 같이 옵션이 있다면 정상적으로 off된 것이다.

그리고 테스트에 사용할 환경을 구성한다.
임의의 테스트 테이블(`temp_ad_offset`)을 만들고 대략 1300만건의 데이터를 넣어본다.
그리고 사용할 인덱스 (`customer_id, offset_type`) 역시 추가해준다.
```sql
ALTER TABLE temp_add_offset ADD INDEX idx_temp_ad_offset_2 (customer_id, offset_type);
```

### 테스트
인덱스 컨디션 푸시다운 옵션이 off된 상태(`index_condition_pushdown=off`)에서 아래 쿼리의 실행 계획(`explain`)을 확인해본다.
```sql
SELECT *
FROM temp_ad_offset
WHERE customer_id = 7 AND offset_type LIKE '%LIST';
```

실행 계획을 보면 `Using where`가 나타난 것을 알 수 있다.
![[Pasted image 20251015174127.png]]

MySQL(MariaDB)에서는 `like` 사용 시 와일드카드 (ex, `like %abc`)로 시작되는 값에 대해서는 인덱스가 적용되지 않기 때문에, `customer_id=7`은 인덱스를 통해 걸러내고, `offset_type like '%LIST'`에 대해서는 인덱스가 적용되지 않는 방식이니 걸러진 데이터를 테이블에서 하나씩 비교했기 때문이다. ([참고](https://mariadb.com/docs/server/reference/sql-functions/string-functions/like#optimizing-like))

좀 더 자세히 알아보기 위해 MySQL(MariaDB)의 쿼리 실행 구조를 확인해보자.
MySQL(MariaDB)는 내부적으로 MySQL(MariaDB)엔진과 스토리지 엔진(InnoDB/XtraDB)로 나눠져 있다.
![[Pasted image 20251015174836.png]]
스토리지 엔진이 넘겨 준 데이터(인덱스를 사용해 걸러진 데이터) 중에서 MySQL(MariaDB) 엔진이 한 번 더 걸러야되는 조건 (필터링 혹은 체크 조건)이 있다면 `Using where`이 된다.

즉, 이 쿼리에서 스토리지 엔진이 걸러 낼 수 있는 조건은 `customer_id=7`뿐이며 `offset_type like '%LIST'` 조건은 MySQL(MariaDB) 엔진에서 담당하여 `customer_id=7`인 데이터들을 테이블에서 모두 찾아 `offset_type like '%LIST'` 조건을 비교하게 된다.

그런데 여기서 한 가지 의문이 생긴다.
이미 `customer_id=7`을 통해 idx_temp_ad_offset_2 인덱스 필드 (customer_id, offset_type)를 읽은 상태인데 offset_type 비교를 테이블에서 굳이 할 필요가 있을까?

이건 이유가 있다.
MySQL 5.5 (MariaDB 5.2) 버전까지는 인덱스에 포함된 필드이지만, 인덱스 범위 조건으로 사용할 수 없는 경우엔 스토리지 엔진 조건 자체를 전달조차 못했다.
스토리지 엔진에서 해당 필드에 대한 조건은 받은게 없으니 처리할 수가 없는 것이다.

그 이후 버전 (MySQL 5.6 / MariaDB 5.3) 부터는 인덱스 범위 조건에 사용될 수 없어도, 인덱스에 포함된 필드라면 스토리지 엔진으로 전달하여 최대한 스토리지 엔진에서 걸러낸 데이터만 MySQL (MariaDB) 엔진에만 전달되도록 개선되었다.

> [!info]
> 인덱스 조건을 스토리지 엔진으로 넘겨주기 때문에 인덱스 컨디션 푸시 다운이란 이름이 되었다.

자 그럼 다시 `index_condition_pushdown` 옵션을 `on` 해보자.
```sql
set optimizer_switch = 'index_condition_pushdown=on';
```
그럼 아래와 같이 인덱스 컨디션 푸시 다운이 잘 작동되는 것을 확인할 수 있다.
![[Pasted image 20251015190757.png]]


## 예시

예를 들어, 다음과 같은 테이블이 있다고 하자
```sql
CREATE TABLE user_log(
	id BIGINT PRIMARY KEY,
	user_id INT,
	log_date DATETIME,
	action VARCHAR(50),
	INDEX idx_user_date(user_id, log_date)
);
```

그리고 다음 쿼리를 실행한다고 하자
```sql
SELECT *
FROM user_log
WHERE user_id = 10 AND log_date BETWEEN '2025-01-01' AND '2025-02-01' AND action = 'LOGIN';
```

### ICP 미사용 시
- `user_id`와 `log_date`로 구성된 인덱스를 스캔하면서 일단 후보 레코드들을 모두 가져온다.
- 각 후보마다 테이블 데이터를 읽는다.
- 마지막 `action='LOGIN'` 조건은 테이블 데이터를 읽은 후에야 평가된다.
즉, 인덱스 조건을 통과했더라도 실제로는 `action='LOGIN'`이 아닌 row가 많으면 불필요한 I/O가 발생한다.

### ICP 사용 시
- 스토리지 엔진이 인덱스 엔트리를 스캔하면서, `user_id=10 AND log_date BETWEEN ...` 조건을 만족하는 인덱스 엔트리를 찾을 때 action 조건까지 함께 인덱스 레벨에서 평가할 수 있으면 바로 필터링한다.
결국 테이블 접근 횟수가 줄어들고, 쿼리 속도가 빨라진다.

### ICP 확인
쿼리 실행계획에서 `Extra` 컬럼에 다음 문구가 나오면 ICP가 사용된 것이다.
```sql
EXPLAIN SELECT * FROM user_log
WHERE user_id = 10 AND log_date > '2025-01-01';
```

| id  | select_type | table    | type  | key           | Extra                     |
| --- | ----------- | -------- | ----- | ------------- | ------------------------- |
| 1   | SIMPLE      | user_log | range | idx_user_date | **Using index condition** |

## ICP가 가능한 조건

ICP는 다음 조건에서만 사용된다.
- 인덱스 조건이어야 한다.
	- WHERE 절이 인덱스에 포함된 컬럼에 대해서만 가능하다.
- 스토리지 엔진 지원이 필요하다.
	- InnoDB, MyISAM 등 일부 엔진만 지원한다.
- 커버링 인덱스가 아니여야 한다.
	- 커버링 인덱스일 때는 테이블 접근이 필요 없으므로 ICP가 불필요하다.
- 조건식이 단순해야 한다.
	- 인덱스 컬럼에 대한 비교/범위/LIKE(prefix) 조건 등

## 장점 및 한계

### 장점
- 불필요한 테이블 row 접근 감소 → 디스크 I/O 절감
- 쿼리 성능 향상 (특히 인덱스 범위 스캔에서 효과적)
### 한계
- 인덱스에 포함되지 않은 조건은 ICP 대상이 아니다.
- 인덱스가 너무 커서 random I/O가 많을 경우 효과가 제한적이다.
- WHERE 조건이 복잡한 경우 ICP로 pushdown되지 않을 수 있다.
