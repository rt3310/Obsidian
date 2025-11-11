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
`f` 역시 일반적인 함수의 꼴을 하고 있지는 않지만, `()`를 통해서 호출할 수 있기에 Callable이라 할 수 있다.

## function

C++에서는 이러한 Callable들을 객체의 형태로 보관할 수 있는 `std::function`이라는 클래스를 제공한다.
C 에서의 함수 포인터는 진짜 함수들만 보관할 수 있는 객체라고 볼 수 있다면, 이 `std::function`의 경우 함수 뿐만이 아니라 모든 Callable들을 보관할 수 있는 객체이다.

```cpp
#include <functional>
#include <iostream>
#include <string>

int some_func1(const std::string& a) {
	std::cout << "Func1 호출! " << a << std::endl;
	return 0;
}

struct S {
	void operator()(char c) { std::cout << "Func2 호출! " << c << std::endl; }
};

int main() {
	std::function<int(const std::string&)> f1 = some_func1;
	std::function<void(char)> f2 = S();
	std::function<void()> f3 = []() { std::cout << "Func3 호출! " << std::endl; };
	
	f1("hello");
	f2('c');
	f3();
}
```

일단 `function` 객체를 정의하는 부분부터 살펴보자. `function` 객체는 템플릿 인자로 전달 받을 함수의 타입을 갖게 된다. 여기서 함수의 타입이라 하면, 반환값과 함수의 인자들을 말한다.
```cpp
std::function<int(const string&)> f1 = some_func1;
std::function<void(char)> f2 = S();
std::function<void()> f3 = []() { std::cout << "Func3 호출! " << std::endl; };
```

예를 들어, `some_func1`의 경우 `int`를 반환하며, 인자로 `const string&`을 받기 때문에 위와 같이 `std::function<int(const string&)>`의 형태로 정의된다.

한편 `Functor`인 클래스 S의 객체의 경우 단순히 `S`의 객체를 전달해도 이를 마치 함수인 양 받게 된다. `S`의 경우 `operator()`가 인자로 `char`를 받고 반환 타입이 `void`이므로 `std::function<void(char)>`의 꼴로 표현할 수 있게 된다.

이렇듯 `std::function`은 C++의 모든 Callable을 마음대로 보관할 수 있는 유용한 객체이다. 만약 함수 포인터로 이를 구현하려고 했다면 `Functor`와 같은 경우를 성공적으로 보관할 수 없었을 것이다.

앞서 `function`은 일반적인 Callable들을 쉽게 보관할 수 있었지만, 멤버 함수들의 경우 얘기가 조금 달라진다. 왜냐하면, 멤버 함수 내에서 `this`의 경우 자신을 호출한 객체를 의미하기 때문에, 만약 멤버 함수를 그냥  `function`에 넣게 된다면 `this`가 무엇인지 알 수 없는 문제가 발생하게 된다.
```cpp
#include <functional>
#include <iostream>
#include <string>

class A {
	int c;

public:
	A(int c) : c(c) {}
	int some_func() { std::cout << "내부 데이터 : " << c << std::endl; }
};

int main() {
	A a(5);
	std::function<int()> f1 = a.some_func;
}
```
위 코드를 컴파일 해보면 컴파일 에러가 나는 것을 확인할 수 있다.
![[Pasted image 20251111185744.png]]
왜냐하면 `f1`을 호출했을 때, 함수 입장에서 자신을 호출하는 객체가 무엇인지 알 길이 없어서 `c`를 참조했을 때 어떤 객체의 `c`인지를 알 수 없기 때문이다.
따라서 이 경우 `f1`에 `a`에 관한 정보도 추가로 전달해야 한다.

그러면 이를 어떻게 할까? 사실 멤버 함수들은 구현 상 자신을 호출한 객체를 인자로 암묵적으로 받고 있었다.
따라서 이를 받는 `function`은 아래와 같은 형태로 나타나야 한다.
```cpp
#include <functional>
#include <iostream>
#include <string>

class A {
int c;

public:
	A(int c) : c(c) {}
	int some_func() {
		std::cout << "비상수 함수: " << ++c << std::endl;
		return c;
	}

	int some_const_function() const {
		std::cout << "상수 함수: " << c << std::endl;
		return c;
	}

	static void st() {}
};

int main() {
	A a(5);
	std::function<int(A&)> f1 = &A::some_func;
	std::function<int(const A&)> f2 = &A::some_const_function;
	
	f1(a);
	f2(a);
}
```

```cpp
std::function<int(A&)> f1 = &A::some_func;
std::function<int(const A&)> f2 = &A::some_const_function;
```
위와 같이 원래 인자에 추가적으로 객체를 받는 인자를 전달해주면 된다. 이 때 상수 함수의 경우 당연히 상수 형태로 인자를 받아야 하고(`const A&`), 반면에 상수 함수가 아닌 경우 단순히 `A&` 형태로 인자를 받으면 된다.

