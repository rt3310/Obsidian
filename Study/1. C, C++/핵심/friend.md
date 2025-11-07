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

### 연산자 오버로딩
```cpp
class Test {
private:
	int _value;
public:
	Test(int value);
	Test operator+(const Test& t) const;
	Test operator+(int value) const;
}

Test::Test(int value) {
	_value = value;
};

Test Test::operator+(const Test& t) const {
	Test temp(_value + t._value);
	return temp;
}

Test Test::operator+(int value) const {
	Test temp(value);
	return (*this) + temp;
}

int main() {
	Test test(1);
	test = test + 2;
}
```
위 코드를 살펴보면 `test = test + 2;`를 수행하면 `3`이라는 결과를 얻을 수 있다.

그런데, 다음과 같은 코드는 작동하지 않는다는 것을 알 수 있다.
```cpp
test = 2 + test;
```

자세히 생각해보면, `test = test + 2;`는 `test.operator+(2);`를 수행한 것과 같다. 즉, 다시 말해
어떤 임의의 연산자 `@`에 대해서, `a@b`는
```cpp
*a.operator@(b);
*operator@(a, b);
```
중 가능한 것을 택해서 처리되는데 위 케이스가 실행된 것과 같다.

첫 번째의 `a.operator@(b)`에서의 `operator@`는 클래스 `a`의 멤버 함수로써 사용되는 것이고, `operator@(a, b)`에서의 `operator@`는 클래스 외부에 정의되어 있는 일반적인 함수를 의미하게 된다.

따라서 `test = 2 + test;`를 처리하기 위해 함수를 정의해보면 다음과 같이 할 수 있다.
```cpp
Test operator+(const Test& a, const Test& b) {
	Test temp(a._value, b._value);
}
```

> [!warning] 전역 함수 불가 연산자
> 첨자(`[]`) 연산자, 멤버 접근(`->`) 연산자, 대입(`=`) 연산자, 함수 호출(`()`) 연산자의 경우 멤버 함수로만 존재할 수 있다. 즉, 따로 멤버 함수가 아닌 전역 함수로 뺄 수 없다는 의미이다.

다만 여기서 주의할 점은 `Test operator+(const Test& a, const Test& b)`가 제대로 작동하기 위해서 이 함수가 `a`와 `b`의 멤버 변수`value`에 접근할 수 있어야 한다는 것이다.

따라서 이를 해결하기 위해 이 함수를 `Test` 클래스의 `friend`로 지정하여 해결할 수 있다.
```cpp
class Test {
private:
	int _value;
public:
	Test(int value);
	Test& operator=(const Test& t);
	Test operator+(const Test& t) const;
	Test operator+(int value) const;
	friend Test operator+(const Test& a, const Test& b); // 접근 가능
};

Test::Test(int value) {
	_value = value;
}

Test& Test::operator=(const Test& t) {
	_value = t._value;
	return *this;
}

Test Test::operator+(const Test& t) const {
	Test temp(_value + t._value);
	return temp;
}

Test Test::operator+(int value) const {
	Test temp(value);
	return (*this) + temp;
}

Test operator+(const Test& a, const Test& b) {
	Test temp(a._value, b._value);
	return temp;
}

int main() {
	Test test(1);
	test = test + 2;
}
```

그런데 이제 여기서 또 문제가 발생한다. 다음 코드를 컴파일해보자.
```cpp
int main() {
	Test test(1);
	test = test + test; // error
}
```
그럼 `test = test + test;` 부분에서 컴파일 에러가 발생한다.
때문에 이를 해결하기 위해서는 두 함수 중 하나를 없애야 한다.

통상적으로 자기 자신을 반환하지 않는 이항 연산자들, 예를 들어 위와 같은 `+`, `-`, `*`, `/`들은 모두 외부 함수로 선언하는 것이 원칙이다.
반대로 자기 자신을 반환하는 이항 연산자들, 예를 들어 `+=`, `-=` 같은 애들은 모두 멤버 함수로 선언하는 것이 원칙이다. 따라서 위 코드를 수정해보면 다음과 같이 할 수 있다.
```cpp
class Test {
private:
	int _value;
public:
	Test(int value);
	Test& operator=(const Test& t);
	friend Test operator+(const Test& a, const Test& b); // 접근 가능
};

Test::Test(int value) {
	_value = value;
}

Test& Test::operator=(const Test& t) {
	_value = t._value;
	return *this;
}

Test operator+(const Test& a, const Test& b) {
	Test temp(a._value, b._value);
	return temp;
}

int main() {
	Test test(1);
	test = test + 2;
}
```

### 입출력 연산자 오버로딩
`ostream`에 있는 `<<` 연산자를 살펴보자.

실제로 `iostream` 헤더파일의 내용을 살펴보면 (실제로는 `ostream`에 정의되어 있다. 다만 `iostream`이 `ostream`을 include하고 있다)
```cpp
ostream& operator<<(bool val);
ostream& operator<<(short val);
ostream& operator<<(unsigned short val);
ostream& operator<<(int val);
ostream& operator<<(unsigned int val);
ostream& operator<<(long val);
ostream& operator<<(unsigned long val);
ostream& operator<<(float val);
ostream& operator<<(double val);
ostream& operator<<(long double val);
ostream& operator<<(void* val);
```
와 같이 엄청난 수의 `operator<<`가 정의되어 있는 것을 알 수 있다.

그럼 `Test` 클래스에서 `ostream` 클래스의 연산자 `operator<<`를 자유롭게 사용하게 하고 싶으면 어떻게 할까?
당연하게도, `ostream` 클래스에 `operator<<` 멤버 함수를 새롭게 추가하는 것은 표준 헤더파일의 내용을 수정해야 하기 때문에 불가능하다.

하지만 `ostream` 클래스 객체와 `Test` 객체 두 개를 인자로 받는 전역 `operator<<` 함수를 정의하면 할 수 있다.
```cpp
std::ostream& operator<<(std::ostream& os, const Test& t) {
	os << "( " << t._value << ")";
	return os;
}
```
여기서 `ostream&` 타입을 반환하는 이유는 다음과 같이 연쇄적인 문장을 처리할 수 있게 하기 위해서이다.
```cpp
std::cout << 1 << 2 << std::endl;
```

하지만 위 `operator<<`의 경우 한 가지 문제가 있다. 바로 `t._value`에 접근할 수 없다는 점이다(private 변수이기 때문에).

따라서 이를 해결하기 위해 세 가지 방법을 고려할 수 있다.
1. `_value`를 public으로 변환
2. `Test`에 `print(std::ostream& os`와 같은 멤버 함수를 추가한 뒤, 이를 `operator<<`에서 호출
3. `operator<<`를 `friend`로 지정

여기서 3번째 방법을 선택한다면 위에서 했던 것과 같이 `friend` 키워드를 붙여 해결할 수 있다.
