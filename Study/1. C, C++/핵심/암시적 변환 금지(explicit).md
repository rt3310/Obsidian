```cpp
class Test {
	char* _value1;
	int _value2;
public:
	Test(int value1);
	Test(const char* value2);
	~Test();
};
Test::Test(int value1) {
	_value1 = value1;
	_value2 = new char[10];
}
Test::Test(const char* value2) {
	int length = 0;
	while (value2[length++]) {}
	_value2 = new char[length];
	for (int i = 0; i < length; i++)
		_value2[i] = value2[i];
}
Test::~Test() { delete[] _value2; }
```

위 클래스를 기반으로 다음 코드를 생각해보자.
```cpp
void doSomething(Test t) {
	// Do Something...
}
```

위 함수를 호출하는 다음 코드는 실행될까?
```cpp
doSomething(Test("123"));
```
당연히 된다. 그렇다면 다음은 어떨까?
```cpp
doSomething("123");
```
이것도 된다. C++ 컴파일러는 꽤나 똑똑해서 "123"을 어떻게 하면 `Test`로 바꿀 수 있는지 생각해본다.
다행이 `Test` 생성자 중에는 다음과 같이 `const char*`로 부터 생성하는 것이 있었다.
```cpp
Test(const char* value2);
```
따라서, `doSomething("123")`은 알아서
```cpp
doSomething(Test("123"));
```
로 변환돼서 컴파일 된다. 위와 같은 변환을 **암시적 변환(implicit conversion)** 이라고 부른다. 하지만 암시적 변환이 언제나 사용자에게 편리한 것은 아니다. 때로는 예상치 못한 경우에 암시적 변환이 일어날 수도 있다.

때문에 `explicit` 키워드를 통해 암시적 변환을 할 수 없도록 컴파일러에게 명시할 수 있다. (explicit은 implicit의 반대말로, '명시적'이라는 뜻을 가지고 있다)
```cpp
class Test {
	char* _value1;
	int _value2;
public:
	explicit Test(int value1);
	Test(const char* value2);
	~Test();
};
Test::Test(int value1) {
	_value1 = value1;
	_value2 = new char[10];
}
Test::Test(const char* value2) {
	int length = 0;
	while (value2[length++]) {}
	_value2 = new char[length];
	for (int i = 0; i < length; i++)
		_value2[i] = value2[i];
}
Test::~Test() { delete[] _value2; }
```
이렇게 명시했다면 다음과 같은 코드는 컴파일 에러가 발생한다.
```cpp
doSomething(3);
```