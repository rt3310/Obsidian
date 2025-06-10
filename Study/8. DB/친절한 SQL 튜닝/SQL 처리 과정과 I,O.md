## 소프트 파싱 vs 하드 파싱

> [!note] 라이브러리 캐시
> 내부 프로시저를 반복 재사용할 수 있도록 캐싱해 두는 메모리 공간

> [!note] SGA(System Global Area)
> 서버 프로세스와 백그라운드 프로세스가 공통으로 엑세스하는 데이터와 제어 구조를 캐싱하는 메모리 공간

### 소프트 파싱
SQL을 캐시에서 찾아 곧바로 실행단계로 넘어가는 것
### 하드 파싱
찾는 데 실패해 최적화 및 로우 소스 생성 단계까지 모두 거치는 것

> [!note]
> 데이터베이스에서 이루어지는 처리 과정은 대부분 I/O 작업에 집중되는 반면, 하드 파싱은 CPU를 많이 소비하는 몇 안 되는 작업 중 하나다.

## 바인드 변수의 중요성

사용자 정의 함수/프로시저, 트리거, 패키지 등은 생성할 때부터 이름을 갖는다. 컴파일한 상태로 딕셔너리에 저장되며, 사용자가 삭제하지 않는 한 영구적으로 보관된다.
실행할 때 라이브러리 캐시에 적재함으로써 여러 사용자가 공유하면서 재사용한다.

반면, SQL은 이름이 따로 없다. 전체 SQL 텍스트가 이름 역할을 한다.

DBMS에서 수행되는 SQL이 모두 완성된 SQL은 아니며, 특히 개발 과정에는 수시로 변경이 일어난다. 일회성(ad hoc) SQL도 많다. 일회성 또는 무효화된 SQL까지 모두 저장하려면 많은 공간이 필요하고, 그만큼 SQL을 찾는 속도도 느려진다. 오라클, SQL Server 같은 DBMS가 SQL을 영구저장하지 않는 쪽을 선택한 이유다.

500만 고객을 보유한 어떤 쇼핑몰에서 다음과 같이 작성했다고 하자.
```java
public void login(String login_id) throws Exception {
	String SQLStmt = "SELECT * FROM CUSTOMER WHERE LOGIN_ID = '" + login_id + "'";
	Statement st = con.createStatement();
	ResultSet rs = st.executeQuery(SQLStmt);
	if (rs.next()) {
		// do anything
	}
	rs.close();
	st.close();
}
```
이 쇼핑몰에서 어느 날 12시 정각부터 딱 30분간 대대적인 할인 이벤트를 하기로 했다. 500만 명 중 20%에 해당하는 100만 고객이 이벤트 당일 12시를 전후해 동시에 시스템 접속을 시도할 경우 어떤 일이 발생할까?

DBMS에 발생하는 부하는 대개 과도한 I/O가 원인인데, 이날은 I/O가 거의 발생하지 않음에도 불구하고 CPU 사용률은 급격히 올라가고, 라이브러리 캐시에 발생하는 여러  종류의 경합 때문에 로그인이 제대로 처리되지 않을 것이다.
이는 각 고객에 대해 동시다발적으로 발생하는 SQL 하드파싱 때문이다.

그 순간 라이브러리 캐시(V$SQL)를 조회해 보면, 아래와 같은 SQL로 가득 차 있다.
```sql
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'oraking'
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'javaking'
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'tommy'
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'karajan'
...
```

로그인 프로그램을 이렇게 작성하면, 고객이 로그인 할 때마다 아래와 같이 DBMS 내부 프로시저를 하나씩 만들어서 라이브러리 캐시에 적재하는 셈이다.
```sql
create procedure LOGIN_ORAKING( ) { ... }
create procedure LOGIN_JAVAING( ) { ... }
create procedure LOGIN_TOMMY( ) { ... }
create procedure LOGIN_KARAJAN( ) { ... }
...
```
위 프로시저의 내부 처리 루틴은 모두 같다. 그렇다면 프로시저를 여러 개 생성할 것이 아니라 아래처럼 로그인 ID를 파라미터로 받는 프로시저 하나를 공유하면서 재사용하는 것이 마땅하다.
```sql
create procedure LOGIN (login_id in varchar2) { ... }
```
이처럼 파라미터 Driven 방식으로 SQL을 작성하는 방법이 저공되는데, 바인드 변수가 바로 그것이다.

```java
public void login(String login_id) throws Exception {
	String SQLStmt = "SELECT * FROM CUSTOMER WHERE LOGIN_ID = ?";
	PreparedStatement st = con.prepareStatement(SQLStmt);
	st.setString(1, login_id);
	ResultSet rs = st.executeQuery();
	if (rs.next()) {
		// do anything
	}
	rs.close();
	st.close();
}
```
이 순간 라이브러리 캐시를 조회해 보면, 로그인과 관련해서 아래 SQL 하나만 발견된다.
```sql
SELECT * FROM CUSTOMER WHERE LOGIN_ID = :1
```

## SQL이 느린 이유

SQL이 느린 이유는 십중팔구 I/O 때문이다. 구체적으로 말해, 디스크 I/O 때문이다.

## 데이터베이스 저장 구조

- 블록: 데이터를 읽고 쓰는 단위
- 익스텐트: 공간을 확정하는 단위. 연속된 블록 집합
- 세그먼트: 데이터 저장공간이 필요한 오브젝트(테이블, 인덱스, 파티션, LOB 등)
- 테이블스페이스: 세그먼트를 담는 콘테이너
- 디스크 상의 물리적인 OS 파일

