## C++ 입출력 라이브러리

C++ 입출력 라이브러리는 다음과 같은 클래스들로 구성되어 있다.
![[Pasted image 20260801173853.png]]
C++의 모든 입출력 클래스는 `ios_base`를 기반 클래스로 하게 된다.
`ios_base` 클래스는 많은 일은 하지 않고, 스트림의 입출력 형식 관련 데이터를 처리한다.
예를 들어, 실수 형을 출력할 때 정밀도를 어떤 식으로 할 것인지에 대해, 혹은 정수형을 출력 시에 10진수로 할지, 16진수로 할지 등을 이 클래스에서 처리한다.

그 다음으로 `ios` 클래스가 있다. 이 클래스에서는 실제로 스트림 버퍼를 초기화 한다. 스트림 버퍼란, 데이터를 내보내거나 받아들이기 전에 임시로 저장하는 곳이라 볼 수 있다.
그 외에도, 현재 입출력 작업의 상태를 처리한다. 예를 들어, 파일을 읽다가 끝에 도달했는지 안했는지 확인하려면 `eof` 함수를 호출하면 된다. 또, 현재 입출력 작업을 잘 수행했는지 확인하려면 `good` 함수를 호출하면 된다.

### istream
`ios_base`와 `ios` 클래스들이 입출력 작업을 위해 바탕을 깔아주는 클래스였다면, `istream`은 실제로 입력을 수행하는 클래스이다.

대표적으로 우리가 항상 사용하던 `operator>>`가 이 `istream` 클래스에 정의되어 있는 연산자이다.
또, `cin`은 `istream` 클래스의 객체 중 하나이다. 그렇기 때문에
```cpp
std::cin >> a;
```
와 같은 작업을 할 수 있었던 것이다.

어떤 타입에 대해서도 `cin`을 사용할 수 있었던 이유는 `operator>>`가 그런 모든 기본 타입들에 대해서는 정의가 되어 있기 때문이다.
```cpp
istream& operator>>(bool& val);

istream& operator>>(short& val);

istream& operator>>(unsigned short& val);

istream& operator>>(int& val);

istream& operator>>(unsigned int& val);

istream& operator>>(long& val);

istream& operator>>(unsigned long& val);

istream& operator>>(long long& val);

istream& operator>>(unsigned long long& val);

istream& operator>>(float& val);

istream& operator>>(double& val);

istream& operator>>(long double& val);

istream& operator>>(void*& val);
```
그렇다고 해서, 언제나 위 타입들 빼고는 `operator>>`로 받을 수 없는 것이 아니다.
실제로 `istream` 클래스의 멤버 함수로는 없지만
```cpp
std::string s;
std::cin >> s;
```
`std::string` 클래스의 객체 `s`도 `cin`으로 입력받을 수 있다. 이와 같은 일이 가능한 이유는 멤버 함수를 두는 것 말고도, 외부 함수로 연산자 오버로딩을 할 수 있기 때문이다.

이 경우에는
```cpp
istream& operator>>(istream& in, std::string& s)
{
	// 구현
}
```
와 같이 하면 된다.

`operator>>`의 또 다른 특징으로는, 모든 공백문자(띄어쓰기, 탭, 엔터 등)을 입력 시에는 무시해버린다는 점이다.
그렇기 때문에, 만약 `cin`을 통해서 문장을 입력받는다면, 첫 단어만 입력받고 나머지를 읽을 수 없다.
```cpp
#include <iostream>
#include <string>

int main() {
	std::string s;
	while (true) {
		std::cin >> s;
		std::cout << "word : " << s << std::endl;
	}
}
```
성공적으로 컴파일 했다면
```
this is a long sentence
word : this
word : is
word : a
word : long
word : sentence
ABCD EFGH IJKL
word : ABCD
word : EFGH
word : IJKL
```
와 같이 문장을 입력하더라도, 공백문자에 따라서 각각을 분리해서 입력받게 되는 것이다.
위와 같이 비록 `operator>>`가 매우 편리해보이지만, 반드시 주의해야할 점이 있다.
```cpp
// 주의할 점
#include <iostream>
using namespace std;
int main() {
	int t;
	while (true) {
		std::cin >> t;
		std::cout << "입력 :: " << t << std::endl;
		if (t == 0) break;
	}
}
```
성공적으로 컴파일 했다면
```
3
입력 :: 3
4
입력 :: 4
5
입력 :: 5
6
입력 :: 6
7
입력 :: 7
```
여기서 만약 숫자가 아니라 문자를 입력했다고 해보자.

