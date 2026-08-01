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
가상 함수(virtual function)를 사용하면 **같은 타입으로 선언된 객체 포인터 변수라도 가리키고 있는 객체에 따라 오버라이딩 된 함수를 호출**한다.
컴파일 시 가상함수가 정의된 클래스가 있다면 가상 함수 테이블(virtual function table, vtable)이 만들어져서 **바이너리 'rdata' 영역**에 기록되며, 해당 클래스로 만들어진 객체에서 함수를 호출할 때 해당 클래스의 가상함수 테이블을 참조해서 함수를 호출한다.

가상 함수 테이블은 **자신의 클래스의 vtable을 가리키는 포인터인 vptr을 가지고 있다**.

처음에 객체 생성 시에 vptr을 초기화하고 가상 함수 호출 시 vtable을 참조하여 올바른 함수 포인터를 찾고 호출한다.

### 가상 함수 테이블 장단점
#### 장점
- 유연성: 동일한 인터페이스로 서로 다른 객체를 처리할 수 있어 코드의 유연성이 증가한다.
- 확장성: 새로운 클래스가 추가될 때 기존 코드를 수정할 필요 없이 새로운 클래스만 정의하면 된다.
- 코드 재사용성: 기본 클래스에서 정의한 인터페이스를 재사용할 수 있어 코드 중복을 줄일 수 있다.
- 런타임 다형성: 컴파일 시점이 아니라 실행 시점에 호출할 함수를 결정할 수 있어 동적 행동이 가능하다.
#### 단점
- 성능 오버헤드: 가상 함수 호출 시 **vtable을 참조하는 과정에서 약간의 오버헤드가 발생할 수 있다**. 특히, 게임처럼 성능이 중요한 애플리케이션에서는 주의해야 한다.
- 복잡성 증가: 다형성과 vtable을 사용하는 구조는 코드의 복잡성을 증가시킬 수 있다. 잘못된 사용이나 이해 부족으로 인해 버그가 발생할 수 있다.
- 메모리 사용 증가: **각 클래스마다 vtable이 필요하고, 각 객체마자 vptr이 필요하기 때문에 메모리 사용량이 증가할 수 있다**.
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

### 다중 상속(multiple inheritance)
C++ 에서는 한 클래스가 다른 여러 개의 클래스들을 상속 받는 것을 허용한다. 이를 가리켜서 다중 상속(multiple inheritance)라고 부른다.

```cpp
class A {
public:
	int a;
};

class B {
public:
	int b;
};

class C : public A, public B {
public:
	int c;
};
```
위 경우, `C`가 `A`와 `B`로 부터 동시에 같이 상속받고 있다.

그럼 여기서 `A`와 `B` 중에 어느 생성자가 먼저 호출될까?
```cpp
#include <iostream>

class A {
public:
	int a;
	
	A() { std::cout << "A 생성자 호출" << std::endl; }
};

class B {
public:
	int b;
	
	B() { std::cout << "B 생성자 호출" << std::endl; }
};

class C : public A, public B {
public:
	int c;
	
	C() : A(), B() { std::cout << "C 생성자 호출" << std::endl; }
};
int main() { C c; }
```
성공적으로 컴파일 했다면
```
A 생성자 호출
B 생성자 호출
C 생성자 호출
```
위처럼 `A → B → C` 순으로 호출됨을 알 수 있다.

그런데 여기서,
```cpp
class C : public B, public A
```
로 바꾸고 컴파일하면
```
B 생성자 호출
A 생성자 호출
C 생성자 호출
```
위와 같이 순서가 바뀌는 것을 볼 수 있다. 따라서 이 순서는 다른 것들에 의해 좌우되지 않고 오직 상속하는 순서에만 좌우됨을 알 수 있다.

### 다중 상속 시 주의할 점
```cpp
class A {
public:
	int a;
};

class B {
public:
	int a;
};

class C : public B, public A {
public:
	int c;
};
```
위처럼 만일 두 개 의 클래스에서 이름이 같은 멤버 변수나 함수가 있다고 가정해보자.
예를 들어, 위 예에서는 클래스 `A`와 `B`에 모두 `a`라는 이름의 멤버 변수가 들어가 있다.
```cpp
int main() {
	C c;
	c.a = 3;
}
```

그렇다면 만약 클래스 `C`의 객체를 생성해서, 위처럼 중복되는 멤버 변수에 접근한다면
```
error C2385: ambiguous access of 'a'
1>          could be the 'a' in base 'B'
1>          or could be the 'a' in base 'A'
```
위와 같이 `B`의 `a`인지 `A`의 `a`인지 구분할 수 없다는 오류를 발생하게 된다.
마찬가지로, 클래스 `A`와 `B`에 같은 이름의 함수가 있다면 똑같이 어떤 함수를 호출해야 될 지 구분할 수 없다.

