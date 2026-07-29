```cpp
#include <iostream>

class MyString {
	char* string_content;  // 문자열 데이터를 가리키는 포인터
	int string_length;     // 문자열 길이
	
	int memory_capacity;
	
public:
	// capacity 만큼 미리 할당함.
	MyString(int capacity);
	
	// 문자열로 부터 생성
	MyString(const char* str);
	
	// 복사 생성자
	MyString(const MyString& str);
	
	~MyString();
	
	int length() const;
};

MyString::MyString(int capacity) {
	string_content = new char[capacity];
	string_length = 0;
	memory_capacity = capacity;
	std::cout << "Capacity : " << capacity << std::endl;
}

MyString::MyString(const char* str) {
	string_length = 0;
	while (str[string_length++]) {
	}
	
	string_content = new char[string_length];
	memory_capacity = string_length;
	
	for (int i = 0; i != string_length; i++) string_content[i] = str[i];
}
MyString::MyString(const MyString& str) {
	string_length = str.string_length;
	string_content = new char[string_length];
	
	for (int i = 0; i != string_length; i++)
		string_content[i] = str.string_content[i];
}
MyString::~MyString() { delete[] string_content; }
int MyString::length() const { return string_length; }

int main() { MyString s(3); }
```
이렇게 MyString이라는 클래스가 있다.

여기서 아래와 같이 MyString을 인자로 받는 함수를 생각해보자.
```cpp
void DoSomethingWithString(MyString s) {
	// Do something...
}
```

그런데 만약 다음과 같이 호출한다면 어떻게 될까?
```cpp
DoSomethingWithString("abc");
```
일단 `DoSomethingWithString` 함수를 살펴보면 인자로 `MyString`을 받고 있다. 하지만 `"abc"`는 `MyString` 타입이 아니다. 그런데 C++ 컴파일러는 `"abc"`를 어떻게 하면 `MyString`으로 바꿀 수 있는지 생각해본다.
`MyString` 생성자들 중에서 다음과 같이 `const char*`로 부터 생성하는 것이 있었다.
```cpp
MyString(const char* str);
```
따라서, `DoSomethingWithString("abc")`는 알아서 `DoSomethingWithString(MyString("abc"))`로 변환되어서 컴파일 된다.

위와 같은 변환을 암시적 변환(implicit conversion)이라고 부른다. 하지만 암시적 변환이 언제나 사용자에게 편리한 것은 아니다.

예를 들어, 다음 코드를 보면
```cpp
DoSomethingWithString(3)
```
이는 높은 확률로 위 함수를 사용자가 잘못 사용했을 가능성이 높다. 왜냐하면 문자열을 받는 함수에 문자열을 전달해야지 정수 데이터를 전달하려는 일은 없기 때문이다. 하지만 컴파일러는 위 문장을 오류로 판단하지 않는다.
왜냐하면,
```cpp
MyString(int capacity);
```
위와 같이 `int` 인자를 받는 `MyString` 생성자가 있기 대문에 위 함수는
```cpp
DoSomethingWithString(MyString(3))
```
으로 변환되어서 컴파일된다. 즉, 사용자가 의도하지 않은 암시적 변환이 일어나게 된다.

하지만 다행히 C++에는 `explicit` 키워드를 통해 원하지 않는 암시적 변환을 할 수 없도록 컴파일러에게 명시할 수 있다.
```cpp
#include <iostream>

class MyString {
	char* string_content;  // 문자열 데이터를 가리키는 포인터
	int string_length;     // 문자열 길이
	
	int memory_capacity;
	
public:
	// capacity 만큼 미리 할당함. (explicit 키워드에 주목)
	explicit MyString(int capacity);
	
	// 문자열로 부터 생성
	MyString(const char* str);
	
	// 복사 생성자
	MyString(const MyString& str);
	
	~MyString();
	
	int length() const;
	int capacity() const;
};

// .. (생략) ..

void DoSomethingWithString(MyString s) {
	// Do something...
}

int main() {
	DoSomethingWithString(3);  // ????
}
```
위 코드는 다음과 같이 `explicit` 키워드를 추가했기 때문에 컴파일 에러가 난다.
```cpp
explicit MyString(int capacity);
```

`explicit`은 또한 해당 생성자가 복사 생성자의 형태로도 호출되는 것을 막게 된다.
예를 들어,
```cpp
MyString s = "abc";
MyString s = 5;
```
`MyString(int capacity)`에 `explicit`이 없을 경우, 위 코드는 잘 동작한다. 왜냐하면 컴파일러가 알아서 적당한 생성자를 골라서 호출하기 때문이다.

하지만 생각해보면
```cpp
MyString s = 5;
```
는 마치 s에 5를 대입하고 있다는 의미를 전달하게 된다. 실제로는 `capacity`를 5로 해주는 것인데도 말이다.
따라서,  `explicit`으로 `MyString(int capacity)`를 설정하면
```cpp
MyString s(5); // 허용
MyString s = 5; // 컴파일 오류
```
위와 같이 명시적으로 생성자를 부를 때만 허용할 수 있게 된다.