그럼 다음과 같이 원하던 결과가 나오지 않음을 알 수 있다. (c 하나 입력했을 때의 결과이다)
![[Pasted image 20260801183037.png]]
왜 이런 무한 루프에 빠지는 것일까? 그 이유는 `operator>>`가 어떻게 이를 처리하는지 이해하면 알 수 있다.

앞서 `ios` 클래스에서 스트림의 상태를 관리한다고 했다. 이때, 스트림의 상태를 관리하는 플래그(flag)는 4개가 정의되어 있다. 이 4개의 flag들이 스트림이 현재 어떤 상태인지에 대해 정보를 보관한다.

4개의 flag는 각각 `goodbit`, `badbit`, `eofbit`, `failbit`이렇게 있다.
- `goodbit`: 스트림에 입출력 작업이 가능할 때
- `badbit`: 스트림에 복구 불가능한 오류 발생 시
- `failbit`: 스트림에 복구 가능한 오류 발생 시
- `eofbit`: 입력 작업 때 EOF 도달 시
위와 같은 상황일 때 각각의 비트들이 켜진다.
만약 위와 같이 문자를 입력할 경우 `operator>>`는 `failbit`가 켜지게 된다. 그리고, 입력 값을 받지 않고 반환해버린다.

문제는 이렇게 그냥 반환해버리면서 버퍼에 남아있는 'c\n'이 문자열에는 손대지 않는다는 것이다. 그렇기 때문에 계속 반복해서 읽고 위와 같은 문제를 일으키는 것이다.

그럼 어떻게 해결해야 할까?
```cpp
// 해결 방안
#include <iostream>
#include <string>

int main() {
	int t;
	while (std::cin >> t) {
		std::cout << "입력 :: " << t << std::endl;
		if (t == 0) break;
	}
}
```
성공적으로 컴파일 했다면
```
4
입력 :: 4
3
입력 :: 3
s
```
위와 같이 무한루프에 빠지지 않고 제대로 처리됨을 알 수 있다.

이는
```cpp
while (std::cin >> t) {
```
이 구문의 변화에 있는데, `ios`에 정의되어 있는 함수들을 살펴보면 다음과 같은 함수가 있음을 알 수 있다.
```cpp
operator void*() const;
```
이 함수는 `ios` 객체를 `void*`로 변환해준다. 이 때, `NULL` 포인터가 아닌 값을 반환하는 조건이, `failbit`와 `badbit`가 모두 off일 때 이다. 즉, 스트림에 정상적으로 입출력 작업을 수행할 수 있을 때를 말한다.

그럼 만약 's'를 입력한다면 `operator>>`는 `cin` 객체의 `failbit`를 켜게 될 것이다. 그리고, `std::cin >> t` 후에 `cin`이 반환되는데, 반환값이 `while` 문의 조건식으로 들어가기 때문에 컴파일러는 적절한 타입 변환을 찾게 되고, 결국 `ios` 객체 → `void*` → `bool`로 가는 2단 변환을 통해서 `while` 문을 잘 빠져나오게 된다. (`NULL` 포인터는 `bool` 상 `false`이다)

위와 같이 문제를 해결할 수는 있지만, 입력을 계속 진행할 수는 없다. 왜냐하면 현재 `cin`에 `failbit`가 켜진 상태이므로, flag를 초기화해버리지 않는 한 `cin`을 이용하여 입력 받을 수 없게 된다.
```cpp
#include <iostream>
#include <string>

int main() {
	int t;
	std::cin >> t;  // 고의적으로 문자를 입력하면 failbit 가 켜진다
	std::cout << "fail 비트가 켜진 상태이므로, 입력받지 않는다" << std::endl;
	std::string s;
	std::cin >> s;
}
```
성공적으로 컴파일 했다면
```
s
fail 비트가 켜진 상태이므로, 입력받지 않는다
```
그럼 이 문제를 어떻게 해결할 수 있을까?

