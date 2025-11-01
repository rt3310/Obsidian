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

