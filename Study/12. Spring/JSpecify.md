## JSpecify

자바 생태계에서 NPE는 자주 마주치는 문제이다.

많은 개발자들이 이를 미연에 방지하고자 Null-safety를 강화하는 도구나 annotation을 활용했지만 표준화된 방식이 존재하지 않았다.

이러한 문제를 해결하기 위해 나온 것이 JSpecify이다.

JSpecify는 Java 생태계에서 Null 가능성을 일관적이고 표준화 된 방식으로 표시하고 검증하기 위해 탄생한 오픈소스이다.
쉽게 말해 자바 코드에서 '이 변수(또는 파라미터)는 null이 될 수 있다/없다'를 명확하게 표기하고 이를 도구(IDE, 정적 분석기 등)가 공통적으로 이해하고 체크할 수 있도록 표준 규악을 제시한다.

## JSpecify가 왜 필요한가?

### 1. Java의 NPE에 대한 빈번함
Java에서 가장 흔한 런타임 에러 중 하나가 NPE이다.
이를 방지하고자 개발자들은 IDE의 null 체크 기능 또는 `@Nullable`, `@NotNull` 같은 annotation을 사용해왔다.

### 2. 파편화 된 null annotation
Java 생태계에는 많은 null 관련 annotation들이 존재한다.
목적은 모두 'Null 가능성을 명시하고 런타임 에러를 줄이자'이지만 저마다 다른 패키지나 룰을 가지고 있다.

도구마다 호환성이 조금씩 다르기 때문에 문제가 발생할 수 있다. 이 문제를 해결하기 위한게 JSpecify다.

## JSpecify가 제안하는 해결책

### 1. 표준화 된 null annotation
JSpecify는 크게 `@Nullable`, `@NotNull`, `@NullMarked`, `@NullUnmarked`라는 네 가지 키워드로 널 가능성을 표현한다.

- `@Nullable`: 해당 타입이 null 일 수 있음을 명시
	- ex) `@Nullable String`
- `@NonNull`: 해당 타입이 절대 null이 아님을 명시
	- ex) `@NotNull String`
- `@NullMarked`: 해당 범위(모듈, 패키지, 클래스, 메서드)에 선언된 타입들은 기본적으로 널이 아닐 것으로 간주
- `@NullUnmarked`: 이미 `@NullMarked`가 선언된 범위 안에서 다시 'null에 대한 지정 없음' 상태로 되돌리기


### 2. IDE, 정적 분석 도구와 연동
이미 IntelliJ, Checker Framework, NullAway 등 다양한 도구들이 JSpecify를 지원하고 있다.

JSpecify를 사용하면 이 도구들이 '해당 변수, 메서드가 null일 수 있는지 여부'를 더 명확히 판단해 줄 수 있어, 개발 단계에서 NPE를 더 빠르고 정확히 파악할 수 있다.

## 예시

### @NullMarked 클래스 예시
```java
import org.jspecify.annotations.Nullable;
import org.jspecify.annotations.NullMarked;

@NullMarked class Strings {
	
	// @NullMarked로 선언된 클래스 내부에서는
	// 별도의 Nullable/NonNull 표기가 없으면 모두 NonNull로 간주된다.
	
	// 반환 타입에 @Nullable 명시 → null을 반환할 수 있음
	static @Nullable String emptyToNull(String x) {
		return x.isEmpty() ? null : x;
	}
	
	// 매개변수에 @Nullable 명시 → null을 파라미터로 받을 수 있음
	static String nullToEmpty(@Nullable String x) {
		return x == null ? "" : x;
	}
	
	void doSomething() {
		// nullToEmpty(null)는 허용: null 인자를 받을 수 있는 @Nullable 파라미터
		int length1 = nullToEmpty(null).length();
		System.out.println("length1: " + length1);
		
		// emptyToNull("")는 빈 문자열이면 null을 반환 → 반환값은 @Nullable
		// 바로 .length() 호출 시 정적 분석기나 IDE가 NPE 가능성 경고를 줄 수 있음
		int length2 = emptyToNull("").length(); // 잠재적 NPE
		System.out.println("length2: " + length2);
	}
}
```