```cpp
#include <iostream>
#include <string>

int main() {
	int t;
	while (true) {
		std::cin >> t;
		std::cout << "입력 :: " << t << std::endl;
		if (std::cin.fail()) {
			std::cout << "제대로 입력해주세요" << std::endl;
			std::cin.clear();            // 플래그들을 초기화 하고
			std::cin.ignore(100, '\n');  // 개행문자가 나올 때 까지 무시한다
		}
		if (t == 1) break;
	}
}
```
성공적으로 컴파일 했다면
```
a
입력 :: 0
제대로 입력해주세요
x
입력 :: 0
제대로 입력해주세요
d
입력 :: 0
제대로 입력해주세요
asdf
입력 :: 0
제대로 입력해주세요
2
입력 :: 2
1
입력 :: 1
```
위와 같이 잘 처리되는 것을 볼 수 있다.
`fail()`은 `ios`에 정의되어 있으며, `failbit`가 `true`거나 `badbit`가 `true` 면 `true`를 반환한다. 만일 숫자가 아닌 것을 입력한다면, `failbit`가 `true`이므로, `std::cin.fail()`은 `true`가 되어 조건문을 실행하게 된다.

`clear()`역시 `ios`에 정의되어 있으며, 인자를 주지 않을 경우 flag를 `goodbit`으로 초기화시켜버린다. 따라서 `fail` 상태를 지울 수 있게 된다.
`ignore()`은 `istream`에 정의되어 있는데, 최대 첫 번째 인자(100)만큼 두 번째 인자('\n')가 나올 때까지 버퍼에서 무시한다.

### 형식 플래그(format flag)와 조작자(Manipulator)
앞서 `ios_base` 클래스에서, 스트림의 입출력 형식을 바꿀 수 있다고 했다.
예를 들어, 여태까지 수를 입력하면 10진수로 처리되었지만, 16진수로도 처리할 수 있다.
```cpp
#include <string>
#include <iostream>

int main() {
	int t;
	while (true) {
		std::cin.setf(std::ios_base::hex, std::ios_base::basefield);
		std::cin >> t;
		std::cout << "입력 :: " << t << std::endl;
		if (std::cin.fail()) {
			std::cout << "제대로 입력해주세요" << std::endl;
			std::cin.clear();  // 플래그들을 초기화 하고
							 // std::cin.ignore(100,'n');//개행문자가 나올 때까지
							 // 무시한다
		}
		if (t == 0) break;
	}
}
```
성공적으로 컴파일했다면
```
ff
입력 :: 255
0xFF
입력 :: 255
123
입력 :: 291
ABCDE 
입력 :: 703710
```
위와 같이 16진수 입력을 잘 받는다는 것을 볼 수 있다. (출력 형식은 바꾸지 않았으므로, 10진수로 출력된다)
이처럼 입력 받는 형식을 16진수로 바꿔준 함수는 보다시피 아래와 같은 스트림의 성질을 바꾸는 `setf` 함수 덕분이다.
```cpp
std::cin.setf(ios_base::hex, ios_base::basefield);
```
`setf` 함수의 버전은 2개가 있는데, 하나는 인자를 1개만 받는 것이고 다른 하나는 위처럼 인자를 2개 받는 것이다.

인자 1개를 받는 `setf`는 그냥 인자로 준 형식 플래그를 적용하는 것이지만, 2개 받는 것은 두 번째 인자(`basefield`)의 내용을 초기화하고 첫 번째 인자(`hex`)를 적용하는 것이다.

위의 경우, 수를 처리하는 방식은 1가지 진수만 한 번에 처리할 수 있으므로, 몇 진법으로 수를 처리할지 보관하는 `basefield`의 값을 초기화하고, 16진법(`hex`) 플래그를 적용시킨 것이다.

물론, 16진법을 처리하는 함수를 그냥 만들어도 된다. 하지만 사용자에 따라 `0x`를 붙이거나 `a123`, `A123` 등 다양한 경우가 있기 때문에 라이브러리에서 지원하는 방법을 사용하는 것이 좋다.

