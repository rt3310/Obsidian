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
