```cpp
#include <iostream>

class Base {
public:
	Base() { std::cout << "기반 클래스" << std::endl; }

	virtual void what() { std::cout << "기반 클래스의 what()" << std::endl; }
};

class Derived : public Base {
public:
	Derived() : Base() { std::cout << "파생 클래스" << std::endl; }

	void what() { std::cout << "파생 클래스의 what()" << std::endl; }
};

int main() {
	Base p;
	Derived c;
	
	Base* p_c = &c;
	Base* p_p = &p;
	
	std::cout << " == 실제 객체는 Base == " << std::endl;
	p_p->what();
	
	std::cout << " == 실제 객체는 Derived == " << std::endl;
	p_c->what();
	
	return 0;
```
위 코드를 실행시켜보자.
```
기반 클래스
기반 클래스
파생 클래스
 == 실제 객체는 Base == 
기반 클래스의 what()
 == 실제 객체는 Derived == 
파생 클래스의 what()
```
결과를 보면 `p_p`와 `p_c` 모드 `Base` 객체를 가리키는 포인터이지만, 각각 무엇과 결합해 있는지 아는 것처럼 적절한 `what()` 함수를 호출해준 것을 볼 수 있다.

이는 `virtual` 키워드 때문이다.
```cpp
class Base {
public:
	Base() { std::cout << "기반 클래스" << std::endl; }
	
	virtual void what() { std::cout << "기반 클래스의 what()" << std::endl; }
};
```
`virtual` 키워드는 런타임에 어떤 것을 호출할지 정한다. 이렇게 런타임 시에 정해지는 일을 가리켜 **동적 바인딩(dynamic binding)** 이라고 부른다. 즉,
```cpp
p_c->what();
```
에서 `Derived`의 `what()`을 실행할지, `Base`의 `what()`을 실행할지 결정은 런타임에 이루어지게 된다.

덧붙여서, `virtual` 키워드가 붙은 함수를 **가상 함수(virtual function)** 라고 부른다. 이렇게 파생 클래스의 함수가 기반 클래스의 함수를 override 하기 위해서는 두 함수의 꼴이 정확히 같아야 한다.


## virtual 소멸자

클래스의 상속을 사용함으로써 중요하게 처리해야 되는 부분이 있다.
그건 바로 상속 시에 **소멸자를 가상함수로 만들어야 된다는 점**이다.
```cpp
#include <iostream>

class Parent {
public:
	Parent() { std::cout << "Parent 생성자 호출" << std::endl; }
	~Parent() { std::cout << "Parent 소멸자 호출" << std::endl; }
};
class Child : public Parent {
public:
	Child() : Parent() { std::cout << "Child 생성자 호출" << std::endl; }
	~Child() { std::cout << "Child 소멸자 호출" << std::endl; }
};
int main() {
	std::cout << "--- 평범한 Child 만들었을 때 ---" << std::endl;
	{ Child c; }
	std::cout << "--- Parent 포인터로 Child 가리켰을 때 ---" << std::endl;
	{
		Parent *p = new Child();
		delete p;
	}
}
```

```
--- 평범한 Child 만들었을 때 ---
Parent 생성자 호출
Child 생성자 호출
Child 소멸자 호출
Parent 소멸자 호출
--- Parent 포인터로 Child 가리켰을 때 ---
Parent 생성자 호출
Child 생성자 호출
Parent 소멸자 호출
```
`Child` 객체가 소멸되는 것과 같은 순서로 생성자와 소멸자들이 호출되어야만 한다. 그런데 실제로는, `Child` 소멸자가 호출되지 않는다.

소멸자가 호출되지 않는다면 여러가지 문제가 생길 수 있다. 예를 들어, `Child` 객체에서 메모리를 동적으로 할당하고 소멸자에서 해제하는데, 소멸자가 호출 안됐다면 메모리 누수(memory leak)가 생길 것이다.

이를 해결하려면 단순히 `Parent`의 소멸자를 `virtual`로 만들어버리면 된다. `Parent`의 소멸자를 `virtual`로 만들면, `p`가 소멸자를 호출할 때, `Child` 의 소멸자를 성공적으로 호출할 수 있게 된다.
```cpp
#include <iostream>

class Parent {
public:
	Parent() { std::cout << "Parent 생성자 호출" << std::endl; }
	virtual ~Parent() { std::cout << "Parent 소멸자 호출" << std::endl; }
};

class Child : public Parent {
public:
	Child() : Parent() { std::cout << "Child 생성자 호출" << std::endl; }
	~Child() { std::cout << "Child 소멸자 호출" << std::endl; }
};

int main() {
	std::cout << "--- 평범한 Child 만들었을 때 ---" << std::endl;
	{ 
		// 이 {} 를 빠져나가면 c 가 소멸된다.
		Child c; 
	}
	std::cout << "--- Parent 포인터로 Child 가리켰을 때 ---" << std::endl;
	{
		Parent *p = new Child();
		delete p;
	}
}
```

