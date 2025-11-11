## Callable

Callable이란, 이름 그대로 나타내듯이 호출할 수 있는 모든 것을 의미한다. 대표적인 예시로 함수를 들 수 있다.
C++에서는 `()` 를 붙여서 호출할 수 있는 모든 것을 Callable이라고 정의한다.
```cpp
#include <iostream>

struct S {
	void operator()(int a, int b) { std:: cout << "a + b = " << a + b << std::endl; }
};

int main() {
	S some_obj;
	
	some_obj(3, 5);
}
```
그렇다면 여기서 some_obj는 함수일까? 아니다. 하지만 `same_obj` 클래스 S의 객체이다. 하지만, `same_obj`는 마치 함수처럼 `()`를 이용해서 호출할 수 있다. (실제로는 `same_obj.operator()(3, 5)`)를 한 것이다.

또 다른 예시로 람다 함수를 생각해보자.
```cpp
#include <iostream>

int main() {
	auto f = [](int a, int b) { std::cout << "a + b = " << a + b << std::endl; }
	f(3, 5);
}
```
`f` 역시 일반적인 함수의 꼴을 하고 있지는 않지만, `()`를 통해서 호출할 수 있기에