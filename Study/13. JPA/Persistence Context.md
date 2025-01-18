## 알아두어야 하는 상식

- DB와 영속성 컨텍스트의 동기화 시점
	- commit
	- flush
	- JPQL
	    - repository.findAll() 또한 JPQL의 호출이다.
	    - repository.findById()는 영속성 캐시에 해당 값이 있을 경우에만 JPQL 호출이 아니다.
	    - JPQL이 호출되면 먼저 flush를 호출하고 쿼리문을 실행한다.

- **QueryDSL의 update** 구문은 영속성 컨텍스트에 포함되지 않는다.
	- 하지만 select 구문은 영속성 컨텍스트에 포함시킬 수 있다.
	- QueryDSL로 만든 구문은 JPQL 로 변환된다.
	- 영속성 컨텍스트는 setter와 같이 객체를 직접 변경시켜야 변경사항이 인지된다.

- **flush() 시점**에 EntityManager는 항상 **컨텍스트의 스냅샷과 DB의 스냅샷을 비교**한다.

- **@Transactional(readOnly=true)** 는 강제로 flush()하지 않는 이상 flush가 발생하지 않는다.

- **읽기 전용 쿼리 힌트**를 사용하면 스냅샷을 보관하지 않는다.
	- 대용량 데이터를 처리해야 할 때, 메모리 상의 이점을 얻을 수 있다.

- JPA는 데이터베이스 **SQL 힌트** 기능을 제공하지 않는다. **하이버네이트**가 제공한다.

- commit() 전까지는 쓰기지연의 효과를 가진다. (DB에 락이 걸리지 않는다)

- 트랜잭션 격리수준에 대한 이해

- afterCommit()에서는 아직 영속성이 정리되지 않아 lazy loading이 가능하다.

- commit시, flush가 자동 호출된다.

- 서브쿼리는 쿼리에서의 안티패턴이다. (by 향로)

- JPA에서 **IDENTITY** 생성전략을 사용하면 **Batch Insert가 비활성화** 된다.
	- 새로 할당할 PK 값을 미리 알 수 없기 때문에
	- JdbcTemplate의 batchUpdate 사용 (Type-safe 포기)
	- Exposed 라이브러리 사용

- repository.save()의 구현을 보면 if문으로 persist()와 merge()가 분기한다.
	- 즉, **flush()의 호출은 없다**.

- QueryDSL의 동적 쿼리를 위한 연속적인 if문과 영속성 컨텍스트의 스냅샷 비교 중 뭐가 효율이 좋을까?
	- QueryDSL 승 → 단위별로 쪼개면서 하지 않았을 때, 1만건 기준 2000배의 차이를 보인다고 한다.
	- 대용량 데이터를 update할 때, 특정 단위(ex. 100개)별로 영속성 컨텍스트를 flush하고 clear한다면?
### em.flush()
추가로 사용할 트랜잭션 내에서 엔티티를 데이터베이스에 즉시 저장하고 롤백할 수 있다.
### em.getTransaction().commit
트랜잭션의 종료를 표시하고 트랜잭션 내의 모든 변경 내용을 데이터베이스에 저장하며 롤백할 수 있다.

### [JPA 사실과 오해](https://developer-ping9.tistory.com/255)