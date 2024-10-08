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
하지만 위 `string_content`의 경우 `MyString`에만 존재하는 필드이기 때문에 일반적인 타입 `T` 에 대해서는 작동하지 않는다. 물론 그렇다고 해서 불가능한 것은 아니다. 아래처럼 템플릿 특수화를 이용하면 되기 때문이다.
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
먼저 기존의 `my_swap`함수를 다시 살펴보자. 우리는 위 문장이 복사 생성자 대신에, 이동 생성자가 되기를 원한다. 왜냐하면 `tmp`를 복사 생성할 필요 없이, 단순히 `a`를 잠깐 옮겨놓기만 하면 되기 때문이다. 하지만 문제는 `a` 가 좌측값이라는 점이다 (`a`라는 실체가 있으므로). 따라서 지금 이 상태로는 우리가 무얼해도 이동 생성자는 오버로딩 되지 않는다.

그렇다면, 좌측값이 우측값으로 취급될 수 있게 바꿔주는 함수 같은 것이 있을까?

## move 함수 (move semantics)

다행히 C++ 11 부터 [utility](https://modoocode.com/299) 라이브러리에서 좌측값을 우측값으로 바꾸어주는 [move](https://modoocode.com/301) 함수를 제공하고 있다. 아래 예시를 통해서 간단히 사용하는 방법을 살펴보자.
```cpp
#include <iostream>
#include <utility>

class A {
public:
	A() { std::cout << "일반 생성자 호출!" << std::endl; }
	A(const A& a) { std::cout << "복사 생성자 호출!" << std::endl; }
	A(A&& a) { std::cout << "이동 생성자 호출!" << std::endl; }
};

int main() {
	A a;
	
	std::cout << "---------" << std::endl;
	A b(a);
	
	std::cout << "---------" << std::endl;
	A c(std::move(a));
}
```
성공적으로 컴파일했다면
```
일반 생성자 호출!
---------
복사 생성자 호출!
---------
이동 생성자 호출!
```

일단 먼저 `A`클래스에 아래와 같이 3가지 형태의 생성자들을 정의했다.
```cpp
public:
A() { std::cout << "일반 생성자 호출!" << std::endl; }
A(const A& a) { std::cout << "복사 생성자 호출!" << std::endl; }
A(A&& a) { std::cout << "이동 생성자 호출!" << std::endl; }
```
그리고 이들 생성자를 호출하는 부분을 살펴보자.
```cpp
A a;

std::cout << "---------" << std::endl;
A b(a);
```
위 부분에서 일반 생성자와 복사 생성자가 각각 호출되었음을 알 수 있다. 그 이유는 `b(a)`를 했을 때 `a`가 이름이 있는 좌측값이므로 좌측값 레퍼런스가 참조하기 때문이다.

그렇다면 바로 그 다음 줄을 보자
```cpp
A c(std::move(a));
```
놀랍게도 이번에는 이동 생성자가 호출되었다. 그 이유는 [std::move](https://modoocode.com/301) 함수가 인자로 받은 객체를 우측값으로 변환해서 리턴해주기 때문이다. 사실 이름만 보면 무언가 이동 시킬 것 같지만 실제로는 단순한 타입 변환 만 수행할 뿐이다.

> [!note]
> C++ 의 원저자인 Bjarne Stroustroup 은 move 라고 이름을 지은 것을 후회했다고 한다. 정확히 말하면 move 함수는 move 를 수행하지 않고 그냥 우측값으로 캐스팅만 하기 때문이다! 더 적절한 이름은 rvalue 와 같은 것이 되겠다.

> [!warning] 주의 사항
> [std::move](https://modoocode.com/301) 함수는 이동을 수행하지 않는다. 그냥 인자로 받은 객체를 우측값으로 변환할 뿐이다.

[std::move](https://modoocode.com/301) 덕분에 강제적으로 우측값 레퍼런스를 인자로 받는 이동 생성자를 호출할 수 있었다. 그렇다면 이 아이디어를 바탕으로 우리의 `MyString` 에 어떻게 적용할 수 있을지 살펴보자.
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
	
	// 일반적인 대입 연산자
	MyString &operator=(const MyString &s);
	
	// 이동 대입 연산자
	MyString& operator=(MyString&& s);
	
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
	str.string_length = 0;
	str.memory_capacity = 0;
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
MyString& MyString::operator=(MyString&& s) {
	std::cout << "이동!" << std::endl;
	string_content = s.string_content;
	memory_capacity = s.memory_capacity;
	string_length = s.string_length;
	
	s.string_content = nullptr;
	s.memory_capacity = 0;
	s.string_length = 0;
	return *this;
}
int MyString::length() const { return string_length; }
void MyString::println() {
	for (int i = 0; i != string_length; i++) std::cout << string_content[i];
	
	std::cout << std::endl;
}

template <typename T>
void my_swap(T &a, T &b) {
	T tmp(std::move(a));
	a = std::move(b);
	b = std::move(tmp);
}
int main() {
	MyString str1("abc");
	MyString str2("def");
	std::cout << "Swap 전 -----" << std::endl;
	std::cout << "str1 : ";
	str1.println();
	std::cout << "str2 : ";
	str2.println();
	
	std::cout << "Swap 후 -----" << std::endl;
	my_swap(str1, str2);
	std::cout << "str1 : ";
	str1.println();
	std::cout << "str2 : ";
	str2.println();
}
```
성공적으로 컴파일했다면
```
생성자 호출 ! 
생성자 호출 ! 
Swap 전 -----
str1 : abc
str2 : def
Swap 후 -----
이동 생성자 호출 !
이동!
이동!
str1 : def
str2 : abc
```

먼저 우리의 `my_swap` 함수 부터 살펴보자.
```cpp
template <typename T>
void my_swap(T &a, T &b) {
	T tmp(std::move(a));
	a = std::move(b);
	b = std::move(tmp);
}
```
먼저
```cpp
T tmp(std::move(a));
```
를 통해서 `tmp` 라는 임시 객체를 `a` 로 부터 이동 생성했다. 이동 생성이기 때문에 기존에 복사 생성하는 것 보다 훨씬 빠르게 수행된다.
```cpp
a = std::move(b);
b = std::move(tmp);
```
그 다음에 `a`에 `b`를 이동 시켰고, `b`에 다시 `tmp`를 이동시킴으로써 swap을 수행하게 된다. 왜 여기서 일반적인 대입이 아니라 이동이 되는 것이냐면 우리가 아래와 같이 이동 대입 연산자를 정의했기 때문이다.
```cpp
MyString& MyString::operator=(MyString&& s) {
	std::cout << "이동!" << std::endl;
	string_content = s.string_content;
	memory_capacity = s.memory_capacity;
	string_length = s.string_length;
	
	s.string_content = nullptr;
	s.memory_capacity = 0;
	s.string_length = 0;
	return *this;
}
```
이동 대입 연산자 역시 이동 생성자와 비슷하게 매우 간단하다. 전체 문자열을 복사할 필요 없이 그냥 기존의 문자열을 가리키고 있던 `string_content`만 복사하면 되기 때문이다.

여기서 알 수 있는 한 가지 사실은 실제로 데이터가 이동 되는 과정은 위와 같이 정의한 이동 생성자나 이동 대입 연산자를 호출할 때 수행 되는 것이지 [std::move](https://modoocode.com/301) 를 한 시점에서 수행되는 것이 아니라는 점이다.

만일 `MyString& MyString::operator=(MyString&& s)` 를 정의하지 않았더라면 일반적인 대입 연산자가 오버로딩 되서 매우 느린 복사가 수행된다. 실제로 이동 대입 연산자를 지워보고 실행하면
```
생성자 호출 ! 
생성자 호출 ! 
Swap 전 -----
str1 : abc
str2 : def
Swap 후 -----
이동 생성자 호출 !
복사!
복사!
str1 : def
str2 : abc
```
와 같이 그냥 일반적인 복사가 수행된다.

> [!warning] 주의 사항
> 다시 한번 강조하지만 이동 자체는 [std::move](https://modoocode.com/301) 를 실행함으로써 발생하는 것이 아니라 우측값을 받는 함수들이 오버로딩 되면서 수행되는 것이다.

### 퀴즈
예를 들어서 아래와 같은 두 클래스가 있다고 해보자.
```cpp
class A {
public:
	A() { std::cout << "ctor\n"; }
	A(const A& a) { std::cout << "copy ctor\n"; }
	A(A&& a) { std::cout << "move ctor\n"; }
};

class B {
public:
	A a_;
};
```
클래스 `A`의 경우 편의상 어떠한 종류의 생성자가 호출되는지 나타내고 있는 단순한 클래스이고, 클래스 `B`에는 그냥 `A`객체를 보관하고 있다.

만약에 `B`객체를 생성할 때, 이미 생성되어 있는 `A`객체를 `B`객체 안으로 집어 넣고 싶다면 어떻게 해야할까?
```cpp
int main() {
	A a;
	
	std::cout << "create B-- \n";
	B b(/* ?? */);
}
```
예를 들어서 위의 경우 이미 `A`객체인 `a`가 생성되어 있는데, 아래에서 `B`객체인 `b`를 생성하면서 `a`를 `B`에 이동 시켜야 한다.

그렇다면 `B`의 생성자를 어떠한 방식으로 작성해야 할까?

#### 첫 번째 시도
먼저 가장 단순하게 `B`의 생성자를 `const A&` 타입으로 만들어보자.
```cpp
#include <iostream>

class A {
public:
	A() { std::cout << "ctor\n"; }
	A(const A& a) { std::cout << "copy ctor\n"; }
	A(A&& a) { std::cout << "move ctor\n"; }
};

class B {
public:
	B(const A& a) : a_(a) {}
	
	A a_;
};

int main() {
	A a;
	std::cout << "create B-- \n";
	B b(a);
}
```
성공적으로 컴파일했다면
```
ctor
create B-- 
copy ctor
```
예상했던 대로 복사 생성자가 호출되었다. 하지만 우리가 원하던 것은 이게 아니다. 우리는 `main` 안에서 이미 생성되어 있는 `A`라는 객체를 새로 생성된 `b` 안으로 이동시키고 싶을 뿐이다. 그럼 다시 해보자.

#### 두 번째 시도
아 이동 시킬려면 std::move 를 해야 되니까 그냥 `a_(std::move(a))`로 해볼까?
```cpp
#include <iostream>

class A {
 public:
  A() { std::cout << "ctor\n"; }
  A(const A& a) { std::cout << "copy ctor\n"; }
  A(A&& a) { std::cout << "move ctor\n"; }
};

class B {
 public:
  B(const A& a) : a_(std::move(a)) {}

  A a_;
};

int main() {
  A a;
  std::cout << "create B-- \n";
  B b(a);
}
```
성공적으로 컴파일했다면
```
ctor
create B-- 
copy ctor
```
동일한 결과가 나온다. 왜 `std::move(a)`를 해서 우측값으로 바꾸었는데 왜 이동 생성자가 호출이 되지 않고 복사 생성자가 호출이 되었을까?

일단 `std::move(a)`가 우측값으로 바꾸는 것은 맞다. 그런데 문제는 `a`가 `const A&`이므로, `std::move(a)`의 타입은 `const A&&`가 된다는 것이다. 그런데 `A`의 생성자에는 `const A&`와 `A&&`두 개 밖에 없다. 따라서 여기선 컴파일러가 `const A&`를 택하게 된다. 따라서 복사 생성자가 호출이 되는 것이다.

#### 세 번째 시도
아 그렇다면 B 생성자에서 아예 우측값을 받는 수 받게 없구나.
```cpp
#include <iostream>

class A {
public:
	A() { std::cout << "ctor\n"; }
	A(const A& a) { std::cout << "copy ctor\n"; }
	A(A&& a) { std::coaut << "move ctor\n"; }
};

class B {
public:
	B(A&& a) : a_(a) {}
	
	A a_;
};

int main() {
	A a;
	std::cout << "create B-- \n";
	B b(std::move(a));
}
```
성공적으로 컴파일했다면
```
ctor
create B-- 
copy ctor
```
와 같이 나옵니다. 엥? 아니 왜 `a` 를 우측값 레퍼런스로 받았는데, 왜 이동 생성자 대신에 복사 생성자가 호출이 되었을까? 그 이유는 간단하다. 앞서 이야기 했듯이 `a`는 우측값 레퍼런스 이지만 그 자체로는 좌측값이기 때문이다(이름이 있으니까). 따라서 `a`를 다시 한 번 우측값으로 캐스팅 시켜줘야 한다.

#### 정답
```cpp
#include <iostream>

class A {
public:
	A() { std::cout << "ctor\n"; }
	A(const A& a) { std::cout << "copy ctor\n"; }
	A(A&& a) { std::cout << "move ctor\n"; }
};

class B {
public:
	B(A&& a) : a_(std::move(a)) {}
	
	A a_;
};

int main() {
	A a;
	std::cout << "create B-- \n";
	B b(std::move(a));
}
```
성공적으로 컴파일했다면
```
ctor
create B-- 
move ctor
```
와 같이 비로소 제대로 `A`의 이동 생성자가 호출이 되어서 `a`가 `b`의 `a_`로 잘 이동되었음을 알 수 있다.

## 완벽한 전달 (perfect forwarding)

C++ 11에 우측값 레퍼런스가 도입되기 전까지 해결할 수 없었던 문제가 있었다. 예를 들어서 아래와 같은 `wrapper` 함수를 생각해보자.
```cpp
template <typename T>
void wrapper(T u) {
	g(u);
}
```
이 함수는 인자로 받은 `u` 를 그대로 `g` 라는 함수에 인자로 전달해준다. 물론 '왜 저런 함수가 필요하나, 그냥 저런 `wrapper` 함수를 만들지 말고 그냥 `g(u)` 를 호출하면 되지 않나?'라고 생각할 수 있다.

하지만 실제로 저러한 형태의 전달 방식이 사용되는 경우가 종종 있다. 예를 들어 `vector` 에는 `emplace_back`이라는 함수가 있다. 이 함수는 객체의 생성자에 전달하고 싶은 인자들을 함수에 전달하면, 알아서 생성해서 `vector` 맨 뒤에 추가해준다.

예를 들어서 클래스 `A`를 원소로 가지는 벡터의 뒤에 원소를 추가하기 위해서는
```cpp
vec.push_back(A(1, 2, 3));
```
과 같이 객체를 생성한 뒤에 인자로 전달해줘야만 한다. 하지만 이 과정에서 불필요한 이동 혹은 복사가 발생하게 된다. 그 대신, `emplace_back` 함수를 사용하게 되면
```cpp
vec.emplace_back(1, 2, 3);  // 위와 동일한 작업을 수행한다.
```
`emplace_back`함수는 인자를 직접 전달받아서, 내부에서 `A`의 생성자를 호출한 뒤에 이를 벡터 원소 뒤에 추가하게 된다. ~~이 과정에서 불필요한 이동/복사 모두 발생하지 않는다. 참고로 새로 생성한 객체를 벡터 뒤에 추가할 경우 위와 같이 [push_back](https://modoocode.com/185) 을 이용하는 것 보다 `emplace_back` 을 이용하는 것이 권장되는 방식이다.~~ **사실 [push_back](https://modoocode.com/185) 함수를 사용할 경우 컴파일러가 알아서 최적화를 해주기 때문에 불필요한 복사-이동을 수행하지 않고 `emplace_back` 을 사용했을 때와 동일한 어셈블리를 생성한다**. 따라서 [push_back](https://modoocode.com/185) 을 사용하는 것이 훨씬 낫다. (**`emplace_back` 은 예상치 못한 생성자가 호출될 위험이 있다**)

그렇다면 문제는 `emplace_back`함수가 받은 인자들을 `A`의 생성자에 제대로 전달해야 한다는 점이다. 그렇지 않을 경우 사용자가 의도하지 않은 생성자가 호출될 수 있기 때문이다. 그렇다면 위와 같은 `wrapper` 함수를 어떻게 하면 잘 정의할 수 있을까?
```cpp
#include <iostream>
#include <vector>

template <typename T>
void wrapper(T u) {
	g(u);
}

class A {};

void g(A& a) { std::cout << "좌측값 레퍼런스 호출" << std::endl; }
void g(const A& a) { std::cout << "좌측값 상수 레퍼런스 호출" << std::endl; }
void g(A&& a) { std::cout << "우측값 레퍼런스 호출" << std::endl; }

int main() {
	A a;
	const A ca;
	
	std::cout << "원본 --------" << std::endl;
	g(a);
	g(ca);
	g(A());
	
	std::cout << "Wrapper -----" << std::endl;
	wrapper(a);
	wrapper(ca);
	wrapper(A());
}
```
성공적으로 컴파일했다면
```
원본 --------
좌측값 레퍼런스 호출
좌측값 상수 레퍼런스 호출
우측값 레퍼런스 호출
Wrapper -----
좌측값 레퍼런스 호출
좌측값 레퍼런스 호출
좌측값 레퍼런스 호출
```

```cpp
std::cout << "원본 --------" << std::endl;
g(a);
g(ca);
g(A());
```
먼저 위 경우 우리의 예상대로 좌측값 레퍼런스, 좌측값 상수 레퍼런스, 우측값 레퍼런스가 각각 호출되었다. 반면에 `wrapper` 함수를 거쳐갔을 경우, 공교롭게도 세 경우 모두 좌측값 레퍼런스를 받는 `g`함수가 호출되었다.

이러한 일이 발생한 이유는 C++ 컴파일러가 템플릿 타입을 추론할 때, **템플릿 인자 `T` 가 레퍼런스가 아닌 일반적인 타입이라면 `const` 를 무시하기 때문**이다. 다시 말해,
```cpp
template <typename T>
void wrapper(T u) {
	g(u);
}
```
에서 `T` 가 전부 다 `class A` 로 추론된다. 따라서 위 세 경우 전부 다 좌측값 레퍼런스를 호출하는 `g`를 호출했다.
```cpp
template <typename T>
void wrapper(T& u) {
	g(u);
}
```
그렇다면 위 경우는 어떨까?
![[Pasted image 20241008221901.png]]
위와 같은 컴파일 오류가
```cpp
g(A());
```
에서 발생한다. (참고로 이 오류는 `gcc` 와 `clang` 컴파일러에서 모두 발생하는데, 비주얼 스튜디오 2017 이전 버전에서는 발생하지 않는다. 하지만 원칙적으로 위와 같은 오류를 발생시켜야 하는 것이 맞다)

왜 위와 같은 오류가 발생하는지 생각해보자면 다음과 같다. 일단, `A()` 자체는 `const` 속성이 없으므로 템플릿 인자 추론에서 `T` 가 `class A` 로 추론된다. 하지만 `A&` 는 우측값의 레퍼런스가 될 수 없기 때문에 컴파일 오류가 발생하는 것이다.

그렇다면 아예 우측값을 레퍼런스로 받을 수 있도록 `const A&` 와 `A&` 따로 만들어주는 방법이 있다. 아래와 같이 말이다.
```cpp
#include <iostream>
#include <vector>

template <typename T>
void wrapper(T& u) {
	std::cout << "T& 로 추론됨" << std::endl;
	g(u);
}

template <typename T>
void wrapper(const T& u) {
	std::cout << "const T& 로 추론됨" << std::endl;
	g(u);
}

class A {};

void g(A& a) { std::cout << "좌측값 레퍼런스 호출" << std::endl; }
void g(const A& a) { std::cout << "좌측값 상수 레퍼런스 호출" << std::endl; }
void g(A&& a) { std::cout << "우측값 레퍼런스 호출" << std::endl; }

int main() {
	A a;
	const A ca;
	
	std::cout << "원본 --------" << std::endl;
	g(a);
	g(ca);
	g(A());
	
	std::cout << "Wrapper -----" << std::endl;
	wrapper(a);
	wrapper(ca);
	wrapper(A());
}
```
성공적으로 컴파일했다면
```
원본 --------
좌측값 레퍼런스 호출
좌측값 상수 레퍼런스 호출
우측값 레퍼런스 호출
Wrapper -----
T& 로 추론됨
좌측값 레퍼런스 호출
const T& 로 추론됨
좌측값 상수 레퍼런스 호출
const T& 로 추론됨
좌측값 상수 레퍼런스 호출
```
와 같이 나온다.

일단 `a` 와 `ca` 의 경우 각각 `T&`와 `const T&`로 잘 추론되서 올바른 함수를 호출하고 있음을 알 수 있다. 반면에 `A()` 의 경우 `const T&`로 추론되면서 `g(const T&)` 함수를 호출하게 된다. 물론 이는 예상했던 일이다. 우리가 무엇을 해도 `wrapper`안에 `u`가 좌측값이라는 사실은 변하지 않고 이에 언제나 좌측값 레퍼런스를 받는 함수들이 오버로딩 될 것이다.

뿐만이 아니라 다음과 같은 문제가 있다. 예를 들어, 함수 `g`가 인자를 한 개가 아니라 2개를 받는다고 가정하자. 그렇다면 우리는 다음과 같은 모든 조합의 템플릿 함수들을 정의해야한다.
```cpp
template <typename T>
void wrapper(T& u, T& v) {
	g(u, v);
}
template <typename T>
void wrapper(const T& u, T& v) {
	g(u, v);
}

template <typename T>
void wrapper(T& u, const T& v) {
	g(u, v);
}
template <typename T>
void wrapper(const T& u, const T& v) {
	g(u, v);
}
```
매우 귀찮은 일이다. 위와 같이 짜야하는 이유는 단순히 일반적인 레퍼런스가 우측값을 받을 수 없기 때문이다. 그렇다고 해서 디폴트로 상수 레퍼런스만 받게 된다면, 상수가 아닌 레퍼런스도 상수 레퍼런스로 캐스팅돼서 들어간다.

하지만 놀랍게도 C++ 11 에서는 이를 간단하게 해결할 수 있다.

## 보편적 레퍼런스 (Universal reference)

```cpp
#include <iostream>

template <typename T>
void wrapper(T&& u) {
	g(std::forward<T>(u));
}

class A {};

void g(A& a) { std::cout << "좌측값 레퍼런스 호출" << std::endl; }
void g(const A& a) { std::cout << "좌측값 상수 레퍼런스 호출" << std::endl; }
void g(A&& a) { std::cout << "우측값 레퍼런스 호출" << std::endl; }

int main() {
	A a;
	const A ca;
	
	std::cout << "원본 --------" << std::endl;
	g(a);
	g(ca);
	g(A());
	
	std::cout << "Wrapper -----" << std::endl;
	wrapper(a);
	wrapper(ca);
	wrapper(A());
}
```