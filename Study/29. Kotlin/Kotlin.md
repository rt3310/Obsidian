## 주요 특징

### 명시적인 언어
- `int`를 `long`으로 암시적 변환해주지 않는다.
	- `Int.toLong()`을 명시적으로 호출해야 한다.

### 정적 바인딩 선호
- kotlin은 type-safe한, compositive한 코딩 스타일을 장려한다.
- 확장함수는 정적으로 바인딩된다. 기본적으로 클래스는 확장될 수 없고, 메서드는 다형적이지 않다.
- 명시적으로 다형성과 상속을 활성화해야 한다.
- 리플렉션을 사용하고 싶으면 플랫폼 별로 다른 리플렉션 라이브러리를 의존 관계에 추가해야만 한다.

### 식이 본문인 함수의 반환 타입만 생략 가능

### Visibility Modifier
- default: `public`

### Package
- 여러 클래스를 한 파일에서 관리 가능
- 파일 이름을 클래스 명이랑 맞출 필요 없음
- 하지만 대부분의 경우 자바와 같이 패키지를 디렉터리와 맞춰 구현하는 것을 추천

### enum
- enum은 soft keyword
	- `class` 앞에 있을 때만 의미를 지님
	- `enum class Color { RED, ORANGE }`

```kotlin
enum class Color(
	val r: Int, val g: Int, val b: Int
) {
	RED(255, 0, 0),
	ORANGE(255, 165, 0); // 유일한 세미콜론
	
	fun rgb() = (r * 256 * g) * 256 + b
}
```

### when
- break가 필요하지 않음 (Java 14의 개선된 switch 문과 비슷)
- when도 식이다
```kotlin
when (color) {
	Color.RED -> "RED"
	Color.ORANGE -> "ORANGE
	Color.BLUE, Color.INDIGO -> "Others"
}
```

- 임의의 객체 사용 가능
- 비교할 때는 동등성(Equality) 사용
```kotlin
fun mix(c1: Color, c2: Color) =
	when (setOf(c1, c2)) {
		setOf(RED, YELLOW) -> ORANGE
		else -> throw Exception("No Such Element")
	}
```

- 인자 없이 사용 가능
	- 자주 호출된다면, 불필요한 객체 생성을 막을 수 있음
```kotlin
fun mix(c1: Color, c2: Color) =
	when {
		(c1 == RED && c2 == YELLOW) ||
		(c1 == YELLOW && c2 == RED) ->
			ORANGE
		else -> throw Exception("No Such Element")
	}
```

### smart cast(is)
- Java 16의 Instance of 패턴 매칭과 비슷
- is로 타입을 검사 한 후 컴파일러가 자동 캐스팅 수행
```kotlin
fun getType(c: Car) =
	when (c) {
		is VAN -> c.value // 스마트 캐스트 사용 됨
		is SUV -> c.value
		else -> throw Exception("ex")
	}
```

### if
- kotlin에는 3항 연산자가 없다
- if도 식이기 때문에 3항 연산자처럼 쓸 수 있다.
```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

### for
- `a..b`: a<= i <= b
```kotlin
for (i in 1..10) {
}
```

- `a until b`: a <= i < b
```kotlin
for (i in 1 until 10) {
}
```

- 역방향
```kotlin
for (i in 100 downTo 1 step 2) { // 100부터 1까지 2씩 내려감 (1값도 포함)
}
```
#### map iteration
```kotlin
var binaryReps = TreeMap<Char, String>()

for (c in 'A'..'F') {
	val binary = Integer.toBinaryString(c.toInt())
	binaryReps[c] = binary
}

for ((letter, binary) in binaryReps) {
	println("$letter = $binary")
}
```
#### list iteration
```kotlin
val list = arrayListOf("10", "11", "1001")
for ((index, element) in list.withIndex()) {
	println("$index: $element")
}
```

### in
- `in`을 통해 값이 범위에 속하는지 검사 가능
- `!in`을 통해 값이 범위에 속하지 않는지 검사 가능
```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

```kotlin
"Kotlin" in "Java".."Scala" // true
```

```kotlin
"Kotlin" in setOf("Java", "Scala") // false
```

### 예외 처리
- throw도 식이다.
- 조건이 거짓이면 변수가 초기화되지 않는다.
```kotlin
val percentage = 
	if (number in 0..100)
		number
	else
		throw IllegalArgumentException("ex")
```

#### try..catch..finally
- Java와 달리 `throws`를 명시하지 않는다.
- Checked Exception과 Unchecked Exception을 구별하지 않는다.
```kotlin
fun readNumber(reader: BufferedReader): Int? {
	try {
		val line = reader.readLine()
		return Integer.parseInt(line)
	} catch (e: NumberFormatException) {
		return null
	} finally {
		reader.close()
	}
}
```

### Collection
- Java와 동일한 Collection (상호작용 가능)
```kotlin
val set = hashSetOf(1, 7, 25)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven") // 여기서 to는 키워드가 아니라 일반함수
```

- 대신 Java보다 더 많은 기능 지원
```kotlin
val strings = listOf("first", "second")
val last = strings.last() // second
val numbers = setOf(1, 14, 2)
val max = numbers.max() // 14
```

### 이름 붙인 인자(named argument)
```kotlin
fun <T> joinToString(
	collection: Collection<T>,
	separator: String,
	prefix: String,
	postfix: String	
): String {}
```

