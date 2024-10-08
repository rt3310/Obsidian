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