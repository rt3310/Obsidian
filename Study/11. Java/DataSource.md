t# DataSource

Java에서 커넥션을 얻는 방법은 다양한데, JDBC DriverManager로 신규 커넥션을 생성하거나, HikariCP 등의 커넥션 풀을 사용하는 방법 등이 있다.

만약 JDBC DriverManager를 통해 커넥션을 획득하다가, 커넥션 풀을 사용하는 방법으로 변경하려면 어떻게 해야될까?
그러려면 의존관계가 DriverManager에서 커넥션 풀로 변경되었기 때문에 애플리케이션 코드도 함께 변경되어야 한다(둘의 사용법이 다르기 때문).

하지만 Java는 커넥션을 획득하는 방법을 추상화했다.
![[Pasted image 20251130022944.png]]

Java는 `javax.sql.DataSource`라는 표준 인터페이스를 제공한다.
이 DataSource는 커넥션을 획득하는 방법을 추상화하는 인터페이스다. 핵심 기능은 커넥션 조회 뿐이다.
```java
public interface DataSource {
	Connection getConnection() throws SQLException;
}
```
따라서 개발자는 DBCP2 커넥션 풀, HikariCP 커넥션 풀 등의 구체적인 코드에 의존하는 것이 아니라 `DataSource` 인터페이스에만 의존하도록 애플리케이션 로직을 구성하면 된다.
이후 커넥션 풀 구현 기술을 변경하고 싶을 때는 다른 변경 없이 구현체만 갈아 끼우면 된다.

단, DriverManager로 새 커넥션을 생성하는 코드는 DataSource를 사용하지 않으므로 직접 사용해야 한다.
따라서, 커넥션 풀 ↔ DriverManager 변경 시에는 애플레킹션에서 직접 관련 코드를 변경해야 할 것 같지만, 자바는 DriverManagerDataSource라는 DataSource를 구현한 클래스를 제공한다.

만약 DriverManagerDataSource를 사용중이라면 모든 변경에서 자유로운 셈이다.

## DataSource - DriverManager 사용

DataSource 사용 전 DriverManager
```java
@Slf4j
public class ConnectionTest {
	@Test
	void driverManager() throws SQLException {
		Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
		Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
		log.info("connection={}, class={}", con1, con1.getClass());
		log.info("connection={}, class={}", con2, con2.getClass());
	}
}
```

```
connection=conn0: url=jdbc:h2:tcp://..test user=SA, class=class
org.h2.jdbc.JdbcConnection
connection=conn1: url=jdbc:h2:tcp://..test user=SA, class=class
org.h2.jdbc.JdbcConnection
```

다음은 스프링이 제공하는 DataSource를 확장한 DriverManagerDataSource를 사용해보자.
```java
@Slf4j
public class ConnectionTest {
	@Test
	void driverManager() throws SQLException {
		// DataSource 없이 사용한 DriverManager
		Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
		Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
		log.info("connection={}, class={}", con1, con1.getClass());
		log.info("connection={}, class={}", con2, con2.getClass());
	}
	
	@Test
	void dataSourceDriverManager() throws SQLException {
		// DriverManagerDataSource: 항상 새로운 커넥션 획득
		DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
		useDataSource(dataSource);
	}
	
	private void useDataSource(DataSource dataSource) throws SQLException {
		Connection con1 = dataSource.getConnection();
		Connection con2 = dataSource.getConnection();
		log.info("connection={}, class={}", con1, con1.getClass());
		log.info("connection={}, class={}", con2, con2.getClass());
	}
}
```
기존 DriverManager를 통해서 커넥션을 획득하는 방법과 DataSource를 통해서 커넥션을 획득하는 방법에는 큰 차이가 있다.

DriverManager는 커넥션을 획득할 때마다 URL, USERNAME, PASSWORD 같은 파라미터를 계속 전달해야 한다.
반면에 DataSource를 사용하는 방식은 처음 객체를 생성할 때만 필요한 파라미터를 넘겨두고 커넥션을 획득할 때는 단순히 `dataSource.getConnection()`만 호출하면 된다.

즉, DataSource를 사용하는 방법은 설정(URL, ID, PASSWORD)과 사용이 분리되어 있어서 향후 변경에 더 유연하게 대처할 수 있다.

## DataSource - Connection Pool 사용

```java
@Test
void dataSourceConnectionPool() throws SQLException, InterruptedException {
	HikariDataSource dataSource = new HikariDataSource();
	dataSource.setJdbcUrl(URL);
	dataSource.setUsername(USERNAME);
	dataSource.setPassword(PASSWORD);
	dataSource.setMaximumPoolSize(10);
	dataSource.setPoolName("MyPool");
	useDataSource(dataSource);
	Thread.sleep(1000); // 커넥션 풀에서 커넥션 생성 시간 대기
}
```
HikariCP 커넥션 풀을 사용한다. HikariDataSource는 DataSource 인터페이스를 구현하고 있다.
커넥션 풀 최대 사이즈를 10으로 지정하고, 풀의 이름을 MyPool이라고 지정했다.

커넥션 풀에서 커넥션을 생성하는 작업은 애플리케이션 실행 속도에 영향을 주지 않기 위해 **별도의 스레드에서 작동**한다.

커넥션 풀에 커넥션을 채우는 것은 상대적으로 오래 걸리는 일이다. 애플리케이션을 실행할 때 커넥션 풀을 채울 때까지 마냥 대기하고 있다면 애플리케이션 실행 시간이 늦어진다. 따라서 이렇게 별도의 스레드를 사용해서 커넥션 풀을 채워야 애플리케이션 실행 시간에 영향을 주지 않는다.

별도의 스레드에서 동작하기 때문에 테스트가 먼저 종료되어 버린다. 예제처럼 `Thread.sleep()`을 통해 대기 시간을 주어야 스레드 풀에 커넥션이 생성되는 로그를 확인할 수 있다.

```
실행 결과
#커넥션 풀 전용 쓰레드가 커넥션 풀에 커넥션을 10개 채운다.
[MyPool connection adder] MyPool - Added connection conn0: url=jdbc:h2:..
user=SA
[MyPool connection adder] MyPool - Added connection conn1: url=jdbc:h2:..
user=SA
[MyPool connection adder] MyPool - Added connection conn2: url=jdbc:h2:..
user=SA
...
[MyPool connection adder] MyPool - Added connection conn9: url=jdbc:h2:..
user=SA
```

또 `useDataSource()` 메서드를 통해 커넥션 두 개를 사용중이므로, 아래와 같은 상태 정보도 알 수 있다.
```
MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)
```
