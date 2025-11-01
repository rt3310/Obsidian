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

