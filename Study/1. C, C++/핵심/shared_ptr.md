## shared_ptr

`unique_ptr`을 사용해서 다 해결할 수 있으면 좋겠지만, 때에 따라서 여러 개의 스마트 포인터가 하나의 객체를 같이 소유해야 하는 경우가 발생한다.
예를 들어, 여러 객체에서 하나의 자원을 사용하고자 한다. 후에 자원을 해제하기 위해서는 이 자원을 사용하는 모든 객체들이 소멸되어야 하는데, 어떤 객체가 먼저 소멸되는지 알 수 없기 때문에 이 자원 역시 어느 타이밍에 해제시켜야 할 지 알 수 없게 된다.

따라서 이 경우에 특정 자원을 몇 개의 객체에서 가리키는지 추적한 다음, 그 수가 0이 되어야만 해제를 시켜주는 방식의 포인터가 필요하다. 이를 `shared_ptr`이 수행한다.

`shared_ptr`은 기존에 유일하게 객체를 소유하는 `unique_ptr`과는 다르게, `shared_ptr`로 객체를 가리킬 경우, 다른 `shared_ptr` 역시 그 객체를 가리킬 수 있다.
```cpp
std::shared_ptr<A> p1(new A());
std::shared_ptr<A> p2(p1);  // p2 역시 생성된 객체 A 를 가리킨다.

// 반면에 unique_ptr 의 경우
std::unique_ptr<A> p1(new A());
std::unique_ptr<A> p2(p1);  // 컴파일 오류!
```
![[13.2.1 1.webp]]
이렇게 여러 개의 `shared_ptr`들이 같은 객체를 가리키기 위해서는, 몇 개의 `shared_ptr`이 원래 객체를 가리키는지 알아야 한다. 이를 참조 개수(reference count)라고 하는데, 참조 개수가 0이 되어야 가리키고 있는 객체를 해제할 수 있다.
![[Pasted image 20251111094032.png]]
위 그림의 경우 `p1`과 `p2`가 같은 객체를 가리키고 있으므로, 참조 개수가 2가 된다.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class A {
	int *data;

 public:
	A() {
		data = new int[100];
		std::cout << "자원을 획득함!" << std::endl;
	}
	
	~A() {
		std::cout << "소멸자 호출!" << std::endl;
		delete[] data;
	}
};

int main() {
	std::vector<std::shared_ptr<A>> vec;
	
	vec.push_back(std::shared_ptr<A>(new A()));
	vec.push_back(std::shared_ptr<A>(vec[0]));
	vec.push_back(std::shared_ptr<A>(vec[1]));
	
	// 벡터의 첫번째 원소를 소멸 시킨다.
	std::cout << "첫 번째 소멸!" << std::endl;
	vec.erase(vec.begin());
	
	// 그 다음 원소를 소멸 시킨다.
	std::cout << "다음 원소 소멸!" << std::endl;
	vec.erase(vec.begin());
	
	// 마지막 원소 소멸
	std::cout << "마지막 원소 소멸!" << std::endl;
	vec.erase(vec.begin());
	
	std::cout << "프로그램 종료!" << std::endl;
}
```

```
자원을 획득함!
첫 번째 소멸!
다음 원소 소멸!
마지막 원소 소멸!
소멸자 호출!
프로그램 종료!
```
이 코드의 결과를 보면 `shared_ptr`의 경우 객체를 가리키는 모든 스마트 포인터들이 소멸되어야만 객체를 파괴하기 때문에, 처음 두 번의 `erase`에서는 아무것도 하지 않다가 마지막의 `erase`에서 비로소 A의 소멸자를 호출하는 것을 볼 수 있다.

```cpp
std::shared_ptr<A> p1(new A());
std::shared_ptr<A> p2(p1);  // p2 역시 생성된 객체 A 를 가리킨다.

