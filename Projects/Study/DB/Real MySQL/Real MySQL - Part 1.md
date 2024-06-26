## 모델링

- 논리 모델링 (업무 전문가)
	- 업무 분석
	- 엔터티 & 속성 & 관계 도출
	- 정규화
- 물리 모델링 (DBMS 전문가)
	- DBMS 벤더별 최적 컬럼 타입 선정
	- 접근 패턴 분석
	- 반 정규화
	- 인덱스 전략

## CHAR vs VARCHAR

#### 공통점
- 문자열 저장용 컬럼
- 최대 저장 가능 문자 길이 명시 (바이트 수 아님)

#### 차이점
- 값의 실제 크기에 관계 없이 고정된 공간 할당 여부
- 최대 저장 길이: CHAR(255) vs VARCHAR(16383) → 65535 byte
- 저장된 값의 길이 관리 여부 (VARCHAR와 가변 길이 문자셋 사용하는 CHAR는 저장된 값 길이 관리)
	- 0 ~ 255 bytes                 length-bytes: 1
	- 256 ~ 65535 bytes       length-bytes: 2

### CHAR vs VARCHAR (Latin1)
- Latin1: 고정 길이 문자셋
![[Pasted image 20240626185307.png]]
VARCHAR 타입의 길이 저장 바이트가 컬럼 바로 앞에 저장되어 있는 것 처럼 보이지만 실제로는 더 복잡하게 저장되어 있다.

### CHAR vs VARCHAR (UTF8MB4)
- UTF8MB4: 가변 길이 문자셋
- 