
## Offset 사용 시 문제점

`Offset` 을 사용하면 지정된 수만큼의 데이터를 읽고 버려야 하므로 성능이 저하된다.
```sql
SELECT * FROM blog_posts ORDER BY post_id LIMIT 100000, 10
```

## NoOffset 최적화 전략

### Offset 없이 조회하는 법
마지막으로 처리된 레코드의 키를 기준으로 다음 데이터를 조회

### 배치 시스템을 예시
현재 Chunk를 처리하고 다음 번 Chunk를 읽어오는 상황
- LIMIT 절에 Offset을 사용하면 간단히 이를 구현할 수 있다.
- 그러나 Offset을 사용하지 않고도 가능하다. 마지막으로 처리된 레코드의 키를 사용해서 다음 데이터를 읽으면 된다.
```sql
# 첫 번째 Chunk 읽기 작업: 마지막 Id 값은 15800 이라고 가정
SELECT * data FROM my_table
WHERE status = 'pending'
ORDER BY id
LIMIT 10000
```

```sql
# 다음 번 Chunk 읽기 작업
SELECT * data FROM my_table
WHERE id > 15800
ORDER BY id
LIMIT 10000;
```