## 가상 함수 구현 원리

'여기서 모든 함수들을 `virtual`로 만들어버리면 안되나?' 라는 생각이 들 수 있다. 
물론 모든 함수를 `virtual`로 만든다고 해서 문제될 것이 없다. 실제로 자바의 경우 모든 함수들이 디폴트로 `virtual` 함수로 선언된다.

그러면 C++에서는 왜 사용자가 직접 `virtual` 키워드를 사용하여 선언하도록 했을까?
그 이유는 가상함수를 사용하게 되면 약간의 오버헤드가 존재하기 때문이다.

이를 이해하기 위해 가상 함수라는 것이 어떻게 구현되는지, 다시 말해 마술과 같은 동적 바인딩이 어떻게 구현되는지 살펴보자.
예를 들어서 다음과 같은 간단한 두 개의 클래스를 생각해보자.
```cpp
class Parent {
public:
	virtual void func1();
	virtual void func2();
};
class Child : public Parent {
public:
	virtual void func1();
	void func3();
};
```
C++ 컴파일러는 가상 함수가 하나라도 존재하는 클래스에 대해서, **가상 함수 테이블(virtual function table, vtable)** 을 만들게 된다. 가상 함수 테이블은 전화번호부라고 생각하면 된다.

함수의 이름과 실제로 어떤 함수가 대응되는지 테이블로 저장하고 있는 것이다.
위 경우 `Parent`와 `Child` 모두 가상 함수를 포함하고 있기 때문에 두 개 다 가상 함수 테이블을 생성하게 된다.
그 결과,
![[Pasted image 20251108010210.png]]
와 같이 구성된다.
가상 함수와 가상 함수가 아닌 함수와의 차이점을 살펴보자면 `Child`의 `func3()` 같이 비가상함수들은 그냥 단순히 특별한 단계를 거치지 않고, `func3()`을 호출하면 직접 실행된다.

하지만, 가상 함수를 호출했을 때는 그 실행 과정이 다르다. 위에서도 보다시피, 가상 함수 테이블을 한단계 더 걸쳐서, 실제로 어떤 함수를 고를지 결정하게 된다. 예를 들어,
```cpp
Parent* p = Parent();
p->func1();
```
을 해보자. 그러면, 컴파일러는
1. `p`가 `Parent`를 가리키는 포인터이니까, `func1()`의 정의를 `Parent` 클래스에서 찾아봐야겠다.
2. `func1()`이 가상함수네? 그렇다면 `func1()`을 직접 실행하는게 아니라, 가상 함수 테이블에서 `func1()`에 해당하는 함수를 실행해야겠다.
그리고 실제로 프로그램 실행 시에, 가상 함수 테이블에서 `func1()`에 해당하는 함수(`Parent::func1()`)을 호출하게 된다.

다음의 경우는 어떨까?
```cpp
Parent* c = Child();
c->func1();
```
위처럼 똑같이 프로그램 실행 시에 가상 함수 테이블에서 `func1()`에 해당하는 함수를 호출하게 되는데, 이번에는 `p`가 실제로는 `Child` 객체를 가리키고 있으므로, `Child` 객체의 가상 함수 테이블을 참조하여, `Child:func1()`을 호출하게 된다. 따라서 성공적으로 `Parent::func1()`을 override할 수 있다.

이와 같이 두 단계에 걸쳐서 함수를 호출함을 통해 소프트웨어적으로 동적 바인딩을 구현할 수 있게 된다. 이러한 이유로 가상 함수를 호출하는 경우, 일반적인 함수보다 약간 더 시간이 오래 걸리게 된다.

물론 이 차이는 극히 미미하지만, 최적화가 매우 중요한 분야에서는 이를 감안할 필요가 있다.

### 가상 함수 테이블(virtual function table, vtable)
가상 함수(virtual function)를 사용하면 같은 타입으로 선언된 객체 포인터 변수라도 가리키고 있는 객체에 따라 오버라이딩 된 함수를 호출한다.
컴파일 시 가상함수가 정의된 클래스가 있다면 가상 함수 테이블(virtual function table, vtable)이 만들어져서 바이너리 'rdata' 영역에 기록되며, 해당 클래스로 만들어진 객체에서 함수를 호출할 때 해당 클래스의 가상함수 테이블을 참조해서 함수를 호출한다.

