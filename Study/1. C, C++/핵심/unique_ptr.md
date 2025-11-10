## RAII - Resource Acquisition Is Initialization
C++ 창시자인 비야네 스트로스트룹은 C++에서 자원을 관리하는 방법으로 다음과 같은 디자인 패턴을 제안했다.
바로 흔히 RAII라 불리는 '자원의 획득은 초기화다(Resource Acquisition Is Initialization)'이다. 이는 자원 관리를 스택에 할당한 객체를 통해 수행하는 것이다.

함수는 예외가 발생해서 함수가 빠져나가지도라도, 그 함수의 스택에 정의되어 있는 모든 객체들은 빠짐없이 소멸자가 호출된다(stack unwinding).
그럼 소멸자들 안에 다 사용한 자원을 해제하는 루틴을 넣으면 어떨까?

```cpp
class A {
	int *data;

public:
	A() {
		data = new int[100];
		std::cout << "자원을 획득함!" << std::endl;
	}
	
	~A() {
		std::cout << "자원을 해제함!" << std::endl;
		delete[] data;
	}
};

void thrower() {
	// 예외를 발생시킴!
	throw 1;
}

void do_something() {
	A *pa = new A();
	thrower();
	
	// 발생된 예외로 인해 delete pa 가 호출되지 않는다!
	delete pa;
}

int main() {
	try {
		do_something();
	} catch (int i) {
		std::cout << "예외 발생!" << std::endl;
		// 예외 처리
	}
}
```
예를 들어, 위 `pa`의 경우 객체가 아니기 때문에 소멸자가 호출되지 않는다. 이를, `pa`를 일반적인 포인터가 아닌, 포인터 객체로 만들어서 자신이 소멸될 때 자신이 가리키고 있는 데이터도 같이 `delete`하게 하면 관리가 쉽다.

이렇게 작동하는 포인터 객체를 스마트 포인터(smart pointer)라고 한다.

## unique_ptr

C++에서 메모리를 잘못된 방식으로 관리했을 때 크게 2가지 종류의 문제점이 발생할 수 있다.

첫 번째로 메모리를 사용한 후에 해제하지 않은 경우이다(이를 메모리 누수(memory leak)라고 부른다).
간단한 프로그램의 경우 크게 문제될 일이 없지만, 서버처럼 장시간 작동하는 프로그램의 경우 시간이 지남에 따라 점점 사용하는 메모리 양이 늘어나서 결과적으로 나중에 시스템 메모리가 부족해져서 서버가 죽어버리는 상황이 발생할 수도 있다.
다행이 이는 `RAII`패턴을 사용하면 해결할 수 있다.

두 번째로, 이미 해제된 메모리를 다시 참조하는 경우이다. 예를 들어,
```cpp
Data* data = new Data();
Date* data2 = data;

// data 의 입장 : 사용 다 했으니 소멸시켜야지.
delete data;

// ...

// data2 의 입장 : 나도 사용 다 했으니 소멸시켜야지
delete data2;
```
위 경우 `data`와 `data2`가 동시에 한 객체를 가리키고 있는데, `delete data`를 통해 그 객체를 소멸시켜주었다. 그런데, `data2`가 이미 소멸된 객체를 다시 소멸시키려고 한다. 보통 이럴 경우 메모리 오류가 나면서 프로그램이 죽게된다. 이렇게 이미 소멸된 객체를 다시 소멸시켜서 발생하는 버그를 `double free` 버그라고 한다.

이 문제가 발생한 이유는 만들어진 객체의 소유권이 명확하지 않아서이다. 만약, 우리가 어떤 포인터에 객체의 유일한 소유권을 부여해서, '이 포인터 말고는 객체를 소멸시킬 수 없다'라고 한다면, 위와 같이 같은 객체를 2번 소멸시켜버리는 일은 없을 것이다.

C++에서는 특정 객체에 유일한 소유권을 부여하는 포인터 객체를 `unique_ptr`라고 한다.

