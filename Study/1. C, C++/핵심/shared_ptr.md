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
하지만 이는 바람직한 `shared_ptr`의 생성 방법은 아니다. 왜냐하면 일단 A를 생성하기 위해서 동적 할당이 한 번 일어나야 하고, 그 다음 `shared_ptr`의 제어 블록 역시 동적으로 할당해야 하기 때문이다.