JDK 7u6부터 JDK 7u25까지는 HashMap에 저장된 키-값 쌍이 일정 개수 이상이면 String 객체에 한하여 별도의 해시 함수를 사용할 수 있게 하는 기능이 있다. 이 기능은 JDK 7u40부터는 삭제되었고, 당연히 Java 8에도 해당 기능은 없다. 여기서 말하는 '일정 개수 이상'이나 '별도의 해시 함수 사용 여부 지정'은 JVM을 가동할 때 옵션으로 지정할 수 있다.

```java
hashSeed = useAltHashing ? sun.misc.Hashing.randomHashSeed(this) : 0;
...
int h = hashSeed;
if (0 != h && k instanceof String) {
	return sun.misc.Hashing.stringHash32((String) k);
}
...
int hash32() {
	int h = hash32;
	if (0 == h) {
		h = sun.misc.Hashing.murmur3_32(HASHING_SEED, value, 0, value.length);
		h = (0 != h) ? h : 1;
		hash32 = h;
	}
	return h;
}
```
JDK 7u6부터 JDK 7u25까지는 jdk.map.althashing.threshold 옵션을 지정하면, HashMap에 저장된 키-값 쌍이 일정 개수 이상일 때 String 객체에 String 클래스의 hashCode() 메서드 대신 `sun.misc.Hashing.stringHash32()` 메서드를 사용할 수 있게 했다.
`sun.misc.Hashing.stringHash32()` 메서드는 String 클래스의 `hash32()` 메서드를 호출하게 한 것이고, `hash32()` 메서드는 MurMur 해시를 구현한 것이다.
이 MurMur 해시를 이용하여 String 객체에 대한 해시 충돌을 매우 낮출 수 있었다고 한다.

그러나 부작용도 있다. MurMur 해시는 hash seed를 필요로 하는데, 이를 위한 것이 `sun.misc.Hashing.randomHashSeed()` 메서드다.
이 메서드에서는` Random.nextInt()` 메서드를 사용한다. `Random.nextInt()` 메서드는 compare and swap 연산(이하 CAS 연산)을 사용하는 AtomicLong을 사용하는데, **CAS 연산은 코어가 많을수록 성능이 떨어진다**. 즉 **JDK 7u6부터 등장한 String 객체에 대한 별도의 해시 함수는 멀티 코어 환경에서는 성능이 하락**했고, 이런 문제로 인해 JDK 7u40부터는 해당 기능을 사용하지 않는다. 당연히 Java 8도 사용하지 않는다.