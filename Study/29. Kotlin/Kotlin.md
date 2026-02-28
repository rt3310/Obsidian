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
- private, protected, public, internal

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
	ORANGE(255, 165, 0)\; // 유일한 세미콜론
	
	fun rgb() = (r * 256 * g) * 256 + b
}
```

### when
- break가 필요하지 않음 (Java 14의 개선된 switch 문과 비슷)
- when도 식이다
```kotlin
when (color) {
	Color.RED -> "RED"
	Color.ORANGE -> "ORANGE"
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

### 가변 길이 인자
- Java의 `...`과 비슷하다.
```kotlin
fun listOf<T>(vararg values: T): List<T> { ... }
```

- Kotlin에서는 spread 연산자(\*)를 통해 각 원소가 인자로 전달되게 할 수 있다.
```kotlin
fun main(args: Array<String>) {
	val list = listOf("args: ", *args) // spread 연산자가 배열의 내용을 펼쳐준다.
	println(list)
}
```

### 중위 호출
- 이전에 `to`는 키워드가 아니라 일반 메서드라고 했다.
- 이 코드는 중위 호출(infix call)이라는 방식으로 `to`라는 일반 메서드를 호출한 것이다.
```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")\
```

- 다음 두 호출은 동일하다.
```kotlin
1.to("one")
1 to "one"
```
- 인자가 하나뿐인 일반 메서드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다.
- 함수를 중위 호출에 사용하게 허용하고 싶으면 infix 변경자를 함수 선언 앞에 추가해야 한다.
```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```
- 이 `to` 함수는 `Pair`의 인스턴스를 반환한다.
- `Pair`는 Kotlin 표준 라이브러리 클래스이다.

### 구조 분해 선언
- Pair의 내용으로 두 변수를 즉시 초기화 할 수 있다.
- Javascript와 Python에 있는 것과 비슷하다.
```kotlin
val (number, name) = 1 to "one"
```

- 루프에서도 사용 가능하다.
```kotlin
for ((index, element) in collection.withIndex()) {
	println("$index: #element")
}
```

### 문자열, 정규식
```kotlin
println("12.345-6.A".split("\\.|-".toRegex()))
```

```kotlin
println("12.345-6.A".split(".", "-"))
```

```kotlin
fun parsePath(path: String) {
	val directory = path.substringBeforeLast("/")
	val fullName = path.substringAfterLast("/")
	
	val fileName = fullName.substringBeforeLast(".")
	val extension = fullName.substringAfterLast(".")
	println("Dir: $directory, name: $fileName, ext: $extension")
}

parsePath("/Users/yole/kotlin/chapter.adoc") // Dir: /Users/yole/kotlin, name: chapter, ext: adoc
```

```kotlin
fun parsePath(path: String) {
	val regex = """(.+)/(.+)\.(.+)""".toRegex()
	val matchResult = regex.matchEntire(path)
	if (matchResult != null) {
		val (directory, filename, extension) = matchResult.destructured
		println("Dir: $directory, name: $fileName, ext: $extension")
	}
}
```

### 여러 줄 문자열
- Java에 있는 것과 비슷하다.
- `"""` 3중 따옴표를 통해 작성한다.
- 3중 따옴표를 사용하면 문자열 이스케이프 없이 문자를 그대로 작성 할 수 있다.
- 3중 따옴표 안에 문자열 템플릿(`$`)는 넣을 수 있는데 만약 $ 문자 그대로를 넣어야 한다면
	- `"""${'$'}99.9"""`처럼 문자열 템플릿 안에 '$' 문자를 넣어야 한다.
- 여러 줄 문자열은 테스트 코드에서 예상 출력을 작성할 때 특히 유용하다(가장 완벽한 해법). (코드 예시는 따로)
```kotlin
val kotlinLogo = """     | //
				.| //
				.|/ \"""
```

### 로컬 함수
- Kotlin에서는 로컬 함수를 통해 코드 중복을 개선할 수 있다.
```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
	if (user.name.isEmpty()) {
		throw IllegalArgumentException("empty Name")
	}
	
	if (user.address.isEmpty()) {
		throw IllegalArgumentException("empty Address")
	}
}
```

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
	fun validate(value: String, fieldName: String) {
		if (value.isEmpty()) {
			throw IllegalArgumentException("empty $fieldName")
		}
	} 
	
	validate(user.name, "Name")
	validate(user.address, "Address")
}
```
- 로컬 함수에서 바깥 함수의 파라미터와 변수에도 직접 접근할 수 있다.
- 이를 개선해서 검증 로직을 User 클래스의 확장 함수로 만들 수도 있다.

### 인터페이스
- Kotlin 인터페이스 안에는 추상 메서드뿐 아니라 구현이 있는 메서드도 정의할 수 있다(Java의 default method와 비슷하다).
- 당연히 인터페이스에는 아무런 상태도 들어갈 수 없다.
```kotlin
interface Clickable {
	fun click()
}
```

- Java의 `@Override`와 달리 Kotlin에서 `override` 키워드는 꼭 명시해야 한다.
```kotlin
class Button : Clickable {
	override fun click() = println("I was clicked")
}
```

- 만약 한 클래스에서 default method가 있는 두 인터페이스를 함께 구현한다면 컴파일 에러가 발생할 수 있다.
	- 구현을 대체할 오버라이딩 메서드를 직접 제공해야 한다.
```kotlin
interface Clickable {
	fun click()
	fun showOff() = println("I'm clickable")
}
```

```kotlin
interface Focusable {
	fun setFocus(b: Boolean) = println("I ${if (b) "got" else "lost"} focus.")
	
