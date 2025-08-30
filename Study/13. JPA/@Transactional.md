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

## 트랜잭션 인터셉터

org.springframework.transaction.interceptor에서 낚아채서 invokeWithinTransaction이라는 메서드가 동작하게 된다.

Transaction 시작 -> Target method 실행 -> Transaction 종료

try 문 안에서 `invocation.proceedWithInvocation()`이 Target Object를 직접 호출하고 commitTransactionAfterReturning(txInfo) 동작들이 정상적으로 수행되었을 때, 커밋을 수행한다.

```java
public abstract class TransactionAspectSupport implements BeanFactoryAware, InitializingBean {

	@Nullable  
	protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,  
       final InvocationCallback invocation) throws Throwable {

...

	PlatformTransactionManager ptm = asPlatformTransactionManager(tm);  
	final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);  
	  
	if (txAttr == null || !(ptm instanceof CallbackPreferringPlatformTransactionManager cpptm)) {  
	    // Standard transaction demarcation with getTransaction and commit/rollback calls.  
	    TransactionInfo txInfo = createTransactionIfNecessary(ptm, txAttr, joinpointIdentification);  
	  
	    Object retVal;  
	    try {  
	       // This is an around advice: Invoke the next interceptor in the chain.  
	       // This will normally result in a target object being invoked. 
	         retVal = invocation.proceedWithInvocation();  
	    }  
	    catch (Throwable ex) {  
	       // target invocation exception  
	       completeTransactionAfterThrowing(txInfo, ex);  
	       throw ex;  
	    }  
	    finally {  
	       cleanupTransactionInfo(txInfo);  
	    }  
	  
	    if (retVal != null && vavrPresent && VavrDelegate.isVavrTry(retVal)) {  
	       // Set rollback-only in case of Vavr failure matching our rollback rules...  
	       TransactionStatus status = txInfo.getTransactionStatus();  
	       if (status != null && txAttr != null) {  
	          retVal = VavrDelegate.evaluateTryFailure(retVal, txAttr, status);  
	       }  
	    }  
	  
	    commitTransactionAfterReturning(txInfo);  
	    return retVal;  
	}
...
```
Transaction은 commit이나 rollback 어떤 메서드가 호출될지 모르기에, Advise 메서드를 지정할 수 없고, aspect가 아닌 advisor로 aop를 설정해야 한다.

> [!info]
> org.springframework.transaction.interceptor.TransactionAspectSupport에 있는 invokeWithinTransaction()을 사용하고 있는데, 이것이 Aspect를 사용하는 것처럼 보일 수 있지만, 실제로는 Advisor를 통해 구현된 것이다. `TrasctionInterceptor` 자체가 `MethodInterceptor`를 구현하는 Advisor의 일종이기 때문이다.
> `TransactionInterceptor`는 `invoke()` 메서드를 통해 트랜잭션을 관리하며, 이는 `invokeWithinTransaction()` 메서드를 호출하는 방식으로 동작한다.
> 
> 메서드 별로 다른 트랜잭션을 적용하려면 곧 Advise의 기능을 확장해야 한다.
> Spring에서는 메서드 패턴에 따라 경계 설정을 할 수 있도록 TransactionInterceptor를 제공한다.

## 정리

선언적 트랜잭션 관리 - @Transactional 대부분의 경우에 충분함

- @Transactional 어노테이션이 있으면, Spring AOP를 통해 프록시 객체를 생성하여 사용한다.
- Transaction Advisor에게 전달한다.
- Transaction Advisor는 트랜잭션을 생성한다.
- Transaction Advisor는 commit 또는 rollback 등의 트랜잭션 결과를 반환한다.
- `invocation.proceedWithInvocation()` Target 객체를 직접 호출
- `commitTransactionAfterReturning(txInfo)` 동작들이 정상적으로 수행되었을 때, commit을 수행한다.