
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