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