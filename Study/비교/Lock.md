https://jaeseo0519.tistory.com/399#8.%20%3CLock%20Free%3E%20Redis%3A%20Sorted%20Set-1

## 예시

쿠폰을 300개 발행하고, 300명이 한 장씩 발급받는 단순한 로직이다.
보기엔 아무런 문제가 없어보이지만, 멀티 스레드 환경에선 보장할 수 없다.
```java
@Slf4j
@SpringBootTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class CouponDecreaseTest {
	@Autowired private CouponDecreaseService couponDecreaseService;
	@Autowired private CouponRepository couponRepository;
	private Coupon coupon;
	
	@BeforeEach void setUp() {
		coupon = new Coupon("COUPON_001", 300L);
		couponRepository.save(coupon);
	}
	
	@Test
	@DisplayName("동시성 환경에서 300명 쿠폰 차감 테스트")
	void 쿠폰차감_동시성_300명_테스트() throws InterruptedException {
		// given
		int threadCount = 300;
		ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
		CountDownLatch latch = new CountDownLatch(threadCount);
		
		// when
		for (int i = 0; i < threadCount; i++) {
			executorService.submit(() -> {
				try {
					couponDecreaseService.decreaseStock(coupon.getId());
				} finally {
					latch.countDown();
				}
			});
		}
		latch.await();
		
		// then
		Coupon persistedCoupon = couponRepository.findById(coupon.getId())
				.orElseThrow(IllegalArgumentException::new);
			
		assertThat(persistedCoupon.getAvailableStock()).isNotZero(); // 잔여 쿠폰 수량은 0이 아니다.
		log.debug("잔여 쿠폰 수량: " + persistedCoupon.getAvailableStock());
	}
}
```
선언적 Tx가 보장해주는 것은 원자성일 뿐이지, 동시성 문제까지 제어할 수 있음을 의미하진 않는다.

couponId로 Entity를 조회할 때 여러 Thread가 동일한 stock 개수를 담았고, 거기서 -1을 한 채 update를 수행하면 중복이 발생하게 된다.

이는 공유 자원에 대해 여러 Thread 혹은 Process가 접근하려 할 때 발생하는 경쟁 조건(Race Condition)에 의해 발생하는 문제이며, 데이터의 일관성과 정확성을 해칠 수 있는 critical issue가 된다

## synchronized method/block

Java에서는 멀티 쓰레드 환경의 동시성 문제를 해결하기 위해 synchronized 키워드를 제공하며, 이를 사용하면 쉽게 동시성 문제를 해결할 수 있다.

하지만 선언적 Tx를 혼용할 때는 Silver Bollet이 될 수 없다.

### 이게 왜 실패하지?
```java
@Transactional
public synchronized void decreaseStockWithSynchronized(Long couponId) {
	Coupon coupon = couponRepository.findById(couponId)
			.orElseThrow(() -> new IllegalArgumentException("존재하지 않는 쿠폰입니다.")); 
	coupon.decreaseStock();
}
```

```java
@Test @DisplayName("synchronized: 동시성 환경에서 300명 쿠폰 차감 테스트")
void synchronized_쿠폰차감_동시성_300명_테스트() throws InterruptedException {
	performConcurrencyTest(
			300,
			coupon.getId(),
			couponDecreaseService::decreaseStockWithSynchronized,
			true
	);
}
```
![[Pasted image 20251101120619.png]]반드시 성공할 거라 생각했던 테스트가 실패함을 볼 수 있다.

엥, synchronized가 동시성을 보장한다면서요? 🤔

틀린 말이 아니다.

Java에서 제공하는 synchronized 키워드에 의한 동기화는 특정 객체에 대한 Lock을 획득하고, 해당 Lock이 해제될 때까지 다른 스레드들이 Lock을 획득하려고 하는 것을 방지한다.

또한, main memory와 thread가 작업하는 local memory 사이 일관성도 보장한다.

synchronized 블록에 진입 혹은 빠져나올 때, 모든 local cache(스레드가 보유한 변수 복사본)가 main memory와 동기화되도록 하여, Thread가 최신 데이터를 볼 수 있도록 하기 때문이다.

그렇다면 위 테스트는 왜 실패하는 것일까?