### 다이아몬드 상속(diamond inheritance)
다중 상속 사용 시 또 한 가지 주의해야 할 점으로 다이아몬드 상속(diamond inheritance) 혹은 공포의 다이아몬드 상속(dreadful diamond of derivation) 이라고 부르는 형태의 다중 상속에 있다.

예를 들어, 다음과 같은 형태의 상속 관계를 생각해보자.
```cpp
class Human {
	// ...
};
class HandsomeHuman : public Human {
	// ...
};
class SmartHuman : public Human {
	// ...
};
class Me : public HandsomeHuman, public SmartHuman {
	// ...
};
```
일단 Base 클래스로 `Human`이라는 클래스가 있고, `HandsomHuman`과 `SmartHuman` 클래스는 `Human` 클래스를 모두 상속받는다.

그리고 두 가지 특성을 모두 보유한 나(Me)라는 클래스는 `HandsomeHuman`과 `SmartHuman` 클래스를 둘 다 상속 받는다. 이를 그림으로 표현하자면 아래와 같은 다이아몬드 모양이 나오게 된다.
![[Pasted image 20260801155109.png]]

상속이 되는 두 개의 클래스가 공통의 베이스 클래스를 포함하고 있는 형태를 가리켜서 다이아몬드 상속이라고 부른다. 이러한 형태의 상속에 문제점은 보이게도 명백하다.

만약 `Human`에 `name`이라는 멤버변수가 있다고 해보자. 그러면 `HandsomeHuman`과 `SmartHuman`은 모두 `Human`을 상속받고 있으므로, 여기에도 `name`이라는 변수가 들어가게 된다.

그런데 `Me`가 이 두 개의 클래스를 상속 받으니 `Me`에서는  `name`이라는 변수가 겹치게 되는 것이다.

결과적으로 볼 때 `Handsome`과 `SmartHuman`을 아무리 안겹치게 만든다 해도, `Human`의 모든 내용이 중복되는 문제가 발생한다.

다행히 이를 해결할 수 있는 방법이 있다.
```cpp
class Human {
public:
	// ...
};
class HandsomeHuman : public virtual Human {
	// ...
};
class SmartHuman : public virtual Human {
	// ...
};
class Me : public HandsomeHuman, public SmartHuman {
	// ...
};
```
이러한 형태로 `Human`을 `virtual`로 상속받는다면, `Me`에서 다중 상속 시에도, 컴파일러가 언제나 `Human`을 한 번만 포함하도록 지정할 수 있게 된다. 참고로, 가상 상속 시에, `Me`의 생성자에서 `HandsomeHuman`과 `SmartHuman`의 생성자를 호출함은 당연하고, `Human`의 생성자 또한 호출해주어야만 한다.

### 다중 상속은 언제 사용해야 할까?
예를 들어, 차량에 관련한 클래스를 생성한다고 해보자. 차량의 종류로는 땅에서 다니는 차, 물에서 다니는 차, 하늘에서 다니는 차, 우주에서 다니는 차들이 있다고 해보자.

또한, 이 차량들은 각기 다른 동력원들을 사용하는데, 휘발유를 사용할 수도 있고, 풍력으로 갈 수도 있고 원자력으로 갈 수도 있고, 페달을 밟아서 갈 수도 있다.

이러한 차량들을 클래스로 나타내기 위해서, 다중 상속을 활용할 수 있지만 그 전에, 아래와 같은 질문들에 대한 대답을 생각해보자.
- `LandVehicle` 을 가리키는 `Vehicle&` 레퍼런스를 필요로 할까? 다시 말해, `Vehicle` 레퍼런스가 실제로는 `LandVehicle` 을 참조하고 있다면, `Vehicle` 의 멤버 함수를 호출하였을 때, `LandVehicle` 의 멤버 함수가 오버라이드 되서 호출되기를 바라는가?
- `GasPoweredVehicle` 의 경우도 마찬가지이다. 만일 `Vehicle` 레퍼런스가 실제로는 `GasPoweredVehicle` 을 참조하고 있을 때, `Vehicle` 레퍼런스의 멤버함수를 호출한다면, `GasPoweredVehicle` 의 멤버 함수가 오버라이드 되서 호출되기를 원하는가?

만일 두 개의 질문에 대한 대답이 모두 예 라면 다중 상속을 사용하는 것이 좋을 것이다. 하지만 그 전에, 몇 가지 고려할 점이 더 있다. 만약 이 차량이 작동하는 환경이 N개가 있고 (땅, 물, 하늘, 우주 등등), 동력원의 종류가 M개가 있다고 해보자.

이를 위해서, 크게 3가지 방법으로 이러한 클래스를 디자인 할 수 있다. 바로 **브리지 패턴(bridge pattern), 중첩된 일반화 방식(nested generalization), 다중 상속**이다. 각각의 방식에는 모두 장단점이 있다.

