
## Insert 튜닝

Insert는 특별한 튜닝 방법이 없다.
대부분의 DB 성능 튜닝은 SELECT에 집중되어 있다.

INSERT의 성능을 향상시키는 가장 효과적인 방법은 단순하다. 바로 **단건 INSERT를 Bulk INSERT로 바꾸는 것**이다.

## 코드가 단건 INSERT인지 확인

### 1. 반복문 안에서 INSERT를 실행하는지 확인
```java
for (Item item : items) {
	jdbcTemplate.update("INSERT INTO table1 (col1, col2) VALUES (?, ?)", item.getCol1(), item.getCol2());
}
```

### 2. JPA에서 `save()` 메서드를 반복 호출하는지 확인
```java
for (Item item : items) {
	itemRepository.save(item);
}
```

### 3. 로그에서 같은 INSERT 구문이 반복되는지 확인
```
Hibernate: insert into table1 (col1, col2) values (?, ?)
Hibernate: insert into table1 (col1, col2) values (?, ?)
Hibernate: insert into table1 (col1, col2) values (?, ?)
```

## Bulk Insert 사용하기

단건 INSERT 대신 Bulk INSERT를 사용하면 DB 부하를 크게 줄일 수 있다.
Bulk INSERT는 여러 개의 데이터를 한 번에 삽입하는 기법이다.

### 단건 INSERT vs Bulk INSERT
#### 단건 INSERT
```sql
INSERT INTO table1(col1, col2) VALUES (val11, val12);
INSERT INTO table1(col1, col2) VALUES (val21, val22);
INSERT INTO table1(col1, col2) VALUES (val31, val32);
```

#### Bulk INSERT
```sql
INSERT INTO table1 (col1, col2) VALUES (val11, val12), (val21, val22), (val31, val32);
```

## Bulk INSERT가 효율적인 이유

### DB 연결 횟수 감소
- 단건 INSERT는 매번 DB에 연결하여 쿼리를 실행해야 하지만, Bulk INSERT는 한 번의 연결로 여러 데이터를 삽입할 수 있다.
### 트랜잭션 오버헤드 감소
- Auto commit 모드에서 단건 INSERT를 천 건 실행하면 천 번의 commit이 발생하지만, Bulk INSERT는 단 한 번의 commit 만 발생한다.
### 쿼리 파싱 및 최적화 비용 절감
- 데이터베이스는 각 쿼리를 파싱하고 최적화 한 후 실행한다.
- 단건 INSERT를 1000번 실행하면 이 과정을 1000번 반복해야 하지만, Bulk INSERT는 한 번만 수행하면 된다.
### 로깅 및 인덱싱 효율성 향상
- 데이터베이스는 데이터 삽입 시 로그를 기록하고 인덱스를 업데이트 한다.
- 단건 INSERT는 매번 이 작업을 수행해야 하지만, Bulk INSERT는 한 번에 처리할 수 있어서 더 효율적이다.

## 구현 방법

### JDBC를 사용하는 경우
```java
jdbcTemplate.batchUpdate(sql, new BatchPerparedStatementStter() {
	@Override
	public void setValues(PreparedStatement ps, int i) throws SQLException {
		Item item = items.get(i);
		ps.setString(1, item.getCol1()); // 첫 번째 자리표시자에 값 설정
		ps.setString(2, item.getCol2()); // 두 번째 자리표시자에 값 설정
	}

	@Override
	public int getBatchSize() {
		return items.size(); // Batch 크기 반환
	}
});
```

## Bulk INSERT 사용 시 주의사항

### MySQL 설정: rewriteBatchedStatements=true
MySQL에서는 Batch INSERT를 최적화하려면 JDBC URL에 다음 옵션을 추가해야 한다.
```
jdbc:mysql://localhost:3306/db_name?rewriteBatchedStatements=true
```
이 옵션이 없으면 MySQL이 Batch INSERT를 단건 INSERT로 처리할 수 있다.

### Batch 크기 조정
대량 데이터를 처리할 때 한 번에 너무 많은 데이터를 삽입하면 메모리 문제가 발생할 수 있기 때문에, 적절한 Batch 크기를 설정해야 한다.
```java
int batchSize = 1000; // 한 번에 처리할 데이터 크기

for (int i = 0; i < items.size(); i += batchSize) {
	List<Item> batch = items.subList(i, Math.min(i + batchSize, items.size()));
	jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
		@Override
		public void setValues(PreparedStatement ps, int j) throws SQLException {
			Item item = batch.get(j);
			ps.setString(1, item.getCol1());
			ps.setString(2, item.getCol2());
		}

		@Override
		public int getBatchSize() {
			return batch.size();
		}
	});
}
```

### 3. 트랜잭션 관리
Batch 작업은 트랜잭션으로 묶어 처리하는 것이 좋다.

## 결론

INSERT는 특별한 튜닝 방법이 없지만, 단건 INSERT를 Bulk INSERT로 변경하는 것만으로도 엄청난 성능 향상을 얻을 수 있다.

트랜잭션 관리와 락 관련 문제에 주의하면서 적절한 배치 크기로 구현하면, 데이터베이스 성능을 크게 향상시킬 수 있다.

서버 개발자는 항상 DB와 서버 부하에 대해 고려해야 하고, 가능한 DB에 부하를 주지 않도록 최적화하는 것이 중요하다.