## unique_ptr 소유권 이전

`unique_ptr`은 복사 생성자가 명시적으로 삭제되었다. `unique_ptr`은 어떠한 객체를 유일하게 소유해야 하기 때문이다. 만약 `unique_ptr`을 복사 생성할 수 있게 된다면, 특정 객체를 여러 개의 `unique_ptr`들이 소유하게 되는 문제가 발생한다. 따라서, 각각의 `unique_ptr`들이 소멸될 때 전부 객체를 `delete`하려 해서 `double free` 버그가 발생하게 된다.

때문에 다음과 같이 move를 통해 소유권을 이전 시켜 주어야 한다.
```cpp
#include <iostream>
#include <memory>

class A {
	int *data;

public:
	A() {
		std::cout << "자원을 획득함!" << std::endl;
		data = new int[100];
	}

	void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

	~A() {
		std::cout << "자원을 해제함!" << std::endl;
		delete[] data;
	}
};

void do_something() {
	std::unique_ptr<A> pa(new A());
	std::cout << "pa : ";
	pa->some();
	
	// pb 에 소유권을 이전.
	std::unique_ptr<A> pb = std::move(pa);
	std::cout << "pb : ";
	pb->some();
}

int main() { do_something(); }
```

## unique_ptr 함수 인자 전달

`unique_ptr`을 함수 인자로 전달할 때 어떻게 하는게 좋은 방법일까?
```cpp
#include <iostream>
#include <memory>

class A {
	int* data;

public:
	A() {
		std::cout << "자원을 획득함!" << std::endl;
		data = new int[100];
	}

	void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

	void do_sth(int a) {
		std::cout << "무언가를 한다!" << std::endl;
		data[0] = a;
	}

	~A() {
		std::cout << "자원을 해제함!" << std::endl;
		delete[] data;
	}
};

// 올바르지 않은 전달 방식
void do_something(std::unique_ptr<A>& ptr) { ptr->do_sth(3); }

int main() {
	std::unique_ptr<A> pa(new A());
	do_something(pa);
}
```
만약 위처럼 레퍼런스를 사용해서 전달한다면, `pa`가 유일하게 소유하고 있던 객체는 이제 적어도 `do_something` 함수 내부에서는 `ptr`을 통해서도 소유할 수 있게 되었다. 즉, `unique_ptr`은 소유권을 의미한다는 원칙에 위배된다.
따라서, `unique_ptr`의 레퍼런스를 사용하는 것은 `unique_ptr`을 소유권이라는 중요한 의미를 망각한 채 단순히 포인터의 `Wrapper`로 사용하는 것에 불과하다.

함수에 올바르게 `unique_ptr`를 전달하려면 레퍼런스 대신, 원래의 포인터 주소값을 전달해 주는 형식으로 사용하는 것이 좋다.
```cpp
#include <iostream>
#include <memory>

class A {
	int* data;

public:
	A() {
		std::cout << "자원을 획득함!" << std::endl;
		data = new int[100];
	}

	void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

	void do_sth(int a) {
		std::cout << "무언가를 한다!" << std::endl;
		data[0] = a;
	}

	~A() {
		std::cout << "자원을 해제함!" << std::endl;
		delete[] data;
	}
};

void do_something(A* ptr) { ptr->do_sth(3); }

int main() {
	std::unique_ptr<A> pa(new A());
	do_something(pa.get());
}
```
`unique_ptr`의 `get` 함수를 호출하면, 실제 객체의 주소값을 반환해준다. 위 경우 `do_something` 함수가 일반적인 포인터를 받고 있다.
이렇게 된다면, 소유권이라는 의미는 버린 채, `do_something` 함수 내부에서 객체에 접근할 수 있는 권한을 주는 것이다.