### 로컬 변수에 대한 처리 예시
```java
import org.jspecify.annotations.Nullable;
import org.jspecify.annotations.NullMarked;

@NullMarked
class LocalVariableExample {
	void handleStrings(@Nullable String maybeNull, String definitelyNonNull) {
		// @NullMarked 범위라서 'String definitelyNonNull'는 null이 될 수 없다고 간주
		// @Nullable String maybeNull 은 null 가능성이 존재
		
		String fromNullable = maybeNull; // IDE/정적 분석기: fromNullable은 잠재적 null
		String fromNonNull = definitelyNonNull; // NonNull
		
		// 로컬 변수 선언 시, root type에는 @Nullable/@NonNull을 직접 붙이지 않는 것이 권장됨
		// (분석기가 대입되는 값을 보고 추론하기 때문)
		String either = randomCondition() ? maybeNull : definitelyNonNull;
		// either는 null 가능성이 있다고 분석될 수 있음
		
		if (either != null) {
			// 여기선 either가 null이 아님이 보장된 상태
			System.out.println(either.toUpperCase());
		}
	}
	
	private boolean randomCondition() {
		return Math.random() > 0.5;
	}
}
```
- JSpecify 공식 문서에 따르면 **로컬 변수(root type)** 에 바로 `@Nullable`을 붙이는 대신 파라미터/필드에서 널 가능성을 명시해주고 **정적 분석기가 어떤 값을 대입받는지 추론하게** 하는 방식을 권장한다.

### Generic 활용 예시
```java
import org.jspecify.annotations.Nullable;
import org.jspecify.annotations.NullMarked;
import java.util.ArrayList;
import java.util.List;

@NullMarked
public class Methods {

	// <T> 메서드에서 @Nullable T를 반환할 수 있음
	public static <T> @Nullable T firstOrNull(List<T> list) {
		return list.isEmpty() ? null : list.get(0);
	}
	
	// <T extends @Nullable Object>로 선언하면,
	// T가 @Nullable String 등 널이 가능한 타입으로도 대체될 수 있음
	public static <T extends @Nullable Object> T firstOrDefault(List<T> list, T defaultValue) {
		return list.isEmpty() ? defaultValue : list.get(0);
	}
	
	public static void exampleUsage() {
		// 1) List<String> → firstOrNull은 @Nullable String 반환
		List<String> nonNullList = List.of("A", "B");
		@Nullable String first = firstOrNull(nonNullList); // possibly null
		System.out.println("first: " + first);
		
		// 2) List<@Nullable String>도 가능
		List<@Nullable String> nullableList = new ArrayList<>();
		nullableList.add(null);
		nullableList.add("Hello");
		// firstOrDefault에서 T는 @Nullable String이 될 수 있음
		@Nullable String result = firstOrDefault(nullableList, null);
		// result는 null일 수도 있고, "Hello"일 수도 있음
		System.out.println("result: " + result);
	}
}
```
- `<T extends @Nullable Object>`: 제네릭 타입 변수 T가 null을 포함하는 타입도 허용
- (또는 )로 선언된 경우에는 기본적으로 NonNull로 취급되므로 `List<@Nullable String>` 같은 형식은 허용안 될 수 있다.
### @NullUnmarked 사용 예시
```java
import org.jspecify.annotations.NullMarked;
import org.jspecify.annotations.NullUnmarked;

@NullMarked
public class MixedScopes {

	// 이 클래스 전체는 NullMarked로 처리
	// → 별도 표기가 없는 모든 참조 타입은 NonNull
	public String hello(String name) {
		// 여기서 'String name'은 null이 아니라고 가정
		return "Hello, " + name;
	}
	
	@NullUnmarked public String fetchValue(String key) {
		// NullUnmarked 스코프에선 'String key'가 널인지 아닌지 지정되지 않음(unspecified)
		// 정적 분석기가 완벽히 추론하기 어려울 수 있음
		if (key == null) {
			return "Got a null key!";
		}
		return "Value for: " + key;
	}
}
```
- `@NullUnmarked`를 메서드 단위로 적용해 상위 스코프(`@NullMarked`)의 규칙을 무효화할 수 있다.
- 큰 범위(ex: 패키지 전체)에서 `@NullMarked`를 선언해도 특정 레거시 코드만 `@NullUnmarked`로 지정하여 점진적으로 migration할 수 있다.

### Type-use Annotation Syntax
JSpecify가 제시하는 null annotation(`@Nullable`, `@NonNull`)은 type-use 위치에서 적용할 수 있다.

이는 Java 8부터 도입된 `Type Annotations` 개념을 활용하는 것으로, 어떤 타입에 `@Nullable`을 붙이느냐에 따라 의미가 달라질 수 있다.

1. `@Nullable String[]`
	- 배열 요소(String)가 null일 수 있다는 의미
	- 배열 객체 자체는 `@NullMarked`하에서 NonNull로 간주됨
2. `String @Nullable []`
	- 배열 객체 자체가 null일 수 있다는 의미
	- 배열 안의 String 요소는 NonNull
3. `@Nullable String @Nullable []`
	- 배열 객체도 null 가능 + 배열 안의 요소도 null 가능
	- 가장 넓은 범위로 null을 허용