가상 함수 테이블은 자신의 클래스의 vtable을 가리키는 포인터인 vptr을 가지고 있다.

처음에 객체 생성 시에 vptr을 초기화하고 가상 함수 호출 시 vtable을 참조하여 올바른 함수 포인터를 찾고 호출한다.

### 가상 함수 테이블 장단점
#### 장점
- 유연성: 동일한 인터페이스로 서로 다른 객체를 처리할 수 있어 코드의 유연성이 증가한다.
- 확장성: 새로운 클래스가 추가될 때 기존 코드를 수정할 필요 없이 새로운 클래스만 정의하면 된다.
- 코드 재사용성: 기본 클래스에서 정의한 인터페이스를 재사용할 수 있어 코드 중복을 줄일 수 있다.
- 런타임 다형성: 컴파일 시점이 아니라 실행 시점에 호출할 함수를 결정할 수 있어 동적 행동이 가능하다.
#### 단점
- 성능 오버헤드: 가상 함수 호출 시 vtable을 참조하는 과정에서 약간의 오버헤드가 발생할 수 있다. 특히, 게임처럼 성능이 중요한 애플리케이션에서는 주의해야 한다.
- 복잡성 증가: 다형성과 vtable을 사용하는 구조는 코드의 복잡성을 증가시킬 수 있다. 잘못된 사용이나 이해 부족으로 인해 버그가 발생할 수 있다.
- 메모리 사용 증가: 각 클래스마다 vtable이 필요하고, 각 객체마자 vptr이 필요하기 때문에 메모리 사용량이 증가할 수 있다.
- 디버깅 어려움: 다형성과 가상 함수를 사용하면 디버깅이 어려울 수 있다. 어떤 클래스의 메서드가 호출되는지 추적하기 어려울 수 있다.

## 순수 가상 함수(pure virtual function)와 추상 클래스(abstract class)

```cpp
#include <iostream>

class Animal {
public:
	Animal() {}
	virtual ~Animal() {}
	virtual void speak() = 0;
};

class Dog : public Animal {
public:
	Dog() : Animal() {}
	void speak() override { std::cout << "왈왈" << std::endl; }
};

class Cat : public Animal {
public:
	Cat() : Animal() {}
	void speak() override { std::cout << "야옹야옹" << std::endl; }
};

int main() {
	Animal* dog = new Dog();
	Animal* cat = new Cat();
	
	dog->speak();
	cat->speak();
}
```
위 코드를 보면 다음과 같은 부분을 볼 수 있다.
```cpp
class Animal {
public:
	Animal() {}
	virtual ~Animal() {}
	virtual void speak() = 0;
};
```
여기서 `speak()` 함수는 다른 함수들과 달리, 함수의 몸통이 정의되어 있지 않고 단순히 `= 0;`으로 처리되어 있는 가상 함수이다.

이 함수는 "무엇을 하는지 정의되어 있지 않는 함수"이다. 즉, **반드시 오버라이딩 되어야 하는 함수**이다.

이렇게, 가상 함수에 `= 0;`을 붙여서, 반드시 오버라이딩 되도록 만든 함수를 **순수 가상 함수(pure virtual function)** 이라고 부른다.

당연하게도, 순수 가상 함수는 본체가 없기 때문에, 이 함수를 호출하는 것은 불가능하다. 그래서 `Animal` 객체를 생성하는 것 또한 불가능하다. 왜냐하면,
```cpp
Animal a;
a.speak();
```
이렇게 호출하면 안되게 때문이다. 물론, `speak()` 함수를 호출하는 것을 컴파일러 상에서 금지하면 되지 않냐고 물을 수 있는데, C++ 개발자들은 이러한 방법 대신에 아예 `Animal`의 객체 생성을 금지시키는 것으로 택했다.

따라서 `Animal` 처럼, 순수 가상 함수를 최소 한 개 이상 포함하고 있는 클래스는 객체를 생성할 수 없으며, 인스턴스화 시키기 위해서는 이 클래스를 상속 받는 클래스를 만들어서 모든 순수 가상 함수를 오버라이딩 해주어야만 한다.

이렇게 순수 가상 함수를 최소 한 개 포함하고 있는(반드시 상속되어야 하는) 클래스를 가리켜 **추상 클래스(abstract class)** 라고 부른다.