std::cout << p1.use_count();  // 2
std::cout << p2.use_count();  // 2
```

그러면 `shared_ptr`들은 참조 개수가 몇 개인지 알고 있어야만 하는데, 어떻게 하면 같은 객체를 가리키는 `shared_ptr`끼리 동기화를 시킬 수 있을까?

만약, `shared_ptr` 내부에 참조 개수를 저장한다면 아래와 같은 문제가 생길 수 있다.
이렇게 한 개의 `shared_ptr`이 추가적으로 해당 객체를 가리킨다면
```cpp
std::shared_ptr<A> p3(p2);
```
![[Pasted image 20251111094620.png]]
여차저차해서 `p2`의 참조 카운트 개수는 증가시킬 수 있다고 해도, `p1`에 저장되어 있는 참조 개수를 건드릴 수 없다.

따라서 이와 같은 문제를 방지하기 위해 처음으로 실제 객체를 가리키는 `shared_ptr`이 **제어 블록(control block)** 을 동적으로 할당한 후, `shared_ptr`들이 이 제어 블록에 필요한 정보를 공유하는 방식으로 구현된다.
![[Pasted image 20251111094744.png]]
`shared_ptr`는 복사 생성할 때마다 해당 제어 블록의 위치만 공유하면 되고, `shared_ptr`가 소멸할 때마다 제어 블록의 참조 개수를 하나 줄이고, 생성할 때 마다 하나 늘리는 방식으로 작동할 것이다.

## make_shared

앞서 `shared_ptr`을 처음 생성할 때 아래와 같이 했다.
```cpp
std::shared_ptr<A> p1(new A());
```
하지만 이는 바람직한 `shared_ptr`의 생성 방법은 아니다. 왜냐하면 일단 A를 생성하기 위해서 동적 할당이 한 번 일어나야 하고, 그 다음 `shared_ptr`의 제어 블록 역시 동적으로 할당해야 하기 때문이다. 즉, 2번의 동적 할당이 발생한다.

동적 할당은 상당히 비싼 연산이다. 어차피 동적 할당을 두 번 할 것이라는 것을 알고 있다면, 아예 2개 합친 크기로 한 번 할당하는 것이 훨씬 빠르다.
```cpp
std::shared_ptr<A> p1 = std::make_shared<A>();
```
`make_shared` 함수는 A의 생성자의 인자들을 받아서 이를 통해 객체 `A`와 `shared_ptr`의 제어 블록까지 한 번에 동적 할당 한 후에 만들어진 `shared_ptr`를 반환한다.

위 경우 `A`의 생성자에 인자가 없어서 `make_shared`에 아무것도 전달하지 않았지만, 만약 `A`의 생성자에 인자가 있다면 `make_shared`에 인자로 전달해주면 된다.

## 생성 시 주의할 점

`shared_ptr`은 인자로 주속밧이 전달된다면, 마치 자기가 해당 객체를 첫 번째로 소유하는 `shared_ptr`인 것마냥 행동한다. 예를 들어,
```cpp
A* a = new A();
std::shared_ptr<A> pa1(a);
std::shared_ptr<A> pa2(a);
```
를 하게 된다면 아래와 같이 이 두개의 제어 블록이 따로 생성된다.
![[Pasted image 20251111100253.png]]
따라서 위와 같이 제어 블록들은 다른 제어 블록들의 존재를 모르고 참조 개수를 1로 설정하기 때문에, 만약 `pa1`이 소멸된다면 참조 카운트가 0이 되어서 자신이 가리키는 객체 `A`를 소멸시켜 버린다.

물론 `pa2`의 참조 카운트는 계속 1이기 때문에 자신이 가리키는 객체가 살아있을 것이라 생각할 것이다. 만약 운 좋게 `pa2`를 사용하지 않아도, `pa2`가 소멸되면 참조 개수가 0으로 떨어지고 자신이 가리키고 있는 객체를 소멸시키기 때문에 오류가 발생한다.

이를 방지하려면 `shared_ptr`를 주소값을 통해서 생성하는 것을 지양해야 한다.
하지만, 어쩔 수 없는 상황도 있다. 바로 객체 내부에서 자기 자신을 가리키는 `shared_ptr`를 만들 때를 생각해보자.
```cpp
#include <iostream>
#include <memory>

class A {
	int *data;

public:
	A() {
		data = new int[100];
		std::cout << "자원을 획득함!" << std::endl;
	}

