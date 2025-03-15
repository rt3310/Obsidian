
## Prefix Index 이해

Prefix Index는 칼럼의 전체 값 대신 일부 접두사만을 이용하여 인덱스를 구축하는 방법

### 사용 이유
BLOB, TEXT, 긴 VARCHAR와 같이 전체 인덱싱이 불가능한 경우에 사용
- `REDUNDANT` or `COMPACT` row format 인 경우 최대 767 바이트까지 사용 가능
- `DYNAMIC` or `COMPRESSED` row format 인 경우 최대 3072 바이트까지 사용 가능

## Prefix Index 최적화

### 적절한 접두사 길이 선택
- 카디널리티가 충분히 높아지는 접두사 길이를 선택