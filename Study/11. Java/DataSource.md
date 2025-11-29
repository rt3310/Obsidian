## DataSource

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