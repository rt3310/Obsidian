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
[basic_string](https://modoocode.com/234) 의 `Traits` 에는 `char_traits` 에서 제공하는 모든 멤버 함수들이 구현된 클래스가 전달되어야 합니다. (꼭 `char_traits` 를 상속 받을 필요는 없습니다) 이를 가장 간편히 만들기 위해서는 그냥 `char_traits` 를 상속 받은 후, 필요한 부분만 새로 구현하면 됩니다.

`char_traits` 에 정의되는 함수들은 모두 `static` 함수들 입니다. 그 이유는 `char_traits` 의 존재 이유를 생각해보면 당연한데, `Traits` 는 문자와 문자열들 간에 간단한 연산을 제공해주는 클래스이므로 굳이 데이터를 저장할 필요가 없기 때문입니다. (이를 보통 Stateless 하다고 합니다.)

일반적인 `char` 들을 다루는 `char_traits<char>` 에서 우리가 바꿔줘야 할 부분은 대소 비교하는 부분 뿐입니다. 따라서 아래와 같이 문자들 간의 크기를 비교하는 `lt` 함수와 길이 `n` 의 문자열의 크기를 비교하는 `compare` 함수를 새로 정의해줘야 했습니다.