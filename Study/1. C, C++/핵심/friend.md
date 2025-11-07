`friend` 키워드는 클래스 내부에서 다른 클래스나 함수들을 friend로 정의할 수 있는데, `friend`로 정의된 클래스나 함수들은 원래 클래스의 `private`로 정의된 변수나 함수들에 접근할 수 있다.

```cpp
class A {
private:
	void privateFunc() {}
	int privateValue;
	
	friend class B; // friend B
	friend void func(); // friend func
};

class B {
public:
	void b() {
		A a;
		a.privateFunc(); // 접근 가능
		a.privateValue = 1; // 접근 가능
	}
};

void func() {
	A a;
	
	a.privateFunc(); // 접근 가능
	a.privateValue = 2; // 접근 가능
}
```
다만, 여기서 class `A`는 `B`에 대한 private 개체들에 접근할 수 없다.

## friend 활용

```cpp
class Test {
private:
	int _value;
public:
	Test(const char* str);
	Test operator+(const char* str) const;
}

Test::Test(const char* str) {

}
```