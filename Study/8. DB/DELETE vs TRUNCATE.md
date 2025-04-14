## DELETE vs TRUNCATE

### DELETE가 느린 이유
- DELETE는 삭제할 레코드를 먼저 SELECT로 찾은 후, 하나씩 삭제한다.
- Undo Log 부하: 삭제 시 원본 데이터를 Undo Log에 백업.
	- 1만 행 삭제면 1만 개의 로그 발생
### TRUNCATE가 빠른 비결
- Drop & Create: 테이블 구조를 재생성해서 로그 없이 한 번에 데이터 삭제
- Undo Log 부하: 트랜잭션 로그를 생성 x, Undo Log 부하 없음

## DELETE 주의 사항

### Gap Lock의 저주
MySQL 기본 격리 수준(REPEATABLE-READ)에서 DELETE는 인덱스 범위를 잠군다.
#### ex
id=10~20 삭제 시, 인덱스 기준으로 최소 10~21부터 많게는 5~25까지 갭 락 발생

다른 트랜잭션이 갭 락 구간에 INSERT/UPDATE 시도하면  Lock Wait 하게 되면서 DML이 멈추게 되어 서비스 장애까지 이어질 수 있다.

### Undo Log 폭발
- 행 단위 로깅: 10만 행 삭제 -> 10만 개의 Undo Log 생성 -> 트랜잭션 로그 용량 급증
- 장기간 실행 시 디스크 I/O 과부하로 서버 성능 저하

### 인덱스 의존성
WHERE 절에 인덱스 없으면 SELECT 시 Full 스캔 발생으로 속도 저하

## 성능 최적화 팁

### 대량 삭제 시
DELETE 대신 테이블 복제 -> TRUNCATE -> 데이터 재삽입으로 우회

### 인덱스 활용
WHERE 조건에 인덱스가 없으면 풀 스캔 발생. 반드시 인덱스 추가