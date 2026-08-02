클래스의 상속을 사용함으로써 중요하게 처리해야 되는 부분이 있다.
바로 소멸자를 가상함수로 만들어야 된다는 점이다.

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
성공적으로 컴파일 했다면
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
위와 같이 나온다. 여기서 호출 순서를 살펴보면,
`Parent` 생성자 -> `Child` 생성자 -> `Child` 소멸자 -> `Parent` 소멸자 순으로 호출됨을 알 수 있다.

그런데 `Parent` 포인터가 `Child` 객체를 가리킬 때는 문제가 발생한다.
```cpp
std::cout << "--- Parent 포인터로 Child 가리켰을 때 ---" << std::endl;
{
	Parent *p = new Child();
	delete p;
}
```
`delete p`를 하더라도, `p`가 가리키는 것은 `Parent` 객체가 아닌 `Child` 객체이기 때문에, 위에서 보통의 `Child` 객체가 소멸하는 것과 같은 순서로 생성자와 소멸자들이 호출되어야만 한다. 그런데 실제로는, `Child` 소멸자가 호출되지 않는다.

소멸자가 호출되지 않는다면 여러가지 문제가 생길 수 있다.
예를 들어, `Child` 객체에서 메모리를 동적으로 할당하고 소멸자에서 해제하는데, 소멸자가 호출되지 않았다면 **메모리 누수(memory leak)** 가 생길 것이다.

하지만 `Parent`의 소멸자를 `virtual`로 만든다면 `Child`의 소멸자를 성공적으로 호출할 수 있게 된다.
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
성공적으로 컴파일 했다면
```
--- 평범한 Child 만들었을 때 ---
Parent 생성자 호출
Child 생성자 호출
Child 소멸자 호출
Parent 소멸자 호출
--- Parent 포인터로 Child 가리켰을 때 ---
Parent 생성자 호출
Child 생성자 호출
Child 소멸자 호출
Parent 소멸자 호출
```
여기서 `Parent` 소멸자는 `Child` 소멸자를 호출하면서 `Child` 소멸자가 알아서 `Parent`의 소멸자도 호출해준다.

반면, `Parent` 소멸자를 먼저 호출하게 되면, `Parent`는 `Child`가 있는지 모르므로, `Child` 소멸자를 호출해 줄 수 없다.