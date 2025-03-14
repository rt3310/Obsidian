## INSERT 실행 비용

- **연결 비용(Connecting)**: MySQL 서버와의 연결 비용
- **쿼리 전송(Sending Query to Server)**: 서버로 쿼리를 보내는 비용
- **쿼리 파싱(Parsing Query)**: 서버에서 쿼리를 분석하는 비용
- **행 삽입(Inserting Row)**: 데이터 행을 삽입하는 비용 (1 x size of row)
- **인덱스 삽입(Inserting Indexes)**: 각 인덱스에 대한 삽입 비용 (1 x number of indexes)
- **연결 종료(Closing)**: 연결을 종료하는 비용

## INSERT 문 최적화

- **대량 삽입(Bulk Insert)**: 여러 번의 네트워크 통신을 줄이고 한 번에 많은 데이터를 전송
	- **INSERT 문 개선**: 단일 INSERT 문에 여러 값(MULTIPLE VALUES) 삽입으로 처리 속도 개선
	- **LOAD DATA 활용**: 파일에서 직접 데이터를 불러오는 기능으로 처리 속도가 훨씬 빠름

- 일관성 검사 지연(Consistency check delay): 일관성 검사를 미루어 처리 속도 향상

- 기본 값을 넣을 땐 INSERT 시 생략하기: INSERT 문 파싱 속도를 더 빠르게 하기 위한 방법

## LOAD DATA 사용 시 고려 사항

### `LOAD DATA` Statement 란?
- Storage Engine에서 지원해주는 Bulk Insert 기능.
- INSERT 문보다 처리속도가 20배 더 빠르다고 함
- INSERT 문과는 달리 데이터가 들어있는 File을 줘야한다.

### `LOAD DATA` 문을 사용할 때 주의사항

**auto commit 하지 않도록 변경**
- 이를 하지 않으면 INSERT 할 때마다 Redo Log 플러쉬가 될 것

**병렬 처리 고려**
- LOAD DATA 문은 단일 스레드 + 단일 트랜잭션으로 처리한다.
- 트랜잭션이 처리되는 동안에는 Undo Log를 지울 수 없는 문제가 발생한다.

데이터 파일을 전송하는 경우 보안 설정 필요
- LOAD DATA 문은 MySQL 서버가 파일로 가지고 있어야 실행된다.
- 클라이언트 쪽에서 파일을 가지고 있는 경우에는 LOAD DATA LOCAL을 실행해야 한다.
- 파일을 전송하기 위해서는 서버/클라이언트 모두 `local_infile=ON` 설정을 활성화 해야한다.