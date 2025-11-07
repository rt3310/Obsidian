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

소멸자가 호출되지 않는다면 여러가지 문제가 생길 수 있다. 예를 들어서, `Child` 객체에서 메모리를 동적으로 할당하고 소멸자에서 해제하는데, 소멸자가 호출 안됐다면 메모리 누수(memory leak)가 생길 것이다.

단순히 `Parent`의 소멸자를 `virtual`로 만들어버리면 된다. `Parent`의 소멸자를 `virtual`로 만들면, `p`가 소멸자를 호출할 때, `Child` 의 소멸자를 성공적으로 호출할 수 있게 됩니다.