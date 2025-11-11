
아마 C++을 사용하면서 아래와 같은 실수를 한 번쯤 했을 것이다.
```cpp
#include <iostream>

class A {
 public:
  A() { std::cout << "A 의 생성자 호출!" << std::endl; }
};

int main() {
	A a();  // ?
}
```
실행시켜보면 아무것도 출력되지 않는다. 왜 그럴까?

```cpp
A a(); // ?
```
왜냐하면 위 코드가 A의 객체 a를 만든 것이 아니라, A를 리턴하고, 인자를 받지 않는 함수 a를 정의한 것이기 대문이다.
C++의 컴파일러는 함수의 정의처럼 보이는 것들은 모두 함수의 정의로 해석한다.

심지어 아래와 같은 코드는 더 헷갈린다.
```cpp
#include <iostream>

class A {
public:
	A() { std::cout << "A 의 생성자 호출!" << std::endl; }
};

class B {
public:
	B(A a) { std::cout << "B 의 생성자 호출!" << std::endl; }
};

int main() {
	B b(A());  // 뭐가 출력될까요?
}
```
위 코드도 아무것도 출력되지 않는다.

사실 위 코드를 보면 마치 b라는 클래스 B의 객체를 생성하는 것 같아 보이지만, 사실은 A를 반환하고 인자가 없는 함수를 받으며, 반환 타입이 B인 함수 b를 정의한 것이다.

상당히 골치 아픈 일이다. 이러한 문제가 발생하는 것은 ()가 함수의 인자들을 정의하는데도 사용되고, 그냥 일반적인 객체의 생성자를 호출하는데도 사용되기 때문이다.

따라서 C++11에서는 이러한 문제를 해결하기 위해 균일한 초기화(Uniform Initialization)라는 것을 도입했다.

## 균일한 초기화(Uniform Initialization)

균일한 초기화 문법을 사용하기 위해서는 생성자를 호출하기 위해 `()`를 사용하는 대신에 `{}`를 사용하면 끝이다.
```cpp
#include <iostream>

class A {
public:
	A() { std::cout << "A 의 생성자 호출!" << std::endl; }
};

int main() {
	A a{};  // 균일한 초기화!
}
```

```
A 의 생성자 호출!
```
실행시켜보면 위와 같이 제대로 생성자가 호출되었음을 알 수 있다.

중괄호를 이용해서 생성자를 호출하는 문법은 동일하다. 그냥 기존에 `()` 자리를 `{}`로 바꿔주기만 하면 된다.
하지만, `()`를 이용한 생성과 `{}`를 이용한 생성의 경우 한 가지 큰 차이가 있는데, 바로 일부 암시적 타입 변환들을 불허하고 있다는 점이다.

예를 들어, 아래 코드를 살펴보자.
```cpp
#include <iostream>

class A {
public:
	A(int x) { std::cout << "A 의 생성자 호출!" << std::endl; }
};

int main() {
	A a(3.5);  // Narrow-conversion 가능
	A b{3.5};  // Narrow-conversion 불가
}
```
컴파일하면 다음과 같은 오류가 발생한다.
![[Pasted image 20251111211217.png]]

보다시피
```cpp
A a(3.5); // Narrow-conversion 가능
```
위 코드는 성공적으로 컴파일 되었고 `x`에는 3.5가 정수로 캐스팅 된 3이 전달된다. 반면에,
```cpp
A a{3.5}; // Narrow-conversion 불가
```
의 경우 `double`인 3.5를 `int`로 변환할 수 없다는 오류가 발생한다.

그 이유는 중괄호를 이용해서 생성자를 호출하는 경우 아래와 같은 암시적 타입 변환들이 불가능해진다. 이들은 전부 **데이터 손실이 있는(Narrowing) 변환**이다.
- 부동 소수점 타입에서 정수 타입으로의 변환
- `long double`에서 `double` 혹은 `float`으로의 변환, `double`에서 `float`으로의 변환
- 정수 타입에서 부동 소수점 타입으로의 변환
- 자세한 예시들은 [여기](https://en.cppreference.com/w/cpp/language/list_initialization.html)에서 확인할 수 있다.

따라서 `{}`를 사용하게 된다면, 위와 같이 원하지 않는 타입 캐스팅을 방지해서 미연에 오류를 잡아낼 수 있다.

`{}`를 이용한 생성의 또 다른 쓰임새로, 함수 반환 시에 굳이 생성하는 객체의 타입을 다시 명시하지 않아도 된다.
```cpp
#include <iostream>

class A {
public:
	A(int x, double y) { std::cout << "A 생성자 호출" << std::endl; }
};

A func() {
	return {1, 2.3};  // A(1, 2.3) 과 동일
}

int main() { func(); }
```

```
A 생성자 호출
```
실행을 시켜보면 위와 같이 결과가 잘 나온다.
`{}`를 이용해서 생성하지 않았더라면 `A(1, 2.3)`과 같이 클래스를 명시해줘야만 했지만, `{}`를 이용할 경우 컴파일러가 알아서 함수의 반환 타입을 보고 추론해준다.

## 초기화자 리스트(Initializer list)