참고로 이전 함수들과는 다르게 `&A::some_func`와 같이 함수의 이름만으로는 그 주소값을 전달할 수 없다. 이는 C++ 언어 규칙 때문에 그런데, 멤버 함수가 아닌 함수들의 경우 함수의 이름이 함수의 주소값으로 암시적 변환이 일어나지만, **멤버 함수의 경우 암시적 변환이 발생하지 않으므로 `&` 연산자를 통해 명시적으로 주소값을 전달해줘야 한다**.

따라서 아래와 같이 호출하고자 하는 객체를 인자로 전달해주면 마치 해당 객체의 멤버 함수를 호출한 것과 같은 효과를 낼 수 있다.
```cpp
f1(a);
f2(a);
```

## mem_fn

예를 들어 `vector`들을 가지는 `vector`가 있을 때, 각각의 `vector` 들의 크기를 `vector`로 만들어주는 코드를 생각해보자.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <vector>
using std::vector;

int main() {
	vector<int> a(1);
	vector<int> b(2);
	vector<int> c(3);
	vector<int> d(4);
	
	vector<vector<int>> container;
	container.push_back(b);
	container.push_back(d);
	container.push_back(a);
	container.push_back(c);
	
	vector<int> size_vec(4);
	std::transform(container.begin(), container.end(), size_vec.begin(),
			&vector<int>::size);
	for (auto itr = size_vec.begin(); itr != size_vec.end(); ++itr) {
		std::cout << "벡터 크기 :: " << *itr << std::endl;
	}
}
```
`transform()` 함수는 `<algorithm>`에 있는 함수인데, 각 원소들에 대해 인자로 전달된 함수를 실행시킨 다음 그 결과를 전달된 컨테이너에 넣어준다. 함수 정의를 살펴보면 아래와 같다.
```cpp
template <class InputIt, class OutputIt, class UnaryOperation>
OutputIt transform(InputIt first1, InputIt last1, OutputIt d_first,
                   UnaryOperation unary_op) {
	while (first1 != last1) {
		*d_first++ = unary_op(*first1);
		first1++;
	}
	return d_first;
}
```
여기서 문제는 해당 함수를 아래와 같이 호출한다는 점이다.
```cpp
*d_first++ = unary_op(*first1);
```
`unary_op` 가 멤버 함수가 아닐 경우 위와 같이 호출해도 괜찮다. 하지만 문제는 `unary_op`가 멤버 함수일 경우이다.

사실 위 코드를 컴파일 하면 아래와 같이 컴파일 에러가 난다.
![[Pasted image 20251111190633.png]]왜 그럴까? 이 역시 전달된 `size()` 함수가 멤버 함수여서 발생하는 문제이다. 위 템플릿에 `&vector<int>::size`가 들어간다면 해당 `unary_op`를 호출하는 부분은 아래와 같이 변환된다.
```cpp
unary_op(*first1);
```
가
```cpp
&vector<int>::size(*first);
```
꼴로 되는데,

멤버 함수의 경우
```cpp
(*first).(*&vector<int>::size);
```
혹은
```cpp
first->(*&vector<int>::size);
```
와 같이 호출해야 하기 때문이다. (이는 C++의 규칙이라 생각하면 된다. 위 컴파일러 오류 메세지를 읽어보자)
따라서 이를 위해서는 제대로 `std::function`으로 변환해서 전달해줘야 한다.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <vector>
using std::vector;

int main() {
	vector<int> a(1);
	vector<int> b(2);
	vector<int> c(3);
	vector<int> d(4);
	
	vector<vector<int>> container;
	container.push_back(a);
	container.push_back(b);
	container.push_back(c);
	container.push_back(d);
	
	std::function<size_t(const vector<int>&)> sz_func = &vector<int>::size;
	
	vector<int> size_vec(4);
	std::transform(container.begin(), container.end(), size_vec.begin(), sz_func);
	for (auto itr = size_vec.begin(); itr != size_vec.end(); ++itr) {
		std::cout << "벡터 크기 :: " << *itr << std::endl;
	}
}
```

하지만 매번 위 처럼 `function` 객체를 따로 만들어서 전달하는 것은 매우 귀찮다. 따라서 C++ 개발자 들은 라이브러리에 위 `function` 객체를 반환해버리는 함수를 추가했다.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <vector>
using std::vector;

int main() {
	vector<int> a(1);
	vector<int> b(2);
	vector<int> c(3);
	vector<int> d(4);
	
	vector<vector<int>> container;
	container.push_back(a);
	container.push_back(b);
	container.push_back(c);
	container.push_back(d);
	
	vector<int> size_vec(4);
	transform(container.begin(), container.end(), size_vec.begin(),
			std::mem_fn(&vector<int>::size));
	for (auto itr = size_vec.begin(); itr != size_vec.end(); ++itr) {
		std::cout << "벡터 크기 :: " << *itr << std::endl;
	}
}
```

> [!warning] 주의 사항
> 참고로 `mem_fn`은 그리 자주 쓰이지는 않는데, 람다 함수로도 동일한 작업을 수행할 수 있기 때문이다.
> 위 코드의 경우 `mem_fn(&vector<int>::size)`  대신에 `[](const auto& v { return v.size() }`를 전달해도 동일한 작업을 수행한다. 