	~A() {
		std::cout << "소멸자 호출!" << std::endl;
		delete[] data;
	}

	std::shared_ptr<A> get_shared_ptr() { return std::shared_ptr<A>(this); }
};

int main() {
	std::shared_ptr<A> pa1 = std::make_shared<A>();
	std::shared_ptr<A> pa2 = pa1->get_shared_ptr();
	
	std::cout << pa1.use_count() << std::endl;
	std::cout << pa2.use_count() << std::endl;
}
```
이 코드를 실행시켜보면, 똑같이 소멸 시에 오류가 발생하게 된다.
`get_shared_ptr()` 함수에서 `shared_ptr`를 생성할 때, 이미 자기 자신을 가리키는 `shared_ptr`가 있다는 사실을 모른 채 새로운 제어 블록을 생성하기 때문이다.

이 문제는 `enable_shared_from_this`를 통해 깔끔하게 해결할 수 있다.

### enable_shared_from_this
우리가 `this`를 사용해서 `shared_ptr`를 만들고 싶은 클래스가 있다면 `enable_shared_from_this`를 상속받으면 된다.
```cpp
#include <iostream>
#include <memory>

class A : public std::enable_shared_from_this<A> {
	int *data;

public:
	A() {
		data = new int[100];
		std::cout << "자원을 획득함!" << std::endl;
	}

	~A() {
		std::cout << "소멸자 호출!" << std::endl;
		delete[] data;
	}

	std::shared_ptr<A> get_shared_ptr() { return shared_from_this(); }
};

int main() {
	std::shared_ptr<A> pa1 = std::make_shared<A>();
	std::shared_ptr<A> pa2 = pa1->get_shared_ptr();
	
	std::cout << pa1.use_count() << std::endl;
	std::cout << pa2.use_count() << std::endl;
}
```
위 코드를 실행시켜 보면 제대로 작동하는 것을 볼 수 있다.

`enable_shared_from_this` 클래스에는 `shared_from_this`라는 멤버 함수를 정의하고 있는데, 이 함수는 이미 정의되어 있는 제어 블록을 사용해서 `shared_ptr`를 생성한다.
따라서 이전처럼 같은 객체에 2개의 다른 제어 블록이 생성되는 일을 막을 수 있다.

한 가지 중요한 점은 `shared_from_this`가 잘 작동하기 위해서는 해당 객체의 `shared_ptr`가 **반드시 먼저 정의되어 있어야만 한다**는 것이다. 즉, `shared_from_this`는 있는 제어 블록을 확인만 할 뿐, 없는 제어 블록을 만들지는 않는다. 쉽게 말해 아래 코드는 오류가 발생한다.
```cpp
A* a = new A();
std::shared_ptr<A> pa1 = a->get_shared_ptr();
```

## 서로 참조하는 shared_ptr

앞서 `shared_ptr`는 참조 개수가 0이 되면 가리키는 객체를 메모리에서 해제시킨다고 했다. 그런데, 객체들을 더 이상 사용하지 않는데도 불구하고 참조 개수가 절대로 0이 될 수 없는 상황이 있다.
![[Pasted image 20251111101213.png]]
위 그림의 경우 각 객체는 `shared_ptr`를 하나 씩 가지고 있는데, 이 `shared_ptr`가 다른 객체를 가리키고 있다. 즉 객체1의 `shared_ptr`는 객체2를 가리키고 있고, 객체2의 `shared_ptr`는 객체1을 가리키고 있다.

만약 객체1이 파괴가 되기 위해서는 객체1을 가리키고 있는 `shared_ptr`의 참조 개수가 0이 되어야만 한다. 즉, 객체2가 파괴되어야 한다. 하지만 객체2가 파괴되기 위해서는 마찬가지로 객체2를 가리키고 있는 `shared_ptr`의 참조 개수가 0이 되어야 하는데, 그러기 위해서는 객체1이 파괴되어야만 한다.

이러지도 저러지도 못하는 상황이 된 것이다.
```cpp
#include <iostream>
#include <memory>