[Using @Transactional :: Spring Framework](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
![[Pasted image 20251101120725.png]]
선언적 Tx는 spring의 **AOP** 기능을 활용한다.

심지어 실제 메서드를 사용하는 것이 아니라 **Proxy mode**로 동작한다.
(선언적 tx가 private 메서드에 적용될 수 없는 이유도 이 때문이다. Proxy를 사용하려면 오버라이딩이 가능해야 하는데, private 제한자를 쓰면 이게 불가능해진다.)

AOP 기능을 활용한다고 하면, 선언적 Tx를 메서드 단위에 붙였을 경우, Advisor 쪽에서 메서드의 시그니처 정보(메서드 이름, 파라미터 등)를 가지고 Transaction을 동작시키게 된다.

즉, 다음과 같은 일련의 순서를 같게 된다.
> **Call → AOP Proxy → Transaction Manager → … → target Method**

이게 왜 문제가 되느냐?

**synchronized는 메서드 시그니처가 아니기 때문에 상속이 되지 않는다.**

실제 Advisor가 실제 실행하는 메서드는 Proxy 객체인데, 여기에 synchronized 키워드가 없기 때문에 동시성 제어가 되지 않는 것이다.

### 해결책1. 선언적 Tx 제거
```java
public synchronized void decreaseStockWithSynchronized(Long couponId) {
	Coupon coupon = couponRepository.findById(couponId)
			.orElseThrow(() -> new IllegalArgumentException("존재하지 않는 쿠폰입니다.")); 
	coupon.decreaseStock();
	couponRepository.save(coupon);
}
```
![[Pasted image 20251101120941.png]]
더 이상 _decreaseStockWithSynchronized_() 메서드 레벨에서 Proxy를 사용하지 않으므로 성공함을 알 수 있다.

하지만 더 이상 Jpa의 영속화 기능을 사용할 수 없어서 성능 상 나빠졌다.

#### 주의: 트랜잭션과 락 시점
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class CouponDecreaseService {
	private static final Object lock = new Object();
	
	@Transactional
	public void decreaseStockWithSynchronizedAndTx(Long couponId) {
		synchronized (lock) {
			Coupon coupon = couponRepository.findById(couponId)
					.orElseThrow(() -> new IllegalArgumentException("존재하지 않는 쿠폰입니다.")); 
			coupon.decreaseStock();
		}
	}
}
```
이번에는 synchronized 메서드가 아니라 block을 사용했으므로, Proxy에서도 적절히 처리할 수 있을 것이라 기대해볼 수 있을 것이다.

하지만 이 테스트는 실패한다
이유는 Transaction이 commit되기 전에 lock이 해제되기 때문에 여전히 다른 Thread에서 잘못된 값을 읽을 여지가 존재하기 때문이다.

**Lock의 해제는 언제나 Transaction commit 이후**여야 한다.

### 해결책2. 외부에서 메서드 호출
```java
@Service
@RequiredArgsConstructor
public class CouponService {
	private final CouponDecreaseService couponDecreaseService;
	
	public synchronized void decreaseStock(Long couponId) {
		couponDecreaseService.decreaseStock(couponId);
	}
}
```
![[Pasted image 20251101125621.png]]
외부에서 synchronized를 걸고, Tx가 걸린 메서드를 호출하는 방식을 사용할 수 있다.

코드 복잡도는 조금 높아졌지만, 영속화 기능을 사용하여 속도가 50%정도 빨라졌다.

## ReentrantLock

java.util.concurrent.locks 패키지에서 제공하는 가장 일반적인 Lock이며, synchronized 보다 세밀하게 락을 제어할 수 있다.
- Lock polling 지원
- 타임 아웃 지정 가능
- Condition을 적용해 대기 중인 스레드를 선별적으로 깨울 수 있음
- lock 획득을 위해 waiting pool에 있는 스레드에게 interrupt를 걸 수 있음.

이보다 많은 기능들이 있지만, 가장 중요한 것은 **CPU cache와 main memory 간의 동기화를 명시적으로 제어**할 수 있게 된다는 점이다.
ReentrantLock은 Lock 획득/방출 시, 다음과 같이 동작한다.
- Lock 획득: main memory에서 최신 데이터 읽음
- Lock 방출: 변경 사항을 main memory에 반영

그리고 이 과정에서 **CPU cache 일관성을 유지하기 위한 memory barrier 작업을 자동으로 수행**한다.
```java
@Service
@RequiredArgsConstructor
public class CouponService {
	private final CouponDecreaseService couponDecreaseService;
	private final ReentrantLock lock = new ReentrantLock();
	
	public void decreaseStockWithReentrantLock(Long couponId) {
		lock.lock();
		try {
			couponDecreaseService.decreaseStock(couponId);
		} finally {
			lock.unlock();
		}
	}
}
```
![[Pasted image 20251101125800.png]]
위에서 배운 교훈을 잊지 않았다면, lock을 걸어주는 메서드에 선언적 Tx를 걸 수 없다는 것 또한 이해했을 것이다. 성능 또한 준수한 편이며, ReentrantLock이 제공해주는 메서드를 사용하여 Tx 획득 대기 시간이 지났을 경우 재시도 로직 등을 수행할 수 있다.

여기선 _lock_()을 사용했는데, 이렇게 하면 **lock을 얻을 때까지 스레드를 block시키므로 context switch에 따른 overhead가 발생할 수 있다**.

하지만 Critical Section 접근 시도 수행시간이 매우 짧다면, **`tryLock()`** 을 사용했을 때 SpinLock 방식이 적용되어 효율적으로 lock을 걸 수 있다.

> [!info] 수행 시간이 짧을 경우에만 SpinLock을 적용하는 이유
>   
> SpinLock은 Thread가 Lock을 얻을 때까지 무한 루프를 돌며 확인(Busy waiting)하는 메커니즘을 따른다.
> 
> 이런 방식은 OS가 스케줄링 지원을 받지 않기 때문에 context switching이 발생하지 않는다. 따라서 Lock의 획득이 빠르며, Context Switching으로 발생하는 오버헤드를 줄일 수 있다.  
>   
> 문제는 대기 시간이 길어질 수록, 무한 루프를 통해 CPU 자원을 계속 소모하므로 다른 오버헤드가 발생한다. 그리고 특정 Thread나 Process가 공유 자원을 오랜 시간 점유하면, 다른 Thread가 대기 상태에 갇히는 기아(Starvation) 상태가 된다.  
>   
> 따라서 Critial Section의 경합 상황이 짧음을 보장할 수 있는 경우에만, _tryLock_()을 통해 최적화를 이룰 수 있는 것. 물론 CPU 코어 개수가 많을 수록 더 좋다. **CPU 코어가 하나라면 SpinLock은 피해야 한다**.

## CAS(Compare-And-Swap) Algorithm

> [!info] Lock 문제점
> **Lock에 의한 Thread 차단의 비용은 비싸다.**

![[Pasted image 20251101130225.png]]
synchronized와 ReentrantLock은 Lock을 사용해 동시성 문제를 해결하는 기법이며, 이러한 blocking 방식은 언제나 **성능 이슈**를 빼놓을 수 없다. SpinLock이 됐건, Pub/Sub 방식이 됐건 Critical Section에 접근하려는 Thread는 blocking 상태에 들어가 아무 작업도 하지 못한 채 자원을 낭비하기 때문이다. 그저 Context Switching이냐, Busy waiting이냐를 두고 고민할 수밖에 없다.

전통적인 Lock 기법의 동기화 대신, 더 높은 동시성을 달성하고 성능 병목을 줄이기 위해 고안된 방법이 **Lock-Free 프로그래밍 기법**이다. 그리고 이 방법을 실현하기 위해, atomic 변수를 활용한다.

### CAS 동작 원리
메모리 위치 값을 확인(Compare)하고, 예상되는 값이 현재 메모리 위치에 저장된 값과 일치하는 경우만 새로운 값으로 업데이트(Swap)한다. 이 과정은 모두 원자적(atomic)으로 수행된다.
![[Pasted image 20251101130335.jpg]]![[Pasted image 20251101130342.png]]
1. **기존 값**(Compared Value)을 읽어 **변경할 값**(Exchanged Value)을 계산한다.
2. **기존 값**이 **현재** **메모리의 값**(Destination)과 같다면, **변경할 값**으로 교체한다.
3. **기존 값**이 **현재 메모리의 값**과 다르다면, 값을 변경하지 않거나 (1)부터 재시도한다. (다른 Thread가 먼저 공유 자원을 변경한 경우)

여기서 중요한 점은, 비교(Compare)와 교환(Swap) 과정은 원자적으로 이루어지며, 다른 어떠한 연산도 개입할 수 없어야 한다.

> [!info] 원자적(atomic)
> 
> 분할 불가능한 단일 작업으로 수행되는 하나의 연산 단위.
> 원자적 연산은 동시성 문제가 발생할 수 있는 환경에서도 시스템의 일관성과 데이터 경쟁 조건을 방지할 수 있다.  
> 
> 예를 들어, i++ 연산은 원자적일까? 그렇지 않다. 코드에서는 고작 한 줄이지만, 실제로는 "읽기-수정하기-쓰기" 세 단계로 구성되기 때문이다. 이 과정에 다른 Thread가 개입하면, 값이 예상과 다르게 변경될 수 있다

java에서는 atomic 관련 라이브러리를 제공해주는데, 업데이트 해야하는 값이 long 타입이었으므로 atomicLong의 코드를 대표로 살펴보자.
```java
public class AtomicLong extends Number implements java.io.Serializable {
	 ...
	 /*
	   * This class intended to be implemented using VarHandles, but there
	   * are unresolved cyclic startup dependencies.
	   */
	private static final Unsafe U = Unsafe.getUnsafe();
	private static final long VALUE = U.objectFieldOffset(AtomicLong.class, "value");
	private volatile long value;
	public AtomicLong(long initialValue) {
		value = initialValue;
	}
	
	...
	
	public final long get() {
		return value;
	}
	
	public final long getAndDecrement() {
		return U.getAndAddLong(this, VALUE, -1L);
	}
	...
}
```
여기서 **AtomicLong은 value를 volatile로 관리**한다.

이는 64-bit JVM 환경이라면 long 타입은 그 자체로 원자성을 보장하지만, **가시성을 보장하지 않는 문제를 해소하기 위함**이다. 따라서 AtomicLong 타입의 변수를 volatile로 지정해줄 필요는 없다.

```java
public final class Unsafe {
	...
	
	@IntrinsicCandidate
	public final long getAndAddLong(Object o, long offset, long delta) {
		long v;
		do {
			v = getLongVolatile(o, offset);
		} while (!weakCompareAndSetLong(o, offset, v, v + delta));
		return v;
	}
	...
}
```
값을 갱신할 때 사용하는 메서드들은 Unsafe 클래스를 사용하는데, 내부에서 CAS 알고리즘의 로직을 구현하고 있다.

1. `getLongVolatile()`로 현재 값 v를 읽는다.
2. `weakCompareAndSetLong()`로 기존 값이 현재 메모리 값과 같다면 v + delta로 업데이트하고 true를 반환한다.
![[Pasted image 20251101163823.png]]
확인하는 함수가 while문으로 감싸져 있는 이유는 다른 Thread에서 값을 수정하여 실패한 경우, 다시 현재값을 memory에서 읽어 업데이트를 하기 위함이다.

SpinLock 방식과 비슷해보이지만 다르다.

Lock 기법은 임의의 Thread가 Lock을 획득하면, 다른 모든 Thread가 running - blocked로 상태로 상태가 변경되어야 한다.

하지만 atomic 방식은 무의미한 무한 루프를 돌게 되더라도, true를 반환 받는 순간 Thread 상태를 변경하는 일 없이 바로 이후 작업을 수행할 수 있다.

### 테스트 실패: 동시성을 지키지 않은 사용 방법
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class AtomicCoupon {
	...
	
	/**
	  * 사용 가능 재고수량
	  */
	private AtomicLong availableStock;
	
	public void decreaseStock() { // 실패하는 코드. 비교와 수정 원자적이지 않음.
		validateStock();
		this.availableStock.decrementAndGet();
	}
	
	private void validateStock() {
		if (availableStock.get() < 1) {
			throw new IllegalArgumentException("재고가 부족합니다.");
		}
	}
}
```

![[Pasted image 20251101164058.png]]
기존의 long 타입 변수를 AtomicLong으로 바꾼다고 해서 문제가 해결되진 않는다.

1. `validateStock()`과 `decrementAndGet()` 메서드는 각각 원자적이라 볼 수 있겠지만, 둘을 서로 원자적이지 않다.
2. AtomicLong은 JVM 메모리 내에서 동작하고, JPA의 영속성 컨텍스트와 동기화되지 않는다.
3. AtomicLong의 변경은 즉시 발생하지만, Entity 변경 사항은 트랜잭션 커밋 시점에 DB에 반영된다.

#### timestamp 필드 추가
```java
private AtomicReference<LocalDateTiem> lastTryAt = new AtomicReference<>(LocalDateTime.MIN);

@Transactional
public AtomicCoupon decreaseStockWithAtomic(Long couponId) {
	AtomicCoupon coupon = atomicCouponRepository.findByIdWithOLock(couponId)
			.orElseThrow(() -> new IllegalArgumentException("존재하지 않는 쿠폰입니다."));
	
	LocalDateTime now = LocalDateTime.now();
	LocalDateTime last = lastTryAt.get();
	
	if (Duration.between(last, now).toMillis() < 1000) {
		// 중복 요청 예외 처리
	}
	if (!last.compareAndSet(last, now)) {
		// CAS 타임스탬프 업데이트 예외 처리 (동시 요청 발생)
	}
	
	coupone.decreaseStock();
}
```
위처럼 JPA 환경에 종속되지 않는 별도의 Atomic 변수를 만들면 어느정도 성공을 보장하게 만들 수는 있을 것이다.

## Optimistic Lock

### 다중화 된 서버 환경
지금까지 나왔던 방법은 나름대로 동시성 문제를 해결하고 있지만 큰 취약점이 있다.

단일 애플리케이션 환경의 멀티 스레드 환경에선 문제가 없겠지만, **out-scale을 통해 서버가 다중화 된 환경에선 보장할 수 없다는 점**이다.
![[Pasted image 20251101164348.png]]
1번 애플리케이션, 2번 애플리케이션 각각은 동시성 문제를 제어할 수 있겠지만 전체 시스템 환경에선 그렇지 않다. 왜냐하면 동시성 제어를 위한 Lock과 Lock-free 방법 모두 실행 중인 장치 내의 메모리 혹은 Lock을 통해서 동기화 되어 있기 때문이다.

이를 해결하기 위해선 모든 서버의 동시성을 핸들링 해줄 외부 시스템(DB, Redis 등)이 필요해진다.

> [!info] 낙관적 락
> 자원에 Lock을 걸지 않고, 동시성 문제가 발생하면 그때 처리하자!

![[Pasted image 20251101164438.png]]
낙관적 락은 DB의 Lock을 사용하지 않고, 애플리케이션 레벨에서 **Entity의 버전을 관리**하면서 변경을 감지하는 방법이다.

DB에서 값을 읽고 UPDATE를 하려고 할 때, WHERE 절에 바꾸려는 version 정보를 함께 보낸다.

만약, 다른 Thread에서 값을 수정해버렸다면 버전이 바뀌어있을 것이고, 그럼 UPDATE 하려는 row를 찾지 못해 예외가 발생한다.

[동시성 문제 해결하기 V1 - 낙관적 락 feat.데드락 첫 만남](https://velog.io/@znftm97/%EB%8F%99%EC%8B%9C%EC%84%B1-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-V1-%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BDOptimisitc-Lock-feat.%EB%8D%B0%EB%93%9C%EB%9D%BD-%EC%B2%AB-%EB%A7%8C%EB%82%A8)
참고로 낙관적 락은 경우에 따라 Dead Lock이 걸리기 딱 좋으므로 조심해야 한다. 주로 FK가 걸려있는 테이블인 경우가 그렇다.

낙관적 락의 단점은 재시도 로직을 만들어야 한다는 점인데, 이는 **충돌이 발생했을 때** DB가 아닌 **Application 단에서 처리**해야 한다는 특징 때문이다.

낙관적 락은 경합이 많고 충돌이 많을 수록 트랜잭션을 중단할 가능성이 매우 크고,

롤백은 테이블 행과 레코드를 모두 포함할 수 있는 현재의 보류 중인 변경 사항을 모두 되돌려야 하므로 DB 시스템 비용이 많이 들 수 있다.

## Pessimistic Lock

> [!info] 비관적 락
> 동시성 문제가 발생하기 전에, 자원의 접근을 막아버리자.

![[Pasted image 20251101164933.png]]
비관적 락은 일단 동시성 문제가 발생할 거라 가정하고, 자원에 대한 다른 접근을 막고 시작한다.

따라서 Transaction이 시작할 때 공유 락 혹은 배타적 락을 걸어버린다.
- **공유 락**(Shared Lock): Read Lock이라고도 부르며, 데이터를 읽을 때는 같은 공유 락끼리 접근을 허용하지만 write 작업은 막는다.
- **배타적 락**(Exclusive Lock): Write Lock이라고도 부르며, 트랜잭션이 완료될 때까지 유지되면서 배타락이 끝나기 전까진 read/write 작업을 모두 막는다.

### 테스트
```java
public interface CouponRepository extends JpaRepository<Coupon, Long> {
	@Lock(LockModeType.PESSIMISTIC_WRITE)
	@Query("SELECT c FROM Coupon c WHERE c.id = :id")
	Optional<Coupon> findByIdWithPLock(Long id);
}
```
![[Pasted image 20251101165023.png]]

![[Pasted image 20251101165035.png]]
PLock을 쓰면 정말 간단하게 다중화된 서버 환경의 동시성 문제를 해결할 수 있다.
심지어 성능도 엄청 빠르니, 무조건 비관적 락이 이득이 아닐까?

배타적 락은 특정 데이터에 배타적 락(Exclusive Lock)을 걸어, 하나의 처리가 완료되기 전까진 해당 데이터의 읽기, 수정, 삭제를 방지하기 때문에 동시성 문제 측면에서 매우 안전하긴 하다.

하지만, **모든 작업을 순차적으로 처리하기 때문에 속도가 매우 저하되고, 특정 데이터의 조회까지 막기 때문에 전혀 상관없는 기능에서 조차 병목 현상이 발생할 수 있다는 단점**이 존재한다.

반대로 **충돌이 많이 발생할 수 있는 환경이라면 낙관적 락보다는 비관적 락이 적합**하다.

## Distributed Lock

![[Pasted image 20251101165114.png]]
낙관적 락은 데이터의 쓰기 작업은 별로 없지만, 읽기 작업이 많아 동시 접근 성능이 중요할 때 많이 쓰이고
비관적 락은 충돌이 많이 발생하더라도 데이터의 무결성을 지킬 수 있다는 장점을 갖는다.

하지만 충돌은 충돌대로 막으면서, DB의 부담도 줄이고, 읽기 조회의 병목 현상도 줄일 수 있는 방법이 없을까?
가장 떠올리기 쉬운 대안책은 Lock의 위치를 옮기는 것이다.

DB보다 훨씬 접근이 빠른 Redis를 사용하면서, 특정 작업에서만 동시성이 관리되도록 처리하는 방법이다.

### 분산 락 개발 시 주의할 점
만약 분산 락을 SpinLock 방식으로 구현했다고 가정하자.
그렇다면 반드시 "락이 존재하는 지 확인한다"와 "존재하지 않으면 락을 획득한다"라는 두 연산이 atomic하게 이루어져야 한다.
그리고 try 구문 안에서 Lock 획득에 성공할 때까지 무한 루프를 실행해야 한다.

```java
boolean tryLock(String key) {
	return command.setnx(key, "1");
}

void unLock(String key) {
	command.del(key);
}

void process(String key) {
	try {
		while (!tryLock(key)) {
			// Lock 획득 실패 시 처리
		}
		
		// Critial Section
	} finally {
		unLock(key); // 작업 종료
	}
}
```
이 방식의 문제점이 무엇일까?

#### 1. Lock Timeout
- 어떤 애플리케이션에서 tryLock에 성공해서 **자원 접근을 막았는데 종료되어버리면, 다른 모든 애플리케이션도 영원히 Lock을 얻지 못하는 Dead Lock 현상이 발생**한다.
	- 이 문제를 해결하려면 Lock의 TTL을 설정해주어야 하는데, `setnx()` 명령어로는 이러한 작업을 처리할 수가 없다.
- 굳이 종료까진 아니더라도 연산이 오래 걸리는 작업이라면, 모든 Thread가 Lock을 대기하는 상태가 되어 응답 속도가 현저하게 줄어들 수 있다.
	- 이 경우엔 Lock 시도 횟수에 제한을 걸어주는 식으로 완화할 수 있을 것이다.

#### 2. tryLock은 try-finally 밖에서 수행해야 한다.
- 1번처럼 try 문에서 시도 횟수 초과에 대한 예외를 발생시키면, Lock을 얻지도 못 한 Thread에서 Lock을 해제시킬 수 있게 된다.
- 따라서 try-finally 밖에서 Lock 획득 시도 횟수 초과 예외를 처리해주어야 한다.
```java
void process(String key) {
	int maxTry = 5, curTry = 0;
	while (!tryLock(key)) {
		if (++curTry == maxTry) {
			// 시도 횟수 초과 시 예외 처리
		}
		
		// Lock 획득 실패 시 처리
	}
	
	try {
		// Critial Section
	} finally {
		unLock(key); // 작업 종료
	}
}
```

#### 3. Redis 부하
- SpinLock 방식은 Redis에 엄청난 부담을 줄 수밖에 없다.
    - Critical Section내에서 수행할 작업 속도가 느릴 수록, 빈번히 수행되는 기능일 수록 문제는 더 심해진다.
- Lock 획득 실패 시, Thread를 임시로 sleep시키는 것은 좋은 해결책이 될 수 없다.
    - **처리 시간이 10ms인 작업에 대한 Lock을 획득하지 못 해, Thread를 100ms 동안 sleep시킨다는 것은 매우 비효율적**이다. (그렇다면 최적의 sleep 시간을 어떻게 결정할 것인가?)
    - **Thread를 sleep 상태에서 다시 전환하는 Context Switching 오버헤드 비용** 또한 심하다.
    - 요청을 수용하는 방법이 불공정하다. 먼저 도착한 요청이 **대기 상태에 돌입했을 때, 나중에 도착한 요청이 Lock을 먼저 획득할 우려**가 있다. (더 심한 건 먼저 도착한 요청이 바로 다음에 Lock을 획득할 수 있음을 보장하지도 않는다.)

### Redisson 내부 구현
[Redisson](https://jaeseo0519.tistory.com/381)

#### Lock Time-out
```java
public interface RLock extends Lock, RLockAsync {
	...
	boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException;
	...
}
```
- Redisson은 tryLock 메서드에서 TTL을 명시하도록 명세가 적혀있다.
    - waitTime: 얼마나 Lock 획득을 대기할 것인가?
    - leaseTime: Lock을 언제까지 가지고 있을 수 있는가?

무한 루프의 위험성을 쉽게 제거할 수 있게 되었다.

#### Pub/Sub
![[Pasted image 20251101170048.png]]
```java
/**
  * Distributed implementation of {@link java.util.concurrent.locks.Lock}
  * Implements reentrant lock.<br>
  * Lock will be removed automatically if client disconnects.
  * <p>
  * Implements a <b>non-fair</b> locking so doesn't guarantees an acquire order.
  * @author Nikita Koksharov
  */
public class RedissonLock extends RedissonBaseLock {
	...
	protected final LockPubSub pubSub;
	...
}
```
- Redisson도 SpinLock 방식을 지원은 하지만, 기본적으로 RedissonLock을 사용한다. (RLock 로그 찍어봤음.)
- 그리고 RedissonLock은 pub/sub 기능을 사용함으로써 Redis에 가해지는 부하를 감소시킨다.
    - Lock을 획득하려고 시도했으나 실패하면, 해당 Lock에 대해 구독을 신청하고 대기 상태로 전환한다.
    - Critical Section에서 작업을 하던 Thread가 Lock을 해제하면, 구독 신청한 모든 Thread에게 알린다.
    - 타임아웃이 지나면 Lock 획득에 실패하고 작업을 중단한다.

#### Lua Script
![[Pasted image 20251101170323.png]]
여기가 가장 놀랐던 부분인데, Redisson은 내부적으로 Lua Script를 사용하고 있다.
Lua Script에 대한 설명은 밑에서 다시 할 예정이라, 지금은 Redis **연산의 원자성을 보장**한다는 것만 알면 된다.

락의 존재를 확인하고, 획득하는 연산이 원자적(atomic)으로 수행되지 않으면 다음과 같은 문제가 발생할 수 있다.
- Lock 획득이 가능함을 확인했는데, 다른 Thread가 Lock을 가로채버린다면, 현재 Thread는 Lock 획득 실패 에러가 발생한다.
- Lock의 해제와 pub/sub 알림 또한 atomic 해야 하는데, Lock이 해제되고 바로 다른 Thread에서 Lock을 획득했을 때조차 대기 중이던 Thread들에게 Lock 획득을 시도해도 된다는 알림이 갈 수도 있다.

하지만 Redisson에선 내부적으로 루아 스크립트를 사용함으로써, 위 명령어들의 원자성을 보장하고 있다.

### 테스트
![[Pasted image 20251101170408.png]]
전에는 분산 락을 적용하니 6sec나 걸려서, 비관적 락에 비해 3배나 오래 걸리는 게 심하다 싶었는데
어째 이번엔 4sec 30ms 정도밖에 소요되지 않았다. (심지어 재실행하니까 3sec 46ms..^^)

아마도 test container가 아닌, 로컬 환경의 DB와 Redis를 사용하도록 테스트 환경을 구성해서 조금 더 빨라진 듯하다.

하지만 덕분에 비관적 락 방식도 1sec 7ms 정도밖에 걸리지 않으므로, 여전히 성능은 3배 정도 느리다.

## Redis: Sorted Set

지금까지는 DB 자원에 write 연산을 하기 위해서, 공유 자원 접근을 제어하는 방법에 대해서만 생각을 했다.
하지만 잘 생각해보면 굳이 DB에 바로 반영을 할 필요가 있을까?

이게 무슨 소린가 싶겠지만, RDB에 빈번한 CUD 연산을 전달하는 건 별로 좋지 않다. 특히 index가 많은 Table일 수록, 이 문제는 심한 부작용을 낳을 수 있다.

하지만 지금처럼 단순히 쿠폰 발급이 끝인 작업이라면?
요청에 대한 **Transaction 그 자체를 저장**해두었다가, 발급이 끝나면 마지막에 batch insert로 데이터를 삽입해버리는 게 훨씬 DB에 대한 부담이 줄어든다.

### Redis 정렬 집합 자료 구조
> [!info]
> 하나의 Key에 중복을 허용하지 않는 여러 value를 score 순으로 데이터를 정렬하는 자료구조

Redis에서 제공하는 ZSet은 순서가 보장되는 고유한 문자열(member)들의 컬렉션이다.
그리고 각 member에 연관된 점수(score)를 기준으로 정렬시키는데, 만약 점수가 같다면 member의 알파벳 순으로 정렬된다.

일반적으로 리더보드, 처리율 제한 장치를 만들 때 자주 사용한다.

### 자주 사용하는 명령어
- ZADD
    - 새로운 멤버와 그에 연관된 점수 추가.
    - 이미 멤버가 존재하면 점수를 업데이트
- ZRANGE
    - 정렬되어 있는 멤버들 중 특정 범위 반환
    - 시간 복잡도가 O(logN + M)이므로, 멤버가 수만 개 이상 넘어간다면 성능면에서 비효율적이므로 사용에 주의해야 한다.
- ZRANK
    - 오름차순 기준으로 특정 멤버의 순위 반환
- ZREVRANK
    - 내림차순 기준으로 특정 멤버가 몇 번째에 위치해있는지 순위 반환

### Lua Script
Lua Script는 Redis 서버에 내장된 Lua 인터프리터에서 EVAL 명령어(혹은 EVALSHA 명령어)를 이용하여 임의의 명령어 조합 처리를 수행하는 기능이다. (Redis 2.6 이상부터 지원)
Lua Script의 가장 큰 강점은 조합 뿐만 아니라, **스크립트 자체가 하나의 명령어로 해석되어 원자적(atomic)으로 처리**된다.

Redis는 스케줄링 쪽은 Multi Thread지만, I/O 부분은 Single Thread기 때문에 동시성 문제를 고려할 필요가 있을까 싶을 수도 있다.
하지만, 메서드가 상태 의존적(ex. 쿠폰을 1 차감하기 전에 가능 여부를 먼저 판단)이어야 하는 경우 두 명령 사이는 언제든지 다른 Thread가 침입하여 동시성 문제를 발생시킬 수 있다.

다만 중요한 것은 **Lua Script는 최대한 가벼운 연산**으로 조합해야 한다. 그렇지 않으면 성능에 큰 저하를 불러일으킨다.

또한 Lua Script를 사용하면 단위 테스트를 매우 힘들게 만들기 때문에 리팩토링과 확장성 면을 따지면, 변경이 잦은 시스템에 점진적으로 부담을 줄 수도 있다.

#### 테스트
```java
String luaScript =
	"local count = redis.call('ZCARD', KEYS[1]) " +
		"if count < tonumber(ARGV[2]) then " +
		"    redis.call('ZADD', KEYS[1], tonumber(ARGV[3]), ARGV[1]) " +
		"    return 1 " +
		"else " +
		"    return 0 " +
		"end";
```
우선 사용할 Lua Script 정의해주었다.

- local count = redis.call('ZCARD', KEYS[1])
    - ZCARD 명령어로 KEYS[1]에 해당하는 정렬 집합의 멤버 수를 가져온다.
    - KEYS[1] == 정렬 집합의 키 (여기선 couponId + ":" + couponName 조합으로 구성했다.)
    - 이 값을 count에 저장한다.
- if count < tonumber(ARGV[2]) then
    - count가 ARGV[2] 보다 작은 지 확인한다.
    - ARGV[2] == MAX_REQUESTS (쿠폰 발급 가능 개수, 여기선 300개)
    - tonumber() 함수는 문자열을 숫자로 변환하여 반환한다.
- redis.call('ZADD', KEYS[1], tonumber(ARGV[3]), ARGV[1])
    - 위 조건이 참일 때 실행되며, ZADD 명령어를 사용해 새로운 멤버를 정렬 집합에 추가한다.
    - ARGV[3] == SCORE (여기선 현재 시간(milliseconds)을 담을 예정)
    - ARGV[1] == Thread Id (사용자 식별 정보을 알 수 없으므로, 임시로 thread id값을 담음) 
- return 1 else return 0
    - 새 멤버가 성공적으로 추가되면 1을 반환, 이미 최대 요청 수에 도달했으면 0을 반환한다.

키를 통해 정렬 집합을 찾고, 사이즈를 확인해 삽입 가능 여부를 확인한 후, 가능하면 삽입하는 모든 과정을 원자적(atomic)으로 만들었다.

Script를 작성하면 다음은 매우 쉽다.

우선 요청에 따라 Tx를 저장하는 로직을 구성하면 다음과 같다.
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class CouponDecreaseService {
	private final RedisTemplate<String, String> redisTemplate;
	
	public boolean registerCouponRequest(Long couponId, String key) {
		String threadId = String.valueOf(Thread.currentThread().getId());
		String maxRequestCount = "300";
		
		String luaScript =
				"local count = redis.call('ZCARD', KEYS[1]) " + 
					"if count < tonumber(ARGV[2]) then " +
					" redis.call('ZADD', KEYS[1], tonumber(ARGV[3]), ARGV[1]) " +
					" return 1 " +
					"else " +
					" return 0 " +
					"end";
		
		RedisScript<Long> script = RedisScript.of(luaScript, Long.class);
		Long result = redisTemplate.execute(script,
				Collections.singletonList(couponId + ":" + key),
				threadId,
				maxRequestCount,
				String.valueOf(System.currentTimeMillis())
		);
		
		if (result == 1) {
			log.info("쿠폰 {} 요청이 등록되었습니다. (Thread ID: {})", couponId, threadId);
			return true;
		} else {
			log.info("쿠폰 {} 요청 한도에 도달했습니다. (Thread ID: {})", couponId, threadId);
			return false;
		}
	}
}
```
위 메서드로 요청을 받고 나면, Redis에 저장한 정렬 집합에는 300개의 Tx가 쌓여있을 것이다.

이걸 테스트 케이스에서 직접 호출해서 조작해줘도 상관 없지만, 나는 별도의 서비스를 하나 더 만들어주었다.
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class CouponTransactionSaveService {
	private final CouponRepository couponRepository;
	private final RedisTemplate<String, String> redisTemplate;
	
	@Transactional
	public void saveAll(Long couponId, String suffix) {
		String key = couponId + ":" + suffix;
		
		ZSetOperations<String, String> command = redisTemplate.opsForZSet();
		Set<String> tx = command.range(key, 0, -1);
		Long sz = command.zCard(key);
		
		log.info("저장된 트랜잭션(size={}): {}", sz, tx);
		
		if (sz != null)
			couponRepository.decreaseStock(couponId, sz.intValue());
		
		redisTemplate.delete(key);
	}
}
```

```java
public interface CouponRepository extends JpaRepository<Coupon, Long> {
	@Modifying
	@Query("UPDATE Coupon c SET c.availableStock = c.availableStock - :count WHERE c.id = :couponId")
	void decreaseStock(Long couponId, int count);
}
```
정렬 집합에 저장된 tx들을 모두 조회하고, 원소 개수 만큼 availableStock 개수를 차감해준다.

만약 이 방법이 성공한다면 RDB에 전송되는 쿼리는 고작 한 개밖에 되지 않는다.
```java
@Test
@DisplayName("Sorted Set: 동시성 환경에서 400명 쿠폰 차감 테스트")
void `정렬_집합_쿠폰차감_동시성_400명_테스트`() throws InterruptedException {
	int threadCount = 400;
	ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
	CountDownLatch latch = new CountDownLatch(threadCount);
	
	for (int i = 0; i < threadCount; i++) {
		executorService.submit(() -> {
			try {
				couponDecreaseService.registerCouponRequest(coupon.getId(), coupon.getName());
			} finally {
				latch.countDown();
			}
		});
	}
	latch.await();
	
	couponTransactionSaveService.saveAll(coupon.getId(), coupon.getName());
	
	Coupon persistedCoupon = couponRepository.findById(coupon.getId())
			.orElseThrow(IllegalArgumentException::new);
	assertThat(persistedCoupon.getAvailableStock()).isZero();
	log.debug("잔여 쿠폰 수량: " + persistedCoupon.getAvailableStock());
}
```
![[Pasted image 20251101171634.png]]
상한선이 제대로 걸리고 있는지 확인해주기 위해서 thread 개수를 400개로 늘려보았다.
그리고 예상대로 테스트에 통과하는 것을 알 수있다.

DB에 빈번한 업데이트 연산이 가해지는 것을 막고, 접근성이 훨씬 빠른 Redis를 사용해 안전하고 빠르게 동시성 문제를 해결할 수 있었다.

여기서 한 가지 문제점은 동일한 사용자의 중복 요청에 대해서는 처리하지 않았다는 점이다.
만약 사용자가 연속으로 요청했고, 첫 번째 요청이 승인되었다고 가정하자. 문제는 그 다음 요청들로 인해 값이 업데이트되는데 순서가 중요한 작업인 경우 이러한 경우도 예외 처리를 해주어야 한다.
순서가 중요하지 않다? 그럼 ZSet을 사용할 이유가 없다. **멤버가 정렬될 필요가 없다면 ZSet보다 Set을 사용하는 것이 더 권장되기 때문**이다.

> [!info] 왜 ZSet보다 Set을 권장하는가
> 정렬된 배열이 정렬되지 않은 배열보다 연산자 속도가 빠르다는 것은 이미 자명하다.
> 
> 물론 데이터를 삽입하는 과정에서 계속 정렬을 수행해야 한다는 오버헤드가 있지만, Processor의 Branch Prediction의 효율을 높여주기 때문이다.

## Redis: Pipeline

![[Pasted image 20251101171854.png]]
Network 공부할 때 나오는 그 Pipeline 맞다.

- Redis가 TCP 기반의 서버라는 점을 이용해, TCP Pipeline 방식을 적용한 방법
- Request에 대한 Response를 기다리는 것이 아닌, 여러 Request를 한 번에 전송하고 비동기적으로 Response를 받는다.

물론 Pipeline은 네트워크 통신 성능을 향상시킬 수는 있어도 동시성 문제를 해결하진 않는다.

요청을 연속적으로 실행하긴 하지만, 그 사이에 다른 클라이언트의 명령이 끼어들지 않음을 보장하지는 않기 때문이다.

(그런데 이걸 왜 추가했냐면, 누가 Lua Script와 Pipeline을 비교해놓은 글을 봐서 이게 되나 싶어서 연구해봤다.)

따라서 Pipeline을 사용한 테스트는 별도로 작성하지 않았다.

## Messaging Queue

### Redis는 Silver Bullet인가?
다중화된 서버 환경에서 동시성 문제를 해결하기 위해 외부 저장소를 두기로 결정했고, Redis를 사용하자 대부분의 문제가 해결되는 것을 볼 수 있다. (심지어 고성능)

이런 마법같은 경험을 하고 나면 Redis를 만능처럼 사용하고 싶은 욕구가 들 수 있지만, 그렇게 되면 결국 Redis가 **SPOF**가 된다. (Redis 시스템이 중단되면, 전체 시스템이 마비)

이 문제는 Redis 의존성이 높아질 수록 심해지고, 그렇다고 Redis Clustering 환경을 구성하자니 엄청난 기술적 능력을 요구한다. (인프라 구성하는 돈은 땅파면 나오냐구...)

### Message Queue
Redis를 사용하지 않으면 메시지 큐처럼 Task를 관리하는 순차 처리 접근 방식을 떠올려볼 수도 있다.

생산자(Producer)가 Message Queue에 데이터를 삽입하면, 소비자(Consumer)는 데이터를 순차적으로 꺼내 처리하는 방식이다.
1. **요청 저장**: 각 요청은 도착하는 즉시 Message Queue에 추가된다. 고유 식별자 혹은 타임스탬프를 포함해 중복을 방지한다.
2. **순차적 처리**: 작업 Thead가 Queue에서 요청을 하나씩 꺼내서 처리한다. Queue의 순서에 따라 stock을 1씩 감소시킨다. 이 작업은 독립적으로 수행된다.
3. **결과 반환**:  처리 결과를 클라이언트에게 반환한다.

#### 테스트
실제 환경이라면 AWS SQS 혹은 Apache Kafka라도 사용해서 테스트 해봐야겠지만, 외부 Actor가 발생하면 테스트도 까다로워지고 비용도 드니까 java.util.concurrent 라이브러리 몇 가지를 사용해서 단일 서버 환경을 가정하여 구현해볼 생각이다.

물론, 다중화된 서버 환경에서 동시성을 보장해주진 않지만 단순 테스트 목적이니까 ㅎ

##### 1. ConcurrentLinkedQueue
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class MessageQueueCouponDecreaseService {
	private final CouponRepository couponRepository;
	private final Queue<String> couponQueue = new ConcurrentLinkedQueue<>();
	private final ExecutorService executorService = Executors.newSingleThreadExecutor(); // 워커 스레드
	private final AtomicInteger messageCount = new AtomicInteger(0);
	
	@PostConstruct public void init() {
		executorService.submit(this::processQueue);
	}
	
	public void decreaseStockWithMessagingQueue(Long couponId) {
		log.info("메시지 큐에 메시지를 추가합니다. couponId={}", couponId);
		couponQueue.offer(couponId + ":" + Thread.currentThread().getId());
		messageCount.incrementAndGet();
		log.info("메시지 큐에 메시지를 추가했습니다. couponId={}", couponId);
	}
	
	private void processQueue() {
		while (!Thread.currentThread().isInterrupted()) {
			String message = couponQueue.poll();
			log.info("메시지 큐에서 메시지를 가져옵니다. message={}", message);
			
			Long couponId = null;
			if (message != null)
				couponId = Long.parseLong(message.split(":")[0]);
			
			if (couponId != null) {
				try {
					Coupon coupon = couponRepository.findById(couponId)
							.orElseThrow(() -> new IllegalArgumentException("존재하지 않는 쿠폰입니다."));
					coupon.decreaseStock();
					couponRepository.save(coupon);
					messageCount.decrementAndGet();
					log.info("쿠폰 차감을 완료했습니다. couponId={}", couponId);
				} catch (Exception e) {
					log.error("쿠폰 차감 중 에러 발생", e);
					throw e;
				}
			}
		}
	}
	
	public void waitForCompletion() throws InterruptedException {
		while (messageCount.get() > 0) {
			Thread.sleep(100L);
		}
	}
}
```

```java
@Test
@DisplayName("Messaging Queue: 동시성 환경에서 300명 쿠폰 차감 테스트")
void 메시징_큐_쿠폰차감_동시성_300명_테스트() throws InterruptedException {
	int threadCount = 300;
	ExecutorService executorService = Executors.newFixedThreadPool(threadCount);
	CountDownLatch latch = new CountDownLatch(threadCount);
	
	for (int i = 0; i < threadCount; i++) {
		executorService.submit(() -> {
			try {
				messageQueueCouponDecreaseService
						.decreaseStockWithMessagingQueue(coupon.getId());
			} finally {
				latch.countDown();
			}
		});
	}
	latch.await();
	messageQueueCouponDecreaseService.waitForCompletion(); // 워커 스레드 작업 완료 대기
	Coupon persistedCoupon = couponRepository.findById(coupon.getId())
			.orElseThrow(IllegalArgumentException::new);
	assertThat(persistedCoupon.getAvailableStock()).isZero();
	log.debug("잔여 쿠폰 수량: " + persistedCoupon.getAvailableStock());
}
```
![[Pasted image 20251101172437.png]]로그 안 지우면, 디버깅용 로그 때문에 테스트 로그를 확인할 수 없으므로 주의

![[Pasted image 20251101172448.png]]
- ConcurrentLinkedQueue는 **내부적으로 CAS 알고리즘을 사용하여 삽입/삭제 연산을 수행하므로 동시성이 보장**된다.
- 클래스가 생성되면, worker thread는 **바쁜 대기 상태로 Queue의 상태를 계속해서 관찰**한다.
- 위 작업은 **비동기 작업으로 수행되므로, 테스트 케이스에선 워커 스레드의 작업이 끝날 때까지 기다리는 로직이 필요**하다.

위 방법을 사용하면, 요청들을 순차적으로 처리하기 때문에 경쟁 상태(Race Condition) 없이 작업을 처리할 수 있다. 또한 요청이 도착하자마자 Queue에 넣고 관리하므로 모니터링, 로깅, 재시도 메커니즘을 구현하기도 용이하다.

하지만 워커 스레드 하나가 CPU를 계속 점유하게 되어, 권장하고 싶은 방법은 아니다.

##### 2. LinkedBlockingQueue
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class MessageQueueCouponDecreaseService {
	private final CouponRepository couponRepository;
	private final BlockingQueue<String> couponQueue = new LinkedBlockingQueue<>();
	private final ExecutorService executorService = Executors.newSingleThreadExecutor(); // 워커 스레드
	private final AtomicInteger messageCount = new AtomicInteger(300);
	
	@PostConstruct
	public void init() {
		executorService.submit(this::processQueue);
	}
	
	public void decreaseStockWithMessagingQueue(Long couponId) {
		couponQueue.offer(couponId + ":" + Thread.currentThread().getId());
	}
	
	private void processQueue() {
		while (!Thread.currentThread().isInterrupted()) {
			try {
				String message = couponQueue.take();
				log.info("메시지를 처리합니다. message={}", message);
				
				Long couponId = Long.parseLong(message.split(":")[0]);
				
				Coupon coupon = couponRepository.findById(couponId)
						.orElseThrow(() -> new IllegalArgumentException("존재하지 않는 쿠폰입니다."));
				coupon.decreaseStock();
				couponRepository.save(coupon);
				
				messageCount.decrementAndGet();
			} catch (InterruptedException e) {
				Thread.currentThread().interrupt();
			} catch (Exception e) {
				log.error("메시지 처리 중 에러 발생", e);
			}
		}
	}
	
	public void waitForCompletion() throws InterruptedException {
		while (messageCount.get() > 0) {
			Thread.sleep(100L);
		}
	}
	
	@PreDestroy
	public void close() {
		executorService.shutdownNow();
	}
}
```
![[Pasted image 20251101172943.png]]
- BlockingQueue의 **`take()` 메서드는 큐에 아무런 원소가 없을 때, worker thread가 대기 상태로 전환**된다.
- **바쁜 대기를 없애긴 했지만, Context Switching이라는 오버헤드가 발생**한다.
- 여전히 비동기 처리되는 작업을 기다려주기 위한 `waitForCompletion()`을 필요로 하며, 테스트를 어렵게 만든다.

## Performance

![[Pasted image 20251101173126.png]]
1. synchronized(Non Tx)
    - 소요 시간이 전체적으로 높으며, 스레드 수가 증가하고 재고가 많아질 수록 급격하게 증가한다.
    - 단일 서버 환경의 동시성 제어만 지원한다.
2. synchronized(외부)
    - 트랜잭션이 적용되지 않은 (1)에 비교하여 상대적으로 낮은 소요 시간을 보인다.
    - 단일 서버 환경의 동시성만 제어하는 방법 중 가장 좋은 성능을 보여준다.
3. Reentrant Lock
    - 트래픽이 적을 땐, 단일 서버 환경의 동시성만 제어하는 방법 중 가장 좋은 성능을 보여준다.
    - 트래픽이 많아질 수록 성능이 급격하게 저하된다.
4. Optimistic Lock
    - 애초에 예능 로직이나 다름없어서 평가하는 의미가...
    - 적어도 충돌이 많은 환경에서 낙관적 락을 통해 반드시 쓰기 작업을 완수하려는 시도는 하지 않는 게 정신 건강에 이로움을 알 수 있다.
    - 분산 서버 환경의 동시성 제어를 지원한다.
5. Pessimistic Lock
    - 분산 서버 환경의 동시성을 제어하는 방법 중, 가장 구현이 쉽고 성능 또한 준수한 편에 속한다.
    - 하지만 테이블에 배타적 락을 걸기 때문에 별개 기능들의 성능 저하가 뒤따라오며, 자칫 Dead Lock에 걸리기 쉽다.
6. Distributed Lock
    - DB에 부하를 줄이고, 분산 서버 환경의 동시성을 제어할 수 있다.
    - 하지만 트래픽이 증가할 수록, 소요 시간도 선형적으로 증가하는 양상을 보인다.
7. Redis: Sorted Set
    - 매우 낮은 소요 시간을 보이며, 스레드 수와 재고가 증가해도 성능 저하가 매우 적다.
    - 분산 서버 환경의 동시성을 제어할 수 있으며, Redis의 높은 성능과 효율적인 데이터 구조를 잘 사용하고 있다.
    - 하지만 Redis 의존적이며, Redis가 SPOF로 작용할 수 있다.
8. Messaging Queue
    - 이것도 예능 코드에 가까워서 평가의 의미가 없다.

### 정렬 집합(Sorted Set)의 무적인가?
애석하게도 정렬 집합으로 효과적인 성능 개선 사례를 볼 수 있는 것은 한정적이다.
- 순위 시스템
- 시간 기반 이벤트 정렬
- 리더보드
- 작업 우선순위 큐
- 재고 관리
- API 처리율 제한 장치

Lua Script를 사용하는 만큼 Unit Test를 어렵게 만들고, 로직이 복잡해질 수록 병목 현상이 심해져 성능 저하를 유발할 수 있다.

만약 여러 사용자가 같은 아이디로 동시에 회원가입을 시도하려 할 때는 그렇게 많은 트래픽이 몰리진 않을 것이며, 차라리 분산락을 사용하는 것이 훨씬 간편할 것이다.

데이터 일관성이 매우 중요하고, Transaction을 한 번에 저장해놓았다가 한 번에 insert를 하는 은행 시스템이면 비관적 락을 사용하는 것이 안전할 수도 있다.

가장 빠른 성능을 낸다고 해서, 반드시 정렬 집합을 고집할 이유는 없다.