	fun showOff() = println("I'm focusable")
}
```

```kotlin
class Button: Clickable, Focusable {
	override fun click() = println("I was clicked")
	override fun showOff() {
		super<Clickable>.showOff()
		super<Focusable>.showOff()
	}
}
```
- 상위 타입의 구현을 호출할 때는 Java와 똑같이 `super`를 사용한다. 하지만 Type을 지정하는 부분은 다르다.
	- Java: `Clickable.super.showOff()`
	- Kotlin: `super<Clickable>.showOff()`

### class (default final)
- fragile base class 문제는 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로써 깨져버린 경우에 생긴다.
- Effective Java에서는 "상속을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 상송을 금지하라" 라는 조언을 한다. 이는 특별히 하위 클래스에서 오버라이드하게 의도된 클래스와 메서드가 아니라면 모두 `final`로 만들라는 뜻이다.
- Kotlin도 이와 같은 철학을 따른다. Kotlin의 클래스와 메서드는 기본적으로 `final` 이다.

- 어떤 클래스의 상속을 허용하려면 클래스 앞에 `open` 변경자를 붙어야 한다.
- 이와 더불어 오버라이드를 허용하고 싶은 메서드나 프로퍼티 앞에도 `open` 변경자를 붙어야 한다.
```kotlin
open class RichButton : Clickable {
	fun disable() {} // final
	open fun animate() {}
	override fun click() {}
}
```

- 기반 클래스나 인터페이스의 멤버를 오버라이드하는 경우 그 메서드는 기본적으로 열려 있다.
- 오버라이드하는 메서드의 구현을 하위 클래스에서 오버라이드하지 못하게 금지하려면 오버라이드하는 메서드 앞에 `final`을 명시해야 한다.
```kotlin
open class RichButton : Clickable {
	final override fun click() {}
}
```

### abstract class
- Kotlin도 Java처럼 abstract class를 선언할 수 있다.
- abstract로 선언한 추상 클래스는 인스턴스화 할 수 없다.
- 추상 멤버에는 `open` 변경자를 명시할 필요가 없다.

```kotlin
abstract class Animated {
	abstract fun animate() // 하위 클래스에서 반드시 override 해야된다
	open fun stopAnimating() {} // 추상 클래스에 속했더라도 비 추상함수는 기본적으로 final이다.
	fun animateTwice() {}
}
```

### visibility modifier (default public)
- 아무 변경자도 없는 경우 선언은 모두 공개(public) 된다.
- Kotlin은 패키지를 namespace를 관리하기 위한 용도로만 사용하기 때문에 Java의 기본 visibility인 패키지 전용은 코틀린에 없다.
- Kotlin은 `internal`이라는 새로운 visibility modifier를 도입했다.
	- `internal`은 "모듈 내부에서만 볼 수 있음" 이라는 뜻이다.
	- 모듈은 한 번에 한꺼번에 컴파일되는 코틀린 파일들을 의미한다.
	- 모듈 내부 가시성은 모듈의 구현에 대해 진정한 캡슐화를 제공한다는 장점이 있다.
	- Java에서는 패키지가 같은 클래스를 선언하기만 하면 어떤 프로젝트의 외부에 있는 코드라도 패키지 내부에 있는 패키지 전용 선언에 쉽게 접근할 수 있다. 그래서 모듈의 캡슐화가 쉽게 깨진다.
- Kotlin에서는 최상위 선언에 대해 private visibility를 허용한다.
	- 그런 최상위 선언에는 클래스, 함수, 프로퍼티 등이 포함된다.
	- 비공개 visibility인 최상위 선언은 그 선언이 들어있는 파일 내부에서만 사용할 수 있다.
- Java는 같은 패키지 내에서 `protected`에 접근할 수 있는 반면, Kotlin은 상속 클래스에서만 접근 가능하다.

```kotlin
internal open class TalkativeButton : Focusable {
	private fun yell() = println("Hey!")
	protected fun whisper() = println("Let's talk!")
}