class A {
	int *data;
	std::shared_ptr<A> other;

public:
	A() {
		data = new int[100];
		std::cout << "자원을 획득함!" << std::endl;
	}

	~A() {
		std::cout << "소멸자 호출!" << std::endl;
		delete[] data;
	}

	void set_other(std::shared_ptr<A> o) { other = o; }
};

int main() {
	std::shared_ptr<A> pa = std::make_shared<A>();
	std::shared_ptr<A> pb = std::make_shared<A>();
	
	pa->set_other(pb);
	pb->set_other(pa);
}
```
코드를 실행시켜보면
```
자원을 획득함!
자원을 획득함!
```
위와 같이 소멸자가 제대로 호출되지 않음을 알 수 있다.

이 문제는 `shared_ptr` 자체에 내재되어 있는 문제이기 때문에 `shared_ptr`을 통해서는 이를 해결할 수 없다.
이러한 순환 참조 문제를 해결하기 위해 나타난 것이 바로 `weak_ptr`이다.

## weak_ptr

먼저 트리 형태를 자료 구조로 나타낸다면 어떻게 할 수 있을까?
```cpp
class Node {
	std::vector<std::shared_ptr<Node>> children;
	/* 어떤 타입이 와야할까? */ parent;
	
public:
	Node() {}
	void AddChild(std::shared_ptr<Node> node) { children.push_back(node); }
};
```
일단 기본적으로 위와 같은 형태를 취한다고 볼 수 있다. 부모가 여러 개의 자식 노드들을 가지므로 `shared_ptr`들의 vector로 나타낼 수 있고, 그 노드 역시 부모 노드가 있으므로 부모 노드를 가리키는 포인터를 가진다.

여기서 질문은 과연 `parent`의 타입을 무엇으로 하냐이다.
- 만약 일반 포인터(`Node*`)로 하게 된다면, 메모리 해제를 까먹고 하지 않을 경우 혹은 예외가 발생했을 경우 적절하게 자원을 해제하기 어렵다. 물론 이미 해제된 메모리를 계속 가리키고 있을 위험도 있다.
- 하지만 이를 `shared_ptr`로 하게 된다면 앞서 본 순환 참조 문제가 생긴다. 부모와 자식이 서로를 가리키기 때문에 참조 개수가 절대로 0이 될 수 없다. 따라서, 이 객체들은 프로그램이 끝날 때까지 절대로 소멸되지 못하고 남아있게 된다.

`weak_ptr`는 일반 포인터와 `shared_ptr` 사이에 위치한 스마트 포인터로, 스마트 포인터처럼 객체를 안전하게 참조할 수 있게 해주지만, **`shared_ptr` 와는 다르게 참조 개수를 늘리지는 않는다**. 이름 그대로 약한 포인터인 것이다.

따라서 설사 어떤 객체를 `weak_ptr`가 가리키고 있다고 하더라도, 다른 `shared_ptr`들이 가리키고 있지 않다면 이미 메모리에서 소멸되었을 것이다.

이 대문에 `weak_ptr` 자체로는 원래 객체를 참조할 수 없고, 반드시 `shared_ptr`로 변환해서 사용해야 한다. 이 때 가리키고 있는 객체가 이미 소멸되었다면 빈 `shared_ptr`로 변환되고, 아닐 경우 해당 객체를 가리키는 `shared_ptr`로 변환된다.

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <vector>

class A {
	std::string s;
	std::weak_ptr<A> other;

public:
	A(const std::string& s) : s(s) { std::cout << "자원을 획득함!" << std::endl; }
	
	~A() { std::cout << "소멸자 호출!" << std::endl; }

	void set_other(std::weak_ptr<A> o) { other = o; }
	void access_other() {
		std::shared_ptr<A> o = other.lock();
		if (o) {
			  std::cout << "접근 : " << o->name() << std::endl;
		} else {
			  std::cout << "이미 소멸됨 ㅠ" << std::endl;
		}
	}
	std::string name() { return s; }
};

int main() {
	std::vector<std::shared_ptr<A>> vec;
	vec.push_back(std::make_shared<A>("자원 1"));
	vec.push_back(std::make_shared<A>("자원 2"));
	
	vec[0]->set_other(vec[1]);
	vec[1]->set_other(vec[0]);
	
	// pa 와 pb 의 ref count 는 그대로다.
	std::cout << "vec[0] ref count : " << vec[0].use_count() << std::endl;
	std::cout << "vec[1] ref count : " << vec[1].use_count() << std::endl;
	
	// weak_ptr 로 해당 객체 접근하기
	vec[0]->access_other();
	
	// 벡터 마지막 원소 제거 (vec[1] 소멸)
	vec.pop_back();
	vec[0]->access_other();  // 접근 실패!
}
```

