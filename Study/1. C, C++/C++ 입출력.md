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
