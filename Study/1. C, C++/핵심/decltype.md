## decltype

`decltype` 키워드는 C++11에 추가된 키워드로, `decltype`라는 이름의 함수처럼 사용된다.
```cpp
decltype(/* 타입을 알고자 하는 식 */)
```
이 때, `decltype`은 함수와는 달리, 타입을 알고자 하는 식의 타입으로 치환되기 된다. 예를 들어,
```cpp
#include <iostream>

struct A {
	double d;
};

int main() {
	int a = 3;
	decltype(a) b = 2;  // int
	
	int& r_a = a;
	decltype(r_a) r_b = b;  // int&
	
	int&& x = 3;
	decltype(x) y = 2;  // int&&
	
	A* aa;
	decltype(aa->d) dd = 0.1;  // double
}
```
위 코드의 경우 `decltype`이 각각 `int`, `int&`, `int&&`로 치환돼서 컴파일 되게 된다.
위와 같이 `decltype`에 전달된 식이 **괄호로 둘러쌓이지 않은 식별자 표현식(id-expression)** 이라면 해당 식의 타입을 얻을 수 있다.

참고로 식별자 표현식이란, 변수의 이름, 함수의 이름, `enum` 이름, 클래스 멤버 변수 (`a.b`나 `a->b` 같은 꼴) 등을 의미한다. 엄밀한 정의는 [여기](https://eel.is/c++draft/expr.prim.id#2.2)에서 볼 수 있는데, 쉽게 생각하면 어떠한 연산을 하지 않고 단순히 객체 하나만을 가리키는 식이라고 보면 된다.

그렇다면 만약에 `decltype`에 식별자 표현식이 아닌 식을 전달하면 어떻게 될까?
이는 해당 식의 값의 종류(value category)에 따라 달라진다.
- 만일 식의 값 종류가 xvalue라면 `decltype`은 `T&&`가 된다.
- 만일 식의 값 종류가 lvalue라면 `decltype`은 `T&`가 된다.
- 만일 식의 값 종류가 prvalue라면 `decltype`은 `T`가 된다.

### Value Category
사람의 경우 이름과 나이라는 정보가 항상 따라다니듯이, 모든 C++ 식(expression)에는 두 가지 정보가 항상 따라다닌다. 바로 식의 **타입**과 **값 카테고리(value category)** 다.

값 카테고리는 좌측값/우측값 등을 일겉는다. C++에서는 총 5가지의 값 카테고리가 존재한다.

C++에서 어떤 식의 값 카테고리를 따질 때 크게 2가지 질문을 던질 수 있다.
- **정체를 알 수 있는가?**
	- 정체를 알 수 있다는 말은 '해당 식이 어떤 다른 식과 같은 것인지 아닌지를 구분할 수 있다'는 말이다. 일반적인 변수라면 주소값을 취해서 구분할 수 있고, 함수의 경우라면 그냥 이름만 확인해보면 될 것이다.
- **이동 시킬 수 있는가?**
	- 해당 식을 다른 곳으로 안전하게 이동할 수 있는지의 여부를 묻는다. 즉, 해당 식을 받는 이동 생성자, 이동 대입 연산자 등을 사용할 수 있어야만 한다.

이를 바탕으로 값 카테고리를 구분해보면 아래 표와 같다.

|                | **이동 시킬 수 있다** | **이동 시킬 수 없다** |
| -------------- | -------------- | -------------- |
| **정체를 알 수 있다** | xvalue         | lvalue         |
| **정체를 알 수 없다** | prvalue        | 쓸모 없음          |
덧붙여서 정체를 알 수 있는 모든 식들을 glvalue라고 하며, 이동 시킬 수 있는 모든 식들을 rvalue라고 한다. 그리고 C++에서 실체도 없으면서 이동도 시킬 수 없는 애들은 어차피 언어 상 아무런 의미를 갖지 않기 대문에 따로 부르는 명칭은 없다.

> [!info]
> - lvalue: 좌측값
> - xvalue(eXpiring value): 소멸하는 값
> - prvalue(pure rvalue): 순수 우측값
> - glvalue(generalized lvalue) 일반화된 좌측값
> - rvalue: 우측값

![[Pasted image 20251111222844.png]]

#### lvalue
예를 들어, 평범한 `int` 타입 변수 `i`를 생각해보자.
```cpp
int i;
i;
```
그리고 우리가 `i`라는 식을 썼을 때, 이 식의 정체를 알 수 있나? 어떤 다른 식 `j`라는 것이 있을 때 구분할 수 있나? 물론이다.
`i`라는 식의 주소값은 실제 변수 `i`의 주소값이 될 것이다. 그렇다면 `i`는 이동 가능한가? 아니다.
`int&& x = i;`는 컴파일되지 않는다. 따라서 `i`는 lvalue이다.

**이름을 가진** 대부분의 객체들은 모두 lvalue이다. 왜냐하면 해당 객체의 주소값을 취할 수 있기 때문이다.
lvalue 카테고리 안에 들어가는 식들을 나열해보자면
- 변수, 함수의 이름, 어떤 타입의 데이터 멤버(예컨대 `std::endl`, `std::cin`) 등
- 좌측값 레퍼런스를 반환하는 함수의 호출식. `std::cout << 1`이나 `++it` 같은 것들
- `a = b`, `a += b`, `a *= b` 같이 복합 대입 연산자 식들
- `++a`, `--a`같은 전위 증감 연산자 식들
- `a.m`, `p->m`과 같이 멤버를 참조할 때. 이 때 `m`은 `enum` 값이거나 `static`이 아닌 멤버 변수인 경우 제외
```cpp
  class A {
  int f();
  static int g();
  };
  
  A a;
  a.g; // <-- lvalue
  a.f; // <-- lvalue 아님 (prvalue)
  ```
- `a[n]`과 같은 배열 참조 식들
- 문자열 리터럴 `"hi"`
등등을 볼 수 있다. 특히 이 lvalue 들은 주소값 연산자(`&`)를 통해 해당 식의 주소값을 알아 낼 수 있다. 예를 들어 `&++i`나 `&std::endl`은 모두 올바른 작업이다. 또한 lvalue 들은 좌측값 레퍼런스를 초기화하는 데 사용할 수 있다.

그럼 여기서 `a`는 무슨 값 카테고리일까?
```cpp
void f(int&& a) {
	a;  // <-- ?
}

f(3);
```
`a`는 우측값 레퍼런스기는 하지만, 식 `a`의 경우는 lvalue이다. 이름이 있기 때문이다.
식 `a`의 타입은 우측값 레퍼런스가 맞지만, 식 `a`의 값 카테고리는 lvalue가 된다. 따라서 아래 같은 식들은 모두 컴파일 된다.
```cpp
#include <iostream>

void f(int&& a) { std::cout << &a; }
int main() { f(3); }
```
만약 `a`가 우측값 레퍼런스니까 `a`는 우측값일꺼야 라고 생각했다면, 타입과 값 카테고리가 다른 개념이란 사실을 헷갈린 경우이다.

#### prvalue
```cpp
int f() { return 10; }

f();  // <-- ?
```
그렇다면 위 코드의 `f()`를 살펴보자. 위 식은 어떤 카테고리에 들어갈까?
먼저 `f()`는 실체가 있을까? 쉽게 생각해서 `f()`의 주소값을 취할 수 있을까? 아니다.
하지만 `f()`는 우측값 레퍼런스에 붙을 수 있다. 따라서 `f()`는 prvalue이다.

prvalue로 대표적인 것들은 아래와 같다.
- 문자열 리터럴을 제외한 모든 리터럴들. `42`, `true`, `nullptr` 등
- 레퍼런스가 아닌 것을 반환하는 함수의 호출 식. 예를 들어 `str.substr(1, 2`, `str1 + str2`
- 후위 증감 연산자 식. `a++`, `a--`
- 산술 연산자, 논리 연산자 식들. `a + b`, `a && b`, `a < b`같은 것들. 물론 이들은 연산자 오버로딩 된 경우들 말고 디폴트로 제공되는 것들을 말한다.
- 주소값 연산자 식 `&a`
- `a.m`, `p->m`과 같이 멤버를 참조할 때. 이 때 `m`은 `enum` 값이거나 `static`이 아닌 멤버 변수
- `this`
- `enum`
- 람다식 `[]() { return 0; }` 등

이 prvalue들은 정체를 알 수 없는 녀석들이기 때문에 주소값을 취할 수 없다. 따라서 `&a++`이나 `&42`와 같은 문장은 모두 오류이다.
또한, prvalue들은 식의 좌측에 올 수 없다. 하지만 prvalue는 우측값 레퍼런스와 상수 좌측값 레퍼런스를 초기화하는 데 사용할 수 있다. 예를 들어,
```cpp
const int& r = 42;
int&& rr = 42;
// int& rrr = 42; <-- 불가능
```
와 같이 된다.
#### xvalue
만일 값 카테고리가 lvalue와 prvalue 두 개로만 구분된다면 문제가 있다. 좌측값으로 분류되는 식을 이동시킬 방법이 없기 때문이다.
따라서 우리는 좌측값처럼 정체가 있지만 이동도 시킬 수 있는 것들을 생각해봐야 한다.

C++에서 이러한 형태의 값의 카테고리에 들어가는 식들로 가장 크게 우측값 레퍼런스를 반환하는 함수의 호출식을 들 수 있다. 대표적으로 `std::move(x)`가 있다.
```cpp
template <class T>
constexpr typename std::remove_reference<T>::type&& move(T&& t) noexcept;
```
다른 복잡한 것들은 모두 건너 뒤더라도 `move`의 반환 타입 만큼은 우측값 레퍼런스임을 알 수 있다.
따라서 `move`를 호출한 식은 lvalue처럼 좌측값 레퍼런스를 초기화하는데 사용할 수도 있고, prvalue처럼 우측값 레퍼런스에 붙이거나 이동 생성자에 전달해서 이동시킬 수 있다.

> [!info]
> 값 카테고리에 대한 자세한 설명은 [cppreference](https://en.cppreference.com/w/cpp/language/value_category.html),  [C++ 표준](https://eel.is/c++draft/basic.lval) 참고

그렇다면 `decltype`으로 돌아와서,
`decltype`에 식별자 표현식이 아닌 식이 전달된다면, 식의 타입이 `T`라고 할 때 아래와 같은 방식으로 타입을 반환한다고 했다.
- 만약 식의 값 종류가 xvalue라면 `decltype`은 `T&&`가 된다.
- 만약 식의 값 종류가 lvalue라면 `decltype`은 `T&`가 된다.
- 만약 식의 값 종류가 prvalue라면 `decltype`은 `T`가 된다.

그렇다면 아래 코드를 살펴보자.
```cpp
int a, b;
decltype(a + b) c; // c의 타입은?
```
위에서 본 바에 따르면 `a + b`는 prvalue이므로 `a + b` 식의 실제 타입인 `int`로 추론된다. 따라서 위 식은 그냥 `int c;`를 한 것과 똑같다.

그러면 아래 식은 어떨까?
```cpp
int a;
decltype((a)) b; // b의 타입은
```
일단 `(a)`는 식별자 표현식이 아니기 때문에 어느 값 카테고리에 들어가는지 생각해봐야 한다.
쉽게 생각하면 `&(a)`와 같이 주소값 연산자를 적용할 수 있고, 당연히도 이동 불가능이므로 lvalue가 된다. 따라서 `b`는 `int`가 될 것으로 예상된다.

하지만 예상과 다르게 `int&`로 추론된다. 이는 C++에서 괄호의 유무로 인해 무언가 결과가 달라지는 첫 번째 경우로 추측된다.

## decltype의 쓰임새

그렇다면 `decltype`은 왜 쓰이는 것일까?
타입 추론이 필요한 부분에는 그냥 `auto`로도 충분하지 않을까? 예를 들어,
```cpp
int i = 4;
auto j = i; // int j = i;
```
를 할 때나
```cpp
int i = 4;
decltype(i) j = i; // int j = i;
```
는 같기 때문이다.
하지만 `auto`는 엄밀히 말하자면 정확한 타입을 표현하지 않는다. 예를 들어,
```cpp
const int i = 4;
auto j = i; // int j = i;
decltype(i) k = i; // const int k = i;
```
`auto`의 경우 `const`를 떼버리지만, `decltype`의 경우 이를 그대로 보존한다. 그 외에도 배열의 경우 **`auto`는 암시적으로 포인터로 변환**하지만, **`decltype`의 경우 배열 타입 그대로를 전달**할 수 있다. 예를 들어,
```cpp
int arr[10];
auto arr2 = arr; // int* arr2 = arr;
decltype(arr) arr3; // int arr3[10];
```
이 될 것이다. 즉, `decltype`을 이용하면 타입 그대로 정확하게 전달할 수 있다.

물론 이 뿐만 아니라, 템플릿 함수에서 어떤 객체 타입이 템플릿 인자들에 의해서 결정되는 경우가 있다. 예를 들어, 아래와 같은 함수를 생각해보자.
```cpp
template <typename T, typename U>
void add(T t, U u, /* 무슨 타입이 와야 할까? */ result) {
  *result = t + u;
}
```
위 `add()` 함수는 단순히 `t`와 `u`를 더해서 `result`에 저장하는 함수이다. 문제는 이 `result`의 타입이 `t + u`의 결과에 의해 결정된다는 사실이다. 예를 들어, `t`가 `double`이고 `u`가 `int`라면 `result`의 타입은 `double*`이 될 것이다.

따라서 이런 경우에 `result`에 타입이 올 자리에 `decltype`을 사용해주면 된다.
```cpp
template <typename T, typename U>
void add(T t, U u, decltype(t + u)* result) {
  *result = t + u;
}
```

그렇다면 위 함수를 살짝 바꿔서 `result`에 전달하는 대신에 그냥 더한 값을 반환해버리는 함수를 만들어보면 어떨까? 만약에 그냥
```cpp
template <typename T, typename U>
decltype(t + u) add(T t, U u) {
  return t + u;
}
```
위와 같이 한다면 한 가지 문제가 있다. 먼저 위 식을 컴파일 해보면 아래와 같이 오류가 발생하는 것을 볼 수 있다.
![[Pasted image 20251114003526.png]]
컴파일러가 위 식을 컴파일할 때 `decltype` 안의 `t`와 `u`를 보고 판단하지 못한 것이다. `t`와 `u`의 정의가 `decltype` 나중에 나오기 때문이다.
이 경우 함수의 반환값을 인자들 정의 부분 뒤에 써야 한다. 이는 C++14 부터 추가된 아래와 같은 문법으로 구현이 가능하다.
```cpp
templat <typename T, typename U>
auto add(T t, U u) -> decltype(t + u) {
	return t + u;
}
```
바로 반환값 자리에는 그냥 `auto`라고 써놓고, `->` 뒤에 함수의 실제 반환 타입을 지정해주는 것이다.
이는 람다 함수 문법과 매우 유사하다.

## std::declval

`declval`은 C++11에 새로 추가된 문법으로, `decltype`과는 다르게 키워드가 아닌 `<utility>`에 정의된 함수이다.

예를 들어, 어떤 타입 `T`의 `f()`라는 함수의 반환 타입을 정의하고 싶다고 해보자. 그러면 `decltype`을 이용하면 아래와 같은 코드를 작성할 수 있다.
```cpp
struct A {
	int f() { return 0; }
};

decltype(A().f()) ret_val;  // int ret_val; 이 된다.
```
참고로 위 과정에서 실제로 `A`의 객체가 생성되거나 함수 `f()`가 호출되거나 그러지는 않는다.
**`decltype`안에 들어가는 식은 그냥 식의 형태로만 존재할 뿐, 컴파일 시에 `decltype` 전체 식이 타입으로 변환되기 때문에 `decltype` 안에 있는 식은 런타임 시에 실행되는 것이 아니다**.

물론 그렇다고 해서 `decltype` 안에 문법상 틀린 식을 전달할 수 있는 것은 아니다. 예를 들어, 어떤 클래스에서 디폴트 생성자가 없다고 해보자.
```cpp
struct B {
	B(int x) {}
	int f() { return 0; }
};

int main() {
	decltype(B().f()) ret_val;  // B() 는 문법상 틀린 문장 :(
}
```
컴파일 한다면 에러가 발생할 것이다.

왜냐하면 `B()`에 해당하는 생성자가 존재하지 않기 때문이다. 우리는 그냥 `B`의 멤버 함수 `f()`의 타입 참조만 필요할 뿐인데, 실제 `B` 객체를 생성할 것도 아닌데도 `B`의 생성자 규칙에 맞는 코드를 작성해야 한다.

물론 우리는 그냥 `B(1)`과 같이 쓰면 된다는 것을 알고 있다. 하지만 다음과 같은 상황을 생각해보자.
```cpp
template <typename T>
decltype(T().f()) call_f_and_return(T& t) {
	return t.f();
}
```
위 함수는 어떤 임의의 타입 `T`의 객체를 받아서 해당 객체의 멤버함수 `f()`를 호출해주는 함수이다. 이 함수를 이용하는 객체들에 멤버 함수 `f()`가 정의되어 있다고 가정한다면, 모두 이용할 수 있다.

문제는 모든 타입 `T` 들이 디폴트 생성자 `T()`를 정의하고 있지 않을 수도 있다는 말이다.
```cpp
template <typename T>
decltype(T().f()) call_f_and_return(T& t) {
	return t.f();
}
struct A {
	int f() { return 0; }
};
struct B {
	B(int x) {}
	int f() { return 0; }
};

int main() {
	A a;
	B b(1);
	
	call_f_and_return(a);  // ok
	call_f_and_return(b);  // BAD
}
```
위 코드를 컴파일해보면 에러가 발생할 것이다.

위 경우 `call_f_and_return()` 함수에 `a`를 전달했을 때는 `A`에 디폴트 생성자가 있으므로 잘 컴파일되지만, `b`를 전달할 때는 `B`에 디폴트 생성자가 없으므로 오류가 발생하게 된다.

따라서 위처럼 직접 생성자를 사용하는 방식은 전달되는 타입들의 생성자가 모두 같은 꼴이지 않을 경우 문제가 생긴다.

이 문제는 `std::declval`을 사용하면 깔끔하게 해결할 수 있다.
```cpp
#include <utility>

template <typename T>
decltype(std::declval<T>().f()) call_f_and_return(T& t) {
	return t.f();
}
struct A {
	int f() { return 0; }
};
struct B {
	B(int x) {}
	int f() { return 0; }
};

int main() {
	A a;
	B b(1);
	
	call_f_and_return(a);  // ok
	call_f_and_return(b);  // ok
}
```
위 코드는 잘 컴파일 된다.

`std::declval`에 타입 `T`를 전달하면, `T`의 생성자를 직접 호출하지 않더라도 `T`가 생성된 객체를 나타낼 수 있다. 즉,
```cpp
std::declval<T>()
```
를 통해 심지어 `T`에 생성자가 존재하지 않더라도 마치 `T()`를 한 것과 같은 효과를 낼 수 있다. 따라서 앞서 발생했던 생성자의 형태가 모두 달라서 발생하는 오류를 막을 수 있다.

참고로 `declval` 함수를 타입 연산에서만 사용해야지, 실제로 런타임에 사용하면 오류가 발생한다.
```cpp
#include <utility>

struct B {
	B(int x) {}
	int f() { return 0; }
};

int main() { B b = std::declval<B>(); }
```
위 코드를 컴파일 해보면 다음과 같이 컴파일 에러가 발생한다.
![[Pasted image 20251114004550.png]]

C++14 부터는 함수의 반환 타입을 컴파일러가 알아서 유추해주는 기능이 추가되었다. 이 경우 그냥 함수 반환 타입을 `auto`로 지정해주면 된다.
```cpp
template <typename T>
auto call_f_and_return(T& t) {
	return t.f();
}
```

물론 그렇다고 해서 `declval`의 쓰임새가 없어지는 것은 아니다.
`decltype`과 `declval`을 활용한 템플릿 메타프로그래밍 기법에서 사용할 수 있다.