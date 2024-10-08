## Move

우측값 레퍼런스를 통해서, 기존에는 불가능했던 우측값에 대한 복사가 아닌 이동의 구현이 가능하게 되었다.
하지만, 만약에 좌측값도 이동을 시키고 싶다면 어떨까? 예를 들어서 아래와 같이 두 변수의 값을 바꾸는 `swap` 함수를 생각해보자.
```cpp
template <typename T>
void my_swap(T &a, T &b) {
	T tmp(a);
	a = b;
	b = tmp;
}
```
위 `my_swap` 함수에서 `tmp` 라는 임시 객체를 생성한 뒤에, `b` 를 `a` 에 복사하고, `b` 에 `a` 를 복사하게 된다. 문제는 무려 복사를 쓸데없이 3번이나 한다는 점이다. 예를 들어서 `T` 가 `MyString` 인 경우를 생각해보자.
```cpp
#include <iostream>
#include <cstring>

class MyString {
	char *string_content;  // 문자열 데이터를 가리키는 포인터
	int string_length;     // 문자열 길이
	
	int memory_capacity;  // 현재 할당된 용량

public:
	MyString();
	
	// 문자열로 부터 생성
	MyString(const char *str);
	
	// 복사 생성자
	MyString(const MyString &str);
	
	// 이동 생성자
	MyString(MyString &&str);
	
	MyString &operator=(const MyString &s);
	~MyString();
	
	int length() const;
	
	void println();
};

MyString::MyString() {
	std::cout << "생성자 호출 ! " << std::endl;
	string_length = 0;
	memory_capacity = 0;
	string_content = NULL;
}

MyString::MyString(const char *str) {
	std::cout << "생성자 호출 ! " << std::endl;
	string_length = strlen(str);
	memory_capacity = string_length;
	string_content = new char[string_length];
	
	for (int i = 0; i != string_length; i++) string_content[i] = str[i];
}
MyString::MyString(const MyString &str) {
	std::cout << "복사 생성자 호출 ! " << std::endl;
	string_length = str.string_length;
	string_content = new char[string_length];
	
	for (int i = 0; i != string_length; i++)
		string_content[i] = str.string_content[i];
}
MyString::MyString(MyString &&str) {
	std::cout << "이동 생성자 호출 !" << std::endl;
	string_length = str.string_length;
	string_content = str.string_content;
	memory_capacity = str.memory_capacity;
	
	// 임시 객체 소멸 시에 메모리를 해제하지
	// 못하게 한다.
	str.string_content = nullptr;
}
MyString::~MyString() {
	if (string_content) delete[] string_content;
}

MyString &MyString::operator=(const MyString &s) {
	std::cout << "복사!" << std::endl;
	if (s.string_length > memory_capacity) {
		delete[] string_content;
		string_content = new char[s.string_length];
		memory_capacity = s.string_length;
	}
	string_length = s.string_length;
	for (int i = 0; i != string_length; i++) {
		string_content[i] = s.string_content[i];
	}

	return *this;
}
int MyString::length() const { return string_length; }
void MyString::println() {
	for (int i = 0; i != string_length; i++) std::cout << string_content[i];

	std::cout << std::endl;
}
template <typename T>
void my_swap(T &a, T &b) {
	T tmp(a);
	a = b;
	b = tmp;
}

int main() {
	MyString str1("abc");
	MyString str2("def");
	std::cout << "Swap 전 -----" << std::endl;
	str1.println();
	str2.println();
	
	std::cout << "Swap 후 -----" << std::endl;
	my_swap(str1, str2);
	str1.println();
	str2.println();
}
```
성공적으로 컴파일했다면
```
생성자 호출 ! 
생성자 호출 ! 
Swap 전 -----
abc
def
Swap 후 -----
복사 생성자 호출 ! 
복사!
복사!
def
abc
```

```cpp
template <typename T>
void my_swap(T &a, T &b) {
	T tmp(a);
	a = b;
	b = tmp;
}
```
위 `my_swap` 함수를 살펴보자. 일단, 첫번째 줄에서, `a`가 좌측값이기 때문에 `tmp`의 복사 생성자가 호출된다. 따라서 1차적으로 `a`가 차지하는 공간만큼 메모리 할당이 발생한 후 `a`의 데이터가 복사된다.
```cpp
a = b;
```
두 번째로 `a = b;` 에서 2 차적으로 복사가 발생한다. 그리고 마지막으로,
```cpp
b = tmp;
```
에서 또 한번 문자열 전체의 복사가 이루어지게 된다. 무려 `swap` 을 하기 위해 문자열 전체 복사를 3번이나 해야 한다. 아래 그림처럼 말이다.
![[9960ED4D5AB96B7D09D8A7.webp]]
하지만 우리는 굳이 문자열 내용을 복사할 필요 없이 각 `MyString` 객체의 `string_content` 주소값만 서로 바꿔주면 되는 것을 알고 있다. (물론 `string_length` 와 `memory_capacity` 도 바꿔야겠지만, 이들은 단순히 4바이트 `int` 복사이기 때문에 속도에 영향을 주지는 않는다).
![[997263435AB96BB9085570.webp]]
하지만 위를 `my_swap`에서 구현하기 위해서는 여러가지 문제가 있다. 일단 첫번째로 `my_swap`함수는 임의의 타입을 받는 함수(Generic) 이다. 다시 말해,
```cpp
template <typename T>
void my_swap(T &a, T &b)
```
위 함수가 일반적인 타입 `T`에 대해 작동해야 한다는 의미이다.
하지만 위 `string_content`의 경우 `MyString`에만 존재하는 필드이기 때문에 일반적인 타입 `T` 에 대해서는 작동하지 않는다. 물론 그렇다고 해서 불가능 한 것은 아니다. 아래처럼 템플릿 특수화를 이용하면 되기 때문이다.

```cpp
template <>
void my_swap(MyString &a, MyString &b) {
	// ...
}
```
문제는 `string_content`가 `private`이기 때문에, 이를 위해 `MyString`내부에 `swap`관련한 함수를 만들어야 된다는 것이다. 사실 이렇게 된다면 굳이 `my_swap`이라는 함수를 정의할 필요가 없게 된다.

위 문제를 원래의 `my_swap` 함수를 사용하면서 좀 더 깔끔하게 해결할 수 있는 방법은 없을까?
```cpp
T tmp(a);
```
먼저 기존의 `my_swap`함수를 다시 살펴보자. 우리는 위 문장이 복사 생성자 대신에, 이동 생성자가 되기를 원한다. 왜냐하면 `tmp`를 복사 생성 할 필요 없이, 단순히 `a`를 잠깐 옮겨놓기만 하면 되기 때문이다. 하지만 문제는 `a` 가 좌측값이라는 점입니다 (`a`라는 실체가 있으므로). 따라서 지금 이 상태로는 우리가 무얼 해도 이동 생성자는 오버로딩 되지 않습니다.

그렇다면, 좌측값이 우측값으로 취급될 수 있게 바꿔주는 함수 같은 것이 있을까요?