위와 다른 방식으로 16진수를 받을 수도 있다.
```cpp
// 조작자의 사용
#include <iostream>
#include <string>

int main() {
	int t;
	while (true) {
		std::cin >> std::hex >> t;
		std::cout << "입력 :: " << t << std::endl;
		if (std::cin.fail()) {
			std::cout << "제대로 입력해주세요" << std::endl;
			std::cin.clear();           // 플래그들을 초기화 하고
			std::cin.ignore(100, 'n');  //개행문자가 나올 때까지 무시한다
		}
		if (t == 0) break;
	}
}
```
성공적으로 컴파일했다면
```
ff
입력 :: 255
0xFF
입력 :: 255
123
입력 :: 291
ABCDE 
입력 :: 703710
```
위 경우 16진수를 잘 입력받는 것을 볼 수 있다.

```cpp
std::cin >> hex >> t;
```
바로 위에서 `hex`가 `cin`에서 수를 받는 방식을 바꿔버렸기 때문이다. 이 때문에 `hex`와 같이, 스트림을 조작하여 입력/출력 방식을 바꿔주는 함수를 조작자라고 부른다. (`hex`는 함수이다)

위에서 사용했던 형식 플래그 `hex`는 `ios_base`에 선언되어 있는 단순한 상수 값이고 조작자 `hex`의 경우 `ios`에 정의되어 있는 함수이다.
이 `hex`의 정의를 살펴보면, `ios_base` 객체를 레퍼런스로 받고, 다시 반환하도록 정의되어 있다.
```cpp
std::ios_base& hex(std::ios_base& str);
```

또한, `operator>>` 중에서 위 함수를 인자로 가지는 경우도 있다.
```cpp
istream& operator>>(ios_base& (*pf)(ios_base&));
```
이렇게 `operator>>`에서 조작자를 받는다면 많은 일을 하는 것이 아니라 단순히 `pf(*this)`를 호출하게 된다.
호출된 hex 함수가 하는 일 또한 별로 없다. 단순히
```cpp
str.setf(std::ios_base::hex, std::ios_base::basefield)
```
를 수행해주는 것이다.

이렇게 `setf`를 사용하지 않더라도, 간단하게 조작자를 사용하면 훨씬 쉽게 입력 형식을 바꿀 수 있게 된다.
`hex` 말고도 꽤 많은데, `true`나 `false`를 1과 0으로 처리하는 대신 문자열 그대로 입력받는 `boolalpha`도 있고, 출력 형식으로 왼쪽/오른쪽으로 정렬시키는 `left`와 `right` 조작자 등 여러가지가 있다.

그 외에도 우리가 여태 아무 생각없이 사용했던 `std::endl`도 있다.
`endl`은 `hex`와는 달리 출력을 관장하는 `ostream`에 정의되어 있는 조작자로, 한 줄 개행문자를 출력하는 것 말고도, 버퍼를 모두 내보내는 `flush` 역할도 수행한다.

### 스트림 버퍼
모든 입출력 객체들은 이에 대응되는 스트림 객체를 가지고 있게 된다. 따라서 C++의 입출력 라이브러리에는 이에 대응되는 스트림 버퍼 클래스도 있는데, 바로 `streambuf` 클래스이다.

사실, 스트림이라 하면 쉽게 말해 문자들의 순차적인 나열이라고 생각하면 된다.
예를 들어, 우리가 화면에 입력하는 문자도 스트림을 통해서 프로그램에 전달되는 것이고, 하드디스크에서 파일을 읽는 것도 다른 컴퓨터와 `TCP/IP` 통신하는 것도 모두 스트림을 통해 이루어진다.

심지어 C++ 에서는 `std::stringstream`을 통해서 평범한 문자열을 마치 스트림인 것처럼 이용할 수도 있게 해준다.

![[Pasted image 20260801234517.png]]
위 사진은 `streambuf` 클래스에서 스트림을 어떤 식으로 추상화하고 있는지 보여주는 그림이다.
`streambuf`는 그림과 같이 맨 아래에 나타나있는 스트림에서 입력을 받던지, 출력을 하던지, 혹은 입력과 출력을 동시에 수행할 수도 있다.