#### 브리지 패턴
브리지 패턴의 경우 차량을 나타내는 한 가지 카테고리를 아예 멤버 포인터로 만들어버린다. 예를 들어서 `Vehicle` 클래스의 파생 클래스로 `LandVehicle`, `SpaceVehicle` 클래스들이 있고, `Vehicle` 클래스의 멤버 변수로 어떤 엔진을 사용하는지 가리키는 `Engine*` 멤버 변수가 있다. 이 `Engine` 은 `GasPowered`, `NuclearPowered` 와 같은 `Engine` 의 파생 클래스들의 객체들을 가리키게 된다. 그리고 런타임 시에 사용자가 `Engine` 을 적절히 설정해주면 된다. 이 경우 동력원이나 환경을 하나 추가하더라도 클래스를 1개만 더 만들면 된다. 즉, 총 N+M 개의 클래스만 생성하면 된다는 뜻이다.

하지만 오버라이딩 가지 수가 N+M 개 뿐이므로 최대 N+M 개 알고리즘 밖에 사용할 수 없다. 만일 N×M 개의 모든 상황에 대한 섬세한 제어가 필요하다면 브리지 패턴을 사용하지 않는 것이 좋다. 또한, 컴파일 타임 타입 체크를 적절히 활용할 수 없다는 문제가 있다. 예를 들어 `Engine` 이 페달이고 작동 환경이 우주라면, 애초에 해당 객체를 생성할 수 없어야 하지만 이를 컴파일 타임에서 강제할 방법이 없고 런타임에서나 확인할 수 있게 된다. 뿐만 아니라, 우주에서 작동하는 모든 차량을 가리킬 수 있는 기반 클래스를 만들 수 있지만 (`SpaceVehicle` 클래스), 작동 환경에 관계 없이 휘발유를 사용하는 모든 차량을 가리킬 수 있는 기반 클래스를 만들 수 는 없다.

#### 중첩된 일반화 방식(nested generalization)
중첩된 일반화 방식을 사용하게 된다면, 한 가지 계층을 먼저 골라서 파생 클래스들을 생성한다. 예를 들어 `Vehicle` 클래스의 파생 클래스들로 `LandVehicle`, `WaterVehicle`, 등등이 있을 것이다. 그 후에, 각각의 클래스들의 대해 다른 계층에 해당하는 파생 클래스들을 더 생성한다. 예컨대 `LandVehicle` 의 경우 동력원으로 휘발유를 사용한다면 `GasPoweredLandVehicle`, 원자력을 사용한다면 `NuclearPoweredLandVehicle` 클래스를 생성할 수 있을 것이다.

따라서 최대 N×M 가지의 파생 클래스들을 생성할 수 있게 된다. 따라서 브릿지 패턴에 비해서 좀 더 섬세한 제어를 할 수 있게 됩니다. 왜냐하면 오버라이딩 가지 수가 N+M 이 아닌 N×M 이 되기 때문이다. 하지만 동력원을 하나 더 추가하게 된다면 최대 N 개의 파생 클래스를 더 만들어야 한다. 뿐만 아니라 앞서 브릿지 패턴에서 나왔던 문제 - 휘발유를 사용하는 모든 차량을 가리킬 수 있는 기반 클래스를 만들 수 없다가 여전히 있다. 따라서 만약에 휘발유를 사용하는 차량들에서 공통적으로 사용되는 코드가 있다면 매 번 새로 작성해줘야만 한다.
    
다중 상속을 이용하게 된다면, 브리지 패턴 처럼 각 카테고리에 해당하는 파생 클래스들을 만들게 되지만, 그 대신 `Engine*` 멤버 변수를 없애고 동력원과 환경에 해당하는 클래스를 상속받는 파생 클래스들을 최대 N×M 개 만들게 된다. 예를 들어서 휘발유를 사용하며 지상에서 다니는 차량을 나타내는 `GasPoweredLandVehicle` 클래스의 경우 `GasPoweredEngine` 과 `LandVehicle` 두 개의 클래스를 상속받을 것이다.

따라서 이 방식을 통해서 브리지 패턴에서 불가능 하였던 섬세한 제어를 수행할 수 있을 뿐더러, 말도 안되는 조합을 (예컨대 `PedalPoweredSpaceVehicle`) 컴파일 타입에서 확인할 수 있다 (애초에 정의 자체를 안하면 되니까). 또한 이전에 두 방식에서 발생했던 휘발유를 사용하는 모든 차량을 가리킬 수 없다 문제를 해결할 수 있다. 왜냐하면 이제 `GasPoweredEngine` 을 통해서 휘발유를 사용하는 모든 차량을 가리킬 수 있기 때문이다.

가장 중요한 점은, 위 3가지 방식 중에서 절대적으로 우월한 방식은 없다는 것이다. 상황에 맞게 최선의 방식을 골라서 사용해야 한다.

다중 상속은 만능 툴이 아니다. 실제로 다중 상속을 이용해서 해결해야 될 것 같은 문제도 알고보면 단일 상속을 통해 해결할 수 있는 경우들이 있다. 하지만 적절한 상황에 다중 상속을 이용한다면 위력적인 도구가 될 수 있을 것이다.