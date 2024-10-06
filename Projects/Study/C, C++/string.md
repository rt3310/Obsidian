## basic_string

[std::string](https://modoocode.com/237) 은 사실 [basic_string](https://modoocode.com/234) 이라는 클래스 템플릿의 인스턴스화 버전이다. [basic_string](https://modoocode.com/234) 의 정의를 살펴보면 아래와 같다.
```cpp
template <class CharT, class Traits = std::char_traits<CharT>,
          class Allocator = std::allocator<CharT> >
class basic_string;
```
[basic_string](https://modoocode.com/234) 은 `CharT` 타입의 객체들을 메모리에 연속적으로 저장하고, 여러 가지 문자열 연산들을 지원해주는 클래스이다. 만약에 `CharT` 자리에 `char` 이 오게 된다면, 우리가 생각하는 [std::string](https://modoocode.com/237) 이 되는 것이다. 사실 우리가 아는 [string](https://modoocode.com/237) 말고도 총 5가지 종류의 인스턴스화 된 문자열들이 있는데

| 타입                                          | 정의                            | 비고                                                          |
| ------------------------------------------- | ----------------------------- | ----------------------------------------------------------- |
| [std::string](https://modoocode.com/237)    | `std::basic_string<char>`     |                                                             |
| [std::wstring](https://modoocode.com/237)   | `std::basic_string<wchar_t>`  | `wchar_t` 의 크기는 시스템 마다 다름. 윈도우에서는 2 바이트이고, 유닉스 시스템에서는 4 바이트 |
| `std::u8string`                             | `std::basic_string<char8_t>`  | `C++ 20` 에 새로 추가되었음; `char8_t` 는 1 바이트; UTF-8 문자열을 보관할 수 있음 |
| [std::u16string](https://modoocode.com/237) | `std::basic_string<char16_t>` | `char16_t` 는 2 바이트; UTF-16 문자열을 보관할 수 있음                    |
| [std::u32string](https://modoocode.com/237) | `std::basic_string<char32_t>` | `char32_t` 는 4 바이트; UTF-32 문자열을 보관할 수 있음                    |
와 같이 있다.

> [!NOTE]
> 보다시피, wchar_t 의 크기가 시스템마다 다르기 때문에 확실한 2바이트 타입과 4바이트 타입을 만들기 위해 char16_t 와 char32_t 가 탄생했다.

그렇다면 나머지 템플릿 인자들을 살펴보자. `Traits` 는 주어진 `CharT` 문자들에 대해 **기본적인 문자열 연산들을 정의해놓은 클래스**를 의미한다. 여기서 기본적인 문자열 연산들이란, **주어진 문자열의 대소 비교를 어떻게 할 것인지, 주어진 문자열의 길이를 어떻게 잴 것인지 등**을 말한다.

다시 말해 [basic_string](https://modoocode.com/234) 에 정의된 문자열 연산들은 사실 전부다 `Traits` 의 기본적인 문자열 연산들을 가지고 정의되어 있다. 덕분에 문자열들을 어떻게 보관하는지에 대한 로직과 문자열들을 어떻게 연산하는지에 대한 로직을 분리시킬 수 있었다. 전자는 [basic_string](https://modoocode.com/234) 에서 해결하고, 후자는 `Traits` 에서 담당하게 된다.

이렇게 로직을 분리한 이유는 [basic_string](https://modoocode.com/234) 의 사용자에게 좀 더 자유를 부여하기 위해서다. 예를 들어, [string](https://modoocode.com/237) 처럼 `char` 이 기본 타입인 문자열에서, 문자열 비교 시 대소문자 구분을 하지 않는 버전을 만들고싶다고 해보자.
그렇다면 그냥 처음 부터 `Traits` 에서 문자열들을 비교하는 부분만 살짝 바꿔주면 된다. 만일 `Traits` 가 없었다면 [basic_string](https://modoocode.com/234) 에서 문자열을 비교하는 부분을 일일이 찾아서 고쳐야 할 것이다. [여기](https://en.cppreference.com/w/cpp/string/char_traits) 에서 예시 코드를 볼 수 있다.

`Traits` 에는 string에 정의된 `std::char_traits` 클래스의 인스턴스화 버전을 전달한다. 예를 들어서 string의 경우 `char_traits<char>` 을 사용하게 된다.

### 숫자들의 순위가 알파벳보다 낮은 문자열
`Traits` 가 어떻게 활용되는지 좀 더 자세히 살펴보기 위해, 직접 `Traits` 클래스를 오버로딩해서 문자열 비교 시의 숫자들의 순위가 제일로 낮은 문자열을 만들어보자. 이게 무슨 말이냐면 원래 ASCII 테이블에서 숫자들의 값이 알파벳보다 작아서 더 앞에 오게 된다. 즉, `1a` 가 `a1` 보다 앞에 온다는 뜻이다.
하지만 이를 바꿔서 숫자들이 다른 문자들 보다 우선순위가 낮은 문자열을 한 번 만들어보자.
```cpp
#include <cctype>
#include <iostream>
#include <string>

// char_traits의 모든 함수들은 static 함수이다.
struct my_char_traits : public std::char_traits<char> {
	static int get_real_rank(char c) {
		// 숫자면 순위를 엄청 떨어트린다.
		if (isdigit(c)) {
			return c + 256;
		}
		return c;
	}

	static bool lt(char c1, char c2) {
		return get_real_rank(c1) < get_real_rank(c2);
	}

	static int compare(const char* s1, const char* s2, size_t n) {
		while (n-- != 0) {
			if (get_real_rank(*s1) < get_real_rank(*s2)) {
				return -1;
			}
			if (get_real_rank(*s1) > get_real_rank(*s2)) {
				return 1;
			}
			++s1;
			++s2;
		}
		return 0;
	}
};

int main() {
	std::basic_string<char, my_char_traits> my_s1 = "1a";
	std::basic_string<char, my_char_traits> my_s2 = "a1";

	std::cout << "숫자의 우선순위가 더 낮은 문자열 : " << std::boolalpha << (my_s1 < my_s2) << std::endl;

	std::string s1 = "1a";
	std::string s2 = "a1";

	std::cout << "일반 문자열 : " << std::boolalpha << (s1 < s2) << std::endl;
}
```
성공적으로 컴파일했다면
```
숫자의 우선순위가 더 낮은 문자열 : false
일반 문자열 : true
```

```cpp
struct my_char_traits : public std::char_traits<char> {
```
[basic_string](https://modoocode.com/234) 의 `Traits` 에는 `char_traits` 에서 제공하는 모든 멤버 함수들이 구현된 클래스가 전달되어야 한다(꼭 `char_traits` 를 상속 받을 필요는 없다). 이를 가장 간편히 만들기 위해서는 그냥 `char_traits` 를 상속 받은 후, 필요한 부분만 새로 구현하면 된다.

`char_traits` 에 정의되는 함수들은 모두 `static` 함수들 이다. 그 이유는 `char_traits` 의 존재 이유를 생각해보면 당연한데, `Traits` 는 문자와 문자열들 간에 간단한 연산을 제공해주는 클래스이므로 굳이 데이터를 저장할 필요가 없기 때문이다(이를 보통 Stateless 하다고 한다).

일반적인 `char` 들을 다루는 `char_traits<char>` 에서 우리가 바꿔줘야 할 부분은 대소 비교하는 부분 뿐이다. 따라서 아래와 같이 문자들 간의 크기를 비교하는 `lt` 함수와 길이 `n` 의 문자열의 크기를 비교하는 `compare` 함수를 새로 정의해줘야 했다.
```cpp
static bool lt(char c1, char c2) {
	return get_real_rank(c1) < get_real_rank(c2);
}

static int compare(const char* s1, const char* s2, size_t n) {
	while (n-- != 0) {
		if (get_real_rank(*s1) < get_real_rank(*s2)) {
			return -1;
		}
		if (get_real_rank(*s1) > get_real_rank(*s2)) {
			return 1;
		}
		++s1;
		++s2;
	}
	return 0;
}
```
코드를 읽어보면 그다지 어렵지 않다. `get_real_rank` 함수는 문자를 받아서 숫자면 `256`을 더해서 순위를 매우 떨어뜨린다. 따라서 숫자들이 모든 문자들 뒤에 오게 된다.
```cpp
std::cout << "숫자의 우선순위가 더 낮은 문자열 : " << std::boolalpha
          << (my_s1 < my_s2) << std::endl;
```
따라서 실제로 `my_s1` 이 `my_s2` 보다 뒤에 온다고 나타나게 된다. (`my_s1 > my_s2`) 반면에 보통의 [string](https://modoocode.com/237) 의 경우에는 `s1` 이 `s2` 앞에 나올 것이다.

이와 같이 간단히 `Traits` 만 바꿔주는 것으로 좀 더 커스터마이징 된 [basic_string](https://modoocode.com/234) 을 사용할 수 있다.

## 짧은 문자열 최적화(SSO)

메모리를 할당하는 작업은 시간을 꽤나 잡아먹는다.

[basic_string](https://modoocode.com/234) 이 저장하는 문자열의 길이는 천차만별이다. 때론 한 두 문자 정도의 짧은 문자열을 저장할 때도 있고, 수십만 바이트의 거대한 문자열을 저장할 때도 있다. 문제는 거대한 문자열은 매우 드물게 저장되는데 반해 길이가 짧은 문자열들은 굉장히 많이 생성되고 소멸 된다는 점이다. 만일 매번 모든 문자열을 동적으로 메모리를 할당 받는다고 해보자. 길이가 짧은 문자열을 여러 번 할당한다면 매번 메모리 할당이 이루어져야 하므로, 굉장히 비효율적일 것이다.

따라서 [basic_string](https://modoocode.com/234) 의 제작자들은 짧은 길이 문자열의 경우 따로 문자 데이터를 위한 메모리를 할당하는 대신에 그냥 객체 자체에 저장해버린다. 이를 짧은 문자열 최적화(SSO - short string optimization) 이라고 부른다.
```cpp
#include <iostream>
#include <string>

// 이와 같이 new 를 전역 함수로 정의하면 모든 new 연산자를 오버로딩 해버린다.
// (어떤 클래스의 멤버 함수로 정의하면 해당 클래스의 new 만 오버로딩됨)
void* operator new(std::size_t count) {
	std::cout << count << " bytes 할당 " << std::endl;
	return malloc(count);
}

int main() {
	std::cout << "s1 생성 --- " << std::endl;
	std::string s1 = "this is a pretty long sentence!!!";
	std::cout << "s1 크기 : " << sizeof(s1) << std::endl;
	
	std::cout << "s2 생성 --- " << std::endl;
	std::string s2 = "short sentence";
	std::cout << "s2 크기 : " << sizeof(s2) << std::endl;
}
```
성공적으로 컴파일했다면
```
s1 생성 --- 
34 bytes 할당 
s1 크기 : 32
s2 생성 --- 
s2 크기 : 32
```

```cpp
// 이와 같이 new 를 전역 함수로 정의하면 모든 new 연산자를 오버로딩 해버린다.
// (어떤 클래스의 멤버 함수로 정의하면 해당 클래스의 new 만 오버로딩됨)
void* operator new(std::size_t count) {
	std::cout << count << " bytes 할당 " << std::endl;
	return malloc(count);
}
```
먼저 메모리가 할당되는지 안되는지 확인하기 위해서 위와 같이 새로 `new` 연산자를 정의해줬다. 참고로 `new` 의 경우 위와 같이 클래스 외부의 함수로 정의하게 된다면 모든 `new` 연산자들이 위 함수를 사용하게 된다. 반면에 클래스 내에 멤버 함수로 `new` 를 정의하게 된다면, 해당 객체를 new로 생성할 때 해당 `new` 함수가 호출된다.

아무튼 위와 같이 `operator new` 를 정의한 덕분에 [basic_string](https://modoocode.com/234) 내부를 바꾸지 않고도 문자열 생성 시에 메모리 할당이 일어나는지 아닌지 관찰할 수 있다.

그리고 그 결과는 위와 같다. 길이가 긴 문자열 `s1` 을 생성할 때에는 메모리 할당이 발생하였고, **길이가 짧은 문자열인 `s2` 의 경우에는 메모리 할당이 발생하지 않았다**.

그 대신 문자열 객체의 크기를 확인하였을 때 32 바이트로 꽤나 크다. 만일 정말 단순하게 문자열 라이브러리를 구현하였다면 문자열 길이를 저장할 변수 하나, 할당한 메모리 공간 크기 저장을 위한 변수 하나, 메모리 포인터 하나로 해서 총 12바이트로 만들 수 도 있을 것이다. 하지만 라이브러리 제작자들은 **메모리 사용량을 조금 희생한 대신 성능 향상을 꾀했다**.

물론 **라이브러리 마다 어느 길이 문자열부터 따로 메모리 할당을 할 지는 다르다**. 하지만 **대부분의 주류 C++ 라이브러리들(gcc 의 libstdc++ 과 clang 의 libc++)은 어떤 방식이든 SSO를 사용하고 있다**.

여담으로, C++11 이전에 [basic_string](https://modoocode.com/234) 의 구현에서는 Copy On Write 라는 기법도 사용되었다. 이는 문자열을 복사할 때, 바로 복사하는 것이 아니라, 복사된 문자열이 바뀔 때 비로소 복사를 수행하는 방식이다. 하지만 이는 C++11 에서 개정된 표준에 따라 불가능해졌다.

## 문자열 리터럴 정의하기

C에서 문자열 리터럴을 정의하기 위해선 아래와 같이 했다.
```c
const char* s = "hello";
// 혹은
char s[] = "hello";
```
위 두 `s` 모두 "`hello`" 라는 문자열을 보관하게 된다.

C++ 의 경우는 어떨까? 만약에
```cpp
auto str = "hello"
```
를 하면 [str](https://modoocode.com/str) 는 [string](https://modoocode.com/237) 으로 정의될까? 아니다. C++ 에서는 C와 마찬가지로 [str](https://modoocode.com/str) 의 타입은 `const char *`로 정의된다. 이는 C를 배우지 않고 C++부터 배운 사람에게는 혼란스러울 여지가 있다. 만일 문자열을 꼭 만들어야겠다 한다면
```cpp
string str = "hello";
```
위처럼 타입을 꼭 명시해줘야 한다. 하지만 C++14에 이 문제를 깜찍하게 해결할 수 있는 방법이 나왔다.

### 리터럴 연산자
재미있게도, C++14에서 리터럴 연산자(literal operator) 라는 것이 새로 추가되었다.
```cpp
auto str = "hello"s;
```
위와 같이 `""` 뒤에 `s` 를 붙여주면 `auto` 가 [string](https://modoocode.com/237) 으로 추론된다. 참고로 이 리터럴 연산자는
```cpp
std::string operator"" s(const char *str, std::size_t len);
```
위 처럼 정의되어 있는데, `"hello"s` 는 컴파일 과정에서 `operator""s("hello", 5);` 로 바뀌게 된다. 참고로 해당 리터럴 연산자를 사용하기 위해서는 **`std::string_literals` 네임스페이스**를 사용해야 한다. 아래 코드를 보자
```cpp
#include <iostream>
#include <string>
using namespace std::literals;

int main() {
	auto s1 = "hello"s;
	std::cout << "s1 길이 : " << s1.size() << std::endl;
}
```
성공적으로 컴파일했다면
```
s1 길이 : 5
```
와 같이 제대로 [string](https://modoocode.com/237) 으로 `auto` 가 추론된 것을 확인할 수 있다.

리터럴 연산자는 위처럼 문자열 리터럴만 가능한 것이 아니라 정수나 부동 소수점 리터럴들 역시 사용 가능하다. 자세한 예시는 [여기](https://en.cppreference.com/w/cpp/language/user_literal)를 살펴보자.