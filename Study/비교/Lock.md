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