```
자원을 획득함!
자원을 획득함!
vec[0] ref count : 1
vec[1] ref count : 1
접근 : 자원 2
소멸자 호출!
이미 소멸됨 ㅠ
소멸자 호출!
```
성공적으로 실행했다면 위와 같이 나올 것이다.

일단 `weak_ptr`을 정의하는 부분을 살펴보자.
```cpp
vec[0]->set_other(vec[1]);
vec[1]->set_other(vec[0]);
```
`set_other` 함수는 `weak_ptr<A>`를 인자로 받고 있었는데, 여기에 `shared_ptr`을 전달했다. 즉, `weak_ptr`는 생성자로 `shared_ptr`나 다른 `weak_ptr`를 받는다. 또한 `shared_ptr`와는 다르게, **이미 제어 블록이 만들어진 객체만이 의미를 가지기 때문에, 평범한 포인터 주소값으로 `weak_ptr`를 생성할 수는 없다**.

그 다음 `weak_ptr`를 `shared_ptr`로 변환하는 과정을 살펴보자.
```cpp
void access_other() {
	std::shared_ptr<A> o = other.lock();
	if (o) {
		std::cout << "접근 : " << o->name() << std::endl;
	} else {
		std::cout << "이미 소멸됨 ㅠ" << std::endl;
	}
}
```
앞서 말했듯, `weak_ptr` 그 자체로는 원소를 참조할 수 없고, `shared_ptr`로 변환해야 한다. 이 작업은 `lock()` 함수를 통해 수행할 수 있다.

`weak_ptr`에 정의된 `lock()` 함수는 만일 `weak_ptr`가 가리키고 있는 객체가 아직 메모리에 살아있다면(참조 개수가 0이 아니라면), 해당 객체를 가리키는 `shared_ptr`를 반환하고, 이미 해제가 되었다면 아무것도 가리키지 않는 `shared_ptr`를 반환한다.
```cpp
std::shared_ptr<A> o = other.lock();
if (o) {
	std::cout << "접근 : " << o->name() << std::endl;
}
```
참고로 아무것도 가리키지 않는 `shared_ptr`는 `false`로 형변환 되므로 위와 같이 `if`문으로 간단히 확인할 수 있다.

앞서 제어 블록에는 몇 개의 `shared_ptr`가 가리키고 있는지를 나타내는 참조 개수(ref count)가 있다고 했다. 그리고 참조 개수가 0이 되면 해당 객체를 메모리에서 해제한다. 그렇다면 참조 개수가 0이 될 때 제어 블록 역시 메모리에서 해제해야 할까?
아니다. 만약 가리키는 `shared_ptr`은 0개 지만 아직 `weak_ptr`가 남아있다고 해보자. 물론 이 상태에서는 이미 객체는 해제되어 있을 것이다. 하지만 제어 블록 마저 해제해 버린다면, 제어 블록에서 참조 카운트가 0이라는 사실을 알 수 없게 된다.

> 메모리가 해제된 이후에, 같은 자리가 다른 용도로 할당될 수 있다. 이 때문에 참조 카운트 위치에 있는 메모리가 다른 값으로 덮어 씌어질 수도 있다.

즉, 제어 블록을 메모리에서 해제하기 위해서는 이를 가리키는 `weak_ptr` 역시 0개여야 한다. 따라서 제어 블록에는 참조 개수와 더불어 **약한 참조 개수(weak count)** 를 기록하게 된다.