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
- 자바와 동일한 Collection (상호작용 가능)
```kotlin
val strings = listOf("first", "second")
val last = strings.last() // second
```