```java
joinToString(collection, /* separator */ " ", /* prefix */ " ", /* postfix */ ".");
```

- 호출 시 인자 중 어느 하나라도 이름을 명시하고 나면 그 뒤에 오는 모든 인자는 이름을 꼭 명시해야 한다.
```kotlin
joinToString(collection, separator = " ", prefix = " ", postfix = ".")
```

### 디폴트 파라미터 값

- default 값이 지정된 파라미터가 default 값이 없는 파라미터보다 뒤에 위치해야 한다.
```kotlin
fun <T> joinToString(
	collection: Collection<T>,
	separator: String = ", ",
	prefix: String = "",
	postfix: String = ""
): String
```

- 오버로딩 메서드가 많아지는 문제를 피할 수 있다.
**Java의 Thread constructors**
![[Pasted image 20260226171247.png]]

- named argument를 조합하면 더욱 유연한 호출이 가능하다.
- named argument는 함수 호출 시 파라미터의 순서를 무시하고 원하는 파라미터에 값을 할당 할 수 있기 때문이다.
```kotlin
fun displayInfo(name: String = "Unknown", age: Int = 0) {
	println("Name: $name, Age: $age")
}

displayInfo(age = 25) // Name: Unknown, Age: 25
```

### 최상위 함수
- Kotlin에서는 소스 파일의 최상위 수준에서 함수를 선언할 수 있다.
- JVM이 컴파일할 때 새로운 클래스를 정의해준다. (이름은 코틀린 소스파일의 이름과 대응한다.)
	- 만약 이름을 바꾸고싶다면 `@JvmName`을 추가한다.
	- `@JvmName`은 파일의 맨 앞, 패키지 이름 선언 이전에 위치해야 한다.

```kotlin
@file:JvmName("StringFunctions")

package strings

fun joinToString(...): String { ... }
```

### 최상위 프로퍼티

- 최상위 프로퍼티도 함수와 마찬가지로 JVM이 새로운 클래스를 정의하여 선언한다(이름도 동일하다)
- `val`의 경우 getter, `var`의 경우 getter와 setter가 생긴다.
```kotlin
var opCount = 0 // 최상위 프로퍼티

fun performOperation() {
	opCount++
}

fun reportOperationCount() {
	println("Operation performed $opCount times")
}
```

```kotlin
val UNIX_LINE_SEPARATOR = "\n"
```
- 이런 프로퍼티 값은 정적 필드에 저장된다.
- 최상위 프로퍼티를 활용해서 코드에 상수를 추가할 수 있다.

- 겉으로 상수처럼 보이는데 실제로 getter를 사용해야한다면 자연스럽지 못하다.
	- 더 자연스럽게 사용하려면 public static final 필드로 컴파일 해야 한다.
	- `const`를 추가하면 프로퍼티를 `public static final` 필드로 컴파일하게 만들 수 있다.
```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
```

```java
public static final String UNIX_LINE_SEPARATOR = "\n"
```

| Kotlin            | Java                              | Java에서 접근       |
| ----------------- | --------------------------------- | --------------- |
| `val x`           | `private field` + getter          | `MainKt.getX()` |
| `var x`           | `private field` + getter + setter | `MainKt.setX()` |
| `const val x`     | `public static final field`       | `MainKt.x`      |
| `@JvmField var x` | `public field`                    | `MainKt.x`      |
### 확장 함수
```kotlin
package strings

fun String.lastChar(): Char = this.get(this.length - 1)
```

```kotlin
println("Kotlin".lastChar()) // n
```
- `this`를 생략할 수 있다.
- 확장 함수 내부에서는 일반적인 인스턴스 메서드의 내부에서와 마찬가지로 수신 객체의 메서드나 프로퍼티를 바로 사용할 수 있다.

- 확장 함수가 캡슐화를 깨지는 않는다.
	- 클래스 안에서 정의한 메서드와 달리 확장 함수 안에서는 클래스 내부에서만 사용할 수 있는 private 멤버나 protected 멤버를 사용할 수 없다.

- as 키워드를 사용하면 임포트한 클래스나 함수를 다른 이름으로 부를 수 있다.
```kotlin
import strings.lastChar as last

var c = "Kotlin".last()
```

#### 확장함수는 오버라이드 할 수 없다.

- 코틀린은 호출될 확장 함수를 정적으로 결정하기 때문에 오버라이드 할 수 없다.
```kotlin
open class View {}
class Button: View {}

fun View.showOff() = println("view")
fun button.showOff() = println("button")

val view: View = Button() // 확장함수는 정적으로 결정된다.
view.showOff() // view
```

### 확장 프로퍼티
- 확장 프로퍼티는 상태를 저장할 적절한 방법이 없기 때문에 실제로 아무 상태도 가질 수 없다.
- 하지만 프로퍼티 문법으로 더 짧게 코드를 작성할 수 있다.
- 뒷바침하는 필드가 없어서 기본 getter 구현을 제공할 수 없으므로 최소한 getter는 정의해야 한다.
```kotlin
val String.lastChar: Char
	get() = get(length - 1)
```

```kotlin
var StringBuilder.lastChar: Char
	get() = get(length - 1)
	set(value: Char) {
		this.setCharAt(length - 1, value)
	}
```