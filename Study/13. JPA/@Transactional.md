## @Transactional 이란?

스프링에서 많이 사용되는 선언적 트랜잭션 방식이다.

getConnection(), setAutoCommit(false), 예외 발생 시 롤백, 정상 종료 시 커밋 등의 필요한 코드를 삽입해준다.

> [!info]
> - @Transactional 애너테이션이 있으면, 프록시 객체가 빈으로 등록된다.
>     
> - 스프링이 제공하는 선언적 트랜잭션 관리를 통해, 서비스 레이어에 트랜잭션 관련된 코드 혹은 특정 기술에 종속된 코드를 분리했다.

## Transactional 사용 방법

스프링부트에서는 **`@EnableTransactionManagement`** 설정이 되어 있어서 자동으로 사용할 수 있으며, 입맛에 맞게 클래스 또는 메서드에 @Transactional을 사용하면 된다.

- `PlatformTransactionalManage`r는 TransactionManager의 최상위 인터페이스로, 환경에 맞는 클래스를 주입할 수 있도록 구성되어 있다.
- `@Transactional` 어노테이션이 있으면, 해당 빈을 상속받은 프록시 객체를 생성한다. 따라서 **private메서드는 상속이 불가능하기 때문에, 어노테이션을 붙여도 동작하지 않는다**.

## 동작원리

Spring AOP를 통해 프록시 객체를 생성하여 사용한다.
스프링에서 직접 참조하지 않고, 프록시 객체를 사용하는 이유는 **Aspect 클래스에서 제공하는 부가기능을 사용하기 위해서**다. 직접 참조하는 경우는 원하는 위치에서 직접 Aspect 클래스를 호출해야 하기 때문에 유지보수가 어려워진다.

- Target에 대한 호출이 오면, AOP 프록시가 인터셉터 체인을 통해 가로채온 후 Transaction Advisor에게 전달한다.
- Transaction Advisor는 트랜잭션을 생성한다.
- Custom Advisor가 있다면 실행한 후 비즈니스 로직을 호출
- Transaction Advisor는 커밋 또는 롤백 등의 트랜잭션 결과를 반환한다.
프록시를 통해 실제 객체를 호출하여 트랜잭션을 처리하는 도중 RuntimeException이 발생하면 처리를 종료하고 rollback을 수행한다. rollbackFor 옵션을 주어 처리하는 방법도 있다.
![[Pasted image 20250830232748.png]]