### 정리
- `unique_ptr`은 어떤 객체의 유일한 소유권을 나타내는 포인터이며, `unique_ptr`이 소멸될 때, 가리키던 객체 역시 소멸된다.
- 만약 다른 함수에서 `unique_ptr`이 소유한 객체에 일시적으로 접근하고 싶다면 `get`을 통해 해당 객체의 포인터를 전달하면 된다.
- 만약 소유권을 이동하고자 한다면, `unique_ptr`을 `move` 하면 된다.

## make_unique

C++14부터 `unique_ptr`을 간단히 만들 수 있는 `std::make_unique` 함수를 제공한다.
```cpp
#include <iostream>
#include <memory>

class Foo {
	int a, b;
	
public:
	Foo(int a, int b) : a(a), b(b) { std::cout << "생성자 호출!" << std::endl; }
	void print() { std::cout << "a : " << a << ", b : " << b << std::endl; }
	~Foo() { std::cout << "소멸자 호출!" << std::endl; }
};

int main() {
	auto ptr = std::make_unique<Foo>(3, 5);
	ptr->print();
}
```

## unique_ptr을 원소로 가지는 컨테이너

`unique_ptr`은 복사 생성자가 없다는 특성 때문에 다음과 같은 코드는 불가하다.
```cpp
#include <iostream>
#include <memory>
#include <vector>

class A {
	int *data;

 public:
	A(int i) {
		std::cout << "자원을 획득함!" << std::endl;
		data = new int[100];
		data[0] = i;
	}

	void some() { std::cout << "일반 포인터와 동일하게 사용가능!" << std::endl; }

	~A() {
		std::cout << "자원을 해제함!" << std::endl;
		delete[] data;
	}
};

int main() {
	std::vector<std::unique_ptr<A>> vec;
	std::unique_ptr<A> pa(new A(1));
	
	vec.push_back(pa);  // error
}
```
당연하게도, `vector`의 `push_back` 함수는 전달된 인자를 복사해서 집어넣기 때문에 에러가 발생하는 것이다.

이를 방지하기 위해서는 명시적으로 `pa`를 `vector` 안으로 이동시켜주어야 한다.
```cpp
int main() {
	std::vector<std::unique_ptr<A>> vec;
	std::unique_ptr<A> pa(new A(1));
	
	vec.push_back(std::move(pa));
}
```

그런데 재밌게도, `emplace_back` 함수를 이용하면, `vector` 안에 `unique_ptr`을 직접 생성하면서 집어넣을 수도 있다. 즉, 불필요한 이동 과정을 생략할 수 있는 것이다.
```cpp
#include <iostream>
#include <memory>
#include <vector>

class A {
	int *data;

 public:
	A(int i) {
		std::cout << "자원을 획득함!" << std::endl;
		data = new int[100];
		data[0] = i;
	}

	void some() { std::cout << "값 : " << data[0] << std::endl; }

	~A() {
		std::cout << "자원을 해제함!" << std::endl;
		delete[] data;
	}
};

int main() {
	std::vector<std::unique_ptr<A>> vec;
	
	// vec.push_back(std::unique_ptr<A>(new A(1))); 과 동일
	vec.emplace_back(new A(1));
	
	vec.back()->some();
}
```

`emplace_back` 함수는 전달된 인자를 완벽한 전달(perfect forwarding)을 통해, 직접  `unique_ptr<A>`의 생성자에 전달해서, `vector` 맨 뒤에 `unique_ptr<A>` 객체를 생성해버리게 된다.
따라서, 불필요한 이동 연산이 필요 없게 된다.

참고로 `emplace_back` 사용 시, 어떤 생성자가 호출되는지 주의해서 사용해야 한다.
예를 들어,
```cpp
std::vector<int> v;
v.emplace_back(100000);
```
을 하게 되면, 100000이란 `int` 값을 `v`에 추가하게 되지만,
```cpp
std::vector<std::vector<int>> v;
v.emplace_back(100000);
```
을 하게 되면 원소가 100000개 들어있는 `vector`를 `v`에 추가하게 된다.