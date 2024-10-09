저번에는 `unique_ptr`에 대해서 다루어보았다. 대부분의 경우 하나의 자원은 한 개의 스마트 포인터에 의해 소유되는 것이 바람직하고, 나머지 접근은 (소유가 아닌) 그냥 일반 포인터로 처리하면 된다.

하지만, 때에 따라서는 여러 개의 스마트 포인터가 하나의 객체를 같이 소유해야 하는 경우가 발생한다. 예를 들어, 여러 객체에서 하나의 자원을 사용하고자 한다면 후에 자원을 해제하기 위해서는 이 자원을 사용하는 모든 객체들이 소멸되야 하는데, 어떤 객체가 먼저 소멸되는지 알 수 없기 때문에 이 자원 역시 어느 타이밍에 해제 시켜야 할 지 알 수 없게 된다.

따라서 이 경우, 좀 더 스마트한 포인터가 있어서, 특정 자원을 몇 개의 객체에서 가리키는지를 추적한 다음에, 그 수가 0이 되어야만 비로소 해제를 시켜주는 방식의 포인터가 필요하다.

## shared_ptr

`shared_ptr`은 앞서 이야기한 방식을 정확히 수행하는 스마트 포인터이다. 기존에 유일하게 객체를 소유하는 `unique_ptr`와는 다르게, `shared_ptr`로 객체를 가리킬 경우, 다른 `shared_ptr`역시 그 객체를 가리킬 수 있다. 예를 들어
```cpp
std::shared_ptr<A> p1(new A());
std::shared_ptr<A> p2(p1);  // p2 역시 생성된 객체 A 를 가리킨다.

// 반면에 unique_ptr 의 경우
std::unique_ptr<A> p1(new A());
std::unique_ptr<A> p2(p1);  // 컴파일 오류!
```
`p1`과 `p2`의 경우 같이 동일한 객체인 `A()`를 가리키지만, `unique_ptr`의 경우 유일한 소유권만 인정되므로 컴파일 오류가 발생하게 된다.
![[13.2.1.webp]]
위 그림과 같이 `shared_ptr`는 같은 객체를 가리킬 수 있다. 이를 위해서는, 앞서 말했듯이, 몇 개의 `shared_ptr`들이 원래 객체를 가리키는지 알아야만 합니다. 이를 **참조 개수(reference count)** 라고 하는데, 참조 개수가 0이 되어야 가리키고 있는 객체를 해제할 수 있다.
![[Pasted image 20241009232701.png]]
위 그림의 경우 `p1`과 `p2`가 같은 객체를 가리키고 있으므로, 참조 개수가 2가 된다.

한번 아래 예제를 살펴보자
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
성공적으로 컴파일했다면
```
자원을 획득함!
첫 번째 소멸!
다음 원소 소멸!
마지막 원소 소멸!
소멸자 호출!
프로그램 종료!
```
와 같이 나온다.

위 예제의 경우 `shared_ptr`를 원소로 가지는 벡터 `vec`을 정의한 후, `vec[0], vec[1], vec[2]`가 모두 같은 A 객체를 가리키는 `shared_ptr`를 생성했다.

```cpp

```