fun TalkativeButton.giveSpeech() { // error: public멤버가 자신의 internal 수신 타입인 TalkativeButton을 노출
	yell() // error: yell 접근 불가
	whisper() // error: whisper 접근 불가
}
```
- 어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 visibility는 그 클래스 자신의 visibility와 같거나 더 높아야 하고, 메서드의 signature에 사용된 모든 타입의 visibility는 그 메서드의 visibility와 같거나 더 높아야 한다는 규칙에 해당한다.

### 내부 클래스
- Kotlin의 중첩 클래스는 명시적으로 요청하지 않는 한 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.

다음은 View 요소를 만드는 예시이다 View의 상태를 직렬화하기 위해 State를 Serializable한다.
```kotlin
interface State: Serializable

interface View {
	fun getCurrentState() : State
	fun restoreState(state: State) {}
}
```

먼저 Java 코드를 보자.
Button 클래스의 상태를 저장하는 클래스는 Button 클래스 내부에 선언하면 편하다.
```java
public class Button implements View {
	@Override
	public State getCurrentState() {
		return new ButtonState();
	}
	
	@Override
	public void restoreState(State state) { /* ... */ }
	
	public class ButtonState implements State { /* ... */ }
}
```
이 코드에서는 버튼의 상태를 직렬화하면 `java.io.NotSerializableException: Button` 오류가 발생한다.

Java에서 다른 클래스 안에 정의한 클래스는 자동으로 내부 클래스(inner class)가 된다.
위 예시의 `ButtonState` 클래스는 바깥쪽 `Button` 클래스에 대한 참조를 묵시적으로 포함한다. 그 참조로 인해 `ButtonState`를 직렬화할 수 없다. `Button`을 직렬화할 수 없으므로 버튼에 대한 참조가 `ButtonState`의 직렬화를 방해한다.

이 문제를 해결하려면 `ButtonState`를 `static` 클래스로 선언해야 한다.
Java에서 중첩 클래스를 `static`으로 선언하면 그 클래스를 둘러싼 바깥쪽 클래스에 대한 묵시적인 참조가 사라진다.

그럼 Kotlin 코드를 보자.
```kotlin
class Button : View {
	override fun getCurrentState() : State = ButtonState()
	override fun restoreState(state: State) { /* ... */ }
	class ButtonState : State { /* ... */ } // 이 클래스는 Java의 정적 중첩 클래스와 대응한다.
}
```
Kotlin 중첩 클래스에 아무런 modifier가 붙지 않으면 Java `static` 중첩 클래스와 같다.
이를 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 `inner` modifier를 붙여야 한다.

| 클래스 B 안에 정의된 클래스 A              | Java           | Kotlin        |
| ------------------------------- | -------------- | ------------- |
| 중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음) | static class A | class A       |
| 내부 클래스(바깥쪽 클래스에 대한 참조를 저장함)     | class A        | inner class A |

Kotlin에서 바깥쪽 클래스의 인스턴스를 가리키는 참조를 표기하는 방법도 Java와 다르다.
내부 클래스 Inner 안에서 바깥쪽 클래스 Outer의 참조에 접근하려면 `this@Outer`라고 써야 한다.
```kotlin
class Outer {
	inner class Inner {
		fun getOuterReference() : Outer = this@Outer
	}
}
```

### 봉인된 클래스
- 상위 클래스에 `sealed` modifier를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.
- `sealed`로 표시된 클래스는 자동으로 `open`이다. 따라서 별도로 `open` modifier를 붙일 필요가 없다.

- Kotlin 1.5 이전까지는 하위 클래스는 중첩 클래스여야 하고, data 클래스로 sealed 클래스를 상속할 수도 없다.
- Kotlin 1.5 부터는 sealed 클래스가 정의된 패키지 안의 아무 위치(최상위, 다른 클래스나 객체나 인터페이스에 내포된 위치)에 선언할 수 있게 됐고, 봉인된 인터페이스도 추가됐다.

### 주 생성자(primary constructor)
- 주 생성자는 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화 되는 프로퍼티를 정의하는 두 가지 목적에 쓰인다.
- `constructor` 키워드는 주 생성자나 부 생성자 정의를 시작할 때 사용한다.
- `init` 키워드는 초기화 블록을 시작한다. 초기화 블록에는 클래스의 객체가 만들어질 때 실행될 초기화 코드가 들어간다.
	- 초기화 블록은 주 생성자와 함께 사용된다.
- 한 클래스 안에 여러 초기화 블록을 선언할 수도 있다.
```kotlin
class User constructor(_nickname: String) {
	val nickname: String
	