`streambuf` 클래스는 스트림의 상태를 나타내기 위해 3개의 포인터를 정의하고 있다.
- 버퍼의 시작 부분을 가리키는 시작 포인터
- 다음으로 읽을 문자를 가리키고 있는 포인터(스트림 위치 지정자)
- 버퍼의 끝 부분을 가리키고 있는 끝 포인터
`streambuf` 클래스는 입력 버퍼와 출력 버퍼를 구분해서 각각 `get area`와 `put area`라 부르는데, 이에 따라 각각을 가리키는 포인터도 `g`와 `p`를 붙여서 표현하게 된다.

아래 예제를 통해 `streambuf`를 어떻게 하면 간단히 조작할 수 있는지 살펴보자.
```cpp
#include <iostream>
#include <string>

int main() {
	std::string s;
	std::cin >> s;
	
	// 위치 지정자를 한 칸 옮기고, 그 다음 문자를 훔쳐본다 (이 때는 움직이지 않음)
	char peek = std::cin.rdbuf()->snextc();
	if (std::cin.fail()) std::cout << "Failed";
	std::cout << "두 번째 단어 맨 앞글자 : " << peek << std::endl;
	std::cin >> s;
	std::cout << "다시 읽으면 : " << s << std::endl;
}
```
성공적으로 컴파일 했다면
```
hello world 두 번째 단어 맨 앞글자 : w 다시 읽으면 : world
```
위와 같이 나옴을 알 수 있다.

```cpp
char peek = std::cin.rdbuf()->snextc();
```
입력 객체 `cin`의 `rdbuf()`를 호출하게 되면, `cin` 객체가 입력을 수행하고 있던 `streambuf` 객체를 가리키는 포인터를 반환하게 된다. 이 때, `cin` 객체가 `istream` 객체이므로 오직 입력만을 수행하고 있기 때문에 이 `streambuf` 객체에는 오직 `get area`만 있음을 알 수 있다.

`snextc()`는 스트림 위치 지정자를 한 칸 전진시킨 후 그 자리에 해당하는 문자를 엿본다(읽는 것이 아니다).
읽는 것과의 차이는 읽게 되면 스트림 위치 지정자를 한 칸 전진시켜서 다음 읽기 때 다음 문자를 읽을 수 있도록 준비하지만, 엿보게되면 해당 문자를 읽고도 스트림 위치 지정자를 움직이지 않는다.

그럼 `peek`의 결과가 왜 `w`가 나올까?
![[Pasted image 20260802020627.png]]
위는 `hello world`를 친 다음, `std::cin >> s`를 한 이후의 `streambuf` 상태이다. 문자열의 경우 공백문자가 나오기 전까지 읽어들이기 때문에 위와 같은 상태가 된다.

이제, `snextc()` 함수를 호출했을 때 상태를 보면
![[Pasted image 20260802020725.png]]
`snextc()` 함수가 스트림 위치 지정자를 한 칸 전진시키므로, 공백 문자를 띄어넘고 `w`를 가리키게 된다. 그리고 이에 해당하는 문자인 `w`를 반환한다. 이때 `snextc()` 함수는 스트림 위치 지정자를 건드리지 않기 때문에
```cpp
std::cin >> s;
std::cout << "다시 읽으면 : " << s << std::endl;
```
여기서 `world` 전체가 나오게 된다.

`streambuf`에는 `snextc()` 함수 말고도 수 많은 함수들이 정의되어 있다. 물론 이 함수들을 직접 사용할 일은 거의 없다. C++ 입출력 라이브러리는 스트림 버퍼도 추상화해서 클래스로 만들었다는 것 정도로 기억하면 좋다.

C++에서 `streambuf`를 도입한 중요한 이유 한 가지는, 1byte 짜리 문자 뿐만 아니라, `wchar_t` 즉, 다중 바이트 문자들(`UTF-8` 같은 것)에 대한 처리도 용이하게 하기 위해서이다.

예를 들어, 다중 바이트 문자의 경우, 사용자가 문자 한 개만 요구했음에도 스트림에서는 1byte만 읽을 수 있고, 2bytes, 심지어 4bytes까지 필요한 경우가 있다. C++에서는 이러한 것들에 대한 처리를 스트림 버퍼 객체 자체에서 수행하도록 해서 사용자가 입출력 처리를 이용하는데 훨씬 용이하게 했다.