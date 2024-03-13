String 객체에 대한 해시 함수 수행 시간은 문자열 길이에 비례한다.

때문에 JDK 1.1에서는 String 객체에 대해서 빠르게 해시 함수를 수행하기 위해, 일정 간격의 문자에 대한 해시를 누적한 값을 문자열에 대한 해시 함수로 사용했다.
```java
public int hashCode() {
	int hash = 0;
	int skip = Math.max(1, length() / 8);
	for (int i = 0; i < length(); i += skip) {
		hash = s[i] + (37 * hash);
	}
	return hash;
}
```
위 예제에서 볼 수 있듯이 모든 문자에 대한 해시 함수를 계산하는 게 아니라, 문자열의 길이가 16을 넘으면 최소 하나의 문자를 건너가며 해시 함수를 계산했다.

그러나 이런 방식은 심각한 문제를 야기했다. 웹상의 **URL은 길이가 수십 글자에 이르면서 앞 부분은 동일하게 구성되는 경우가 많다**. 이 경우 **서로 다른 URL의 해시 값이 같아지는 빈도가 매우 높아질 수 있다는 문제**가 있다. 따라서 이런 방식은 곧 폐기되었고, 다음 예제에서 보는 방식을 현재의 Java 8까지도 계속 사용하고 있다.
```java
public int hashCode() {
	int h = hash;
	if (h == 0 && value.length > 0) {
		char val[] = value;

		for (int i = 0; i < value.length; i++) {
			h = 31 * h + val[i];
		}
		hash = h;
	}
	return h;
}
```
위 예제는 Horner's method를 구현한 것이다. Horner's method는 다항식을 계산하기 쉽도록 단항식으로 이루어진 식으로 표현하는 것이다. 즉 위 예제에서 계산하고자 하는 해시 값 h는 다음과 같다.

![hashmap10](https://d2.naver.com/content/images/2015/06/helloworld-831311-10.png)

![hashmap11](https://d2.naver.com/content/images/2015/06/helloworld-831311-11.png)

![hashmap12](https://d2.naver.com/content/images/2015/06/helloworld-831311-12.png)

이렇게 단항식을 재귀적으로 사용하여 다항식 연산을 표현할 수 있다.

String 객체 해시 함수에서 31을 사용하는 이유는, **31이 소수이며 또한 어떤 수에 31을 곱하는 것은 빠르게 계산할 수 있기 때문**이다. 31N=32N-N인데, 32는 25이니 어떤 수에 대한 32를 곱한 값은 shift 연산으로 쉽게 구현할 수 있다. 따라서 N에 31을 곱한 값은, (N << 5) – N과 같다. **31을 곱하는 연산은 이렇게 최적화된 머신 코드로 생성할 수 있기 때문에, String 클래스에서 해시 값을 계산할 때에는 31을 승수로 사용**한다.