	init {
		nickname = _nickname
	}
}
```

- `nickname` 프로퍼티 초기화하는 코드를 `nickname` 프로퍼티 선언에 포함시킬 수 있어서 초기화 코드를 초기화 블록에 넣을 필요가 없다.
- 주 생성자 앞에 별다른 annotation이나 visibility modifier가 없다면 `constructor`를 생략해도 된다.
```kotlin
class User(_nickname: String) {
	val nickname = _nickname
}
```

- 주 생성자의 파라미터로 프로퍼티를 초기화한다면 그 주 생성자 파라미터 이름 앞에 `val`을 추가하는 방식으로 프로퍼티 정의와 초기화를 간략히 쓸 수 있다.
```kotlin
class User(val nickname: String)
```

- 생성자 파라미터에도 default 값을 정의할 수 있다.
```kotlin
class User(val nickname: String,
		val isSubscribed: Boolean = true)
```
- 모든 생성자 파라미터에 default 값을 지정하면 컴파일러가 자동을 파라미터가 없는 생성자를 만들어준다.
	- 이를 통해 JPA에서 기본 생성자가 필요한 곳에 활용도 가능하다

- 별도의 생성자를 정의하지 않으면 컴파일러가 자동으로 인자가 없는 default 생성자를 만들어준다.
```kotlin
open class User
```
- User의 생성자는 아무 인자도 받지 않지만 User 클래스를 상속한 하위 클래스는 반드시 User 클래스의 생성자를 호출해야 한다.
```kotlin
class Man : User()
```
- 당연히 인터페이스는 생성자가 없기 때문에 괄호가 붙지 않는다.

- 주 생성자에 private modifier를 붙여 private 생성자를 만들 수 있다.
```kotlin
class Util private constructor() {}
```
- 당연히 정적 유틸 클래스를 만들려고 이 짓거리를 할 필요가 없다.
	- Kotlin에서는 최상위 함수를 사용할 수 있기 때문이다.
- Kotlin에서는 singleton 객체를 만들 때 Java 처럼 private 생성자를 만들 필요는 없다. (추후에 설명)

#### 주 생성자 호출
- Java와 다르게 `new` 키워드를 붙이지 않는다.
```kotlin
val user = User("유저")
```

### 부 생성자
- Kotlin에서 생성자를 여러 개 쓰는 경우는 적지만 Spring이나 JPA와 같은 프레임워크, 라이브러리를 위해 생성자를 추가 제공해야 하는 상황이 생길 때도 있다.

```kotlin
open class View {
	constructor(ctx: Context) {
		// 코드
	}
	
	constructor(ctx: Context, attr: AttributeSet) {
		// 코드
	}
}
```
- 이 클래스는 주 생성자를 선언하지 않고 부 생성자만 2가지 선언한다.
- 이 클래스를 확장하면서 똑같이 부 생성자를 정의할 수 있다.
```kotlin
class MyButton : View {
	constructor(ctx: Context) : this(ctx, MY_STYLE) {
	}
	
	constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {
	}
}
```
- `super()`를 통해 자신에 대응하는 상위 클래스 생성자를 호출한다.
- `this()`를 통해 자신의 다른 생성자를 호출할 수도 있다.
- 클래스에 주 생성자가 없다면 모든 부 생성자는 반드시 상위 클래스를 초기화하거나 다른 생성자에게 생성을 위임해야 한다.
- 부 생성자의 1차적인 목표는 자바와의 상호운용성이다. 하지만 클래스 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우에는 부 생성자를 여럿 둘 수 밖에 없다.