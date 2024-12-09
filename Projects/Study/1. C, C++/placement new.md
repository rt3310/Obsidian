https://codingsuru0525.tistory.com/2

placement new는 메모리를 할당하는 방식 중 하나로, 이미 할당된 메모리 영역에 생성자를 명시적으로 호출하여 객체를 직접 생성하기 위한 방식이다.

일반적인 new 키워드를 사용하여 동적으로 메모리를 할당하는 것과는 조금 다르다.
보통 new 키워드를 사용하여 동적으로 메모리를 할당하면 시스템은 메모리를 할당하고 생성자를 호출하여 객체를 초기화한다. 반면 placement new는 이미 할당된 메모리 영역에 생성자를 호출하여 객체를 직접 생성한다(생성자만 호출).
```cpp
#include <iostream>
#include <new>
class Point {
	int x,y;
public:
	Point(int a, int b) : x{a},y{b} { std::cout <<"Point(int,int)"<<std::endl; }
	~Point(){ std::cout << "~Point" << std::endl; }
};

int main() {
	Point* p1 = new Point(1,2); 
	delete p1;
}
```

위 코드에서 동적으로 메모리를 할당하고 해제하는 과정은 다음과 같다.
1. 메모리 할당: `void* p = operator new(sizeof(Point))`
2. 생성자 명시적 호출: `Point* p1 = new(p)Point(1, 2)`
3. 소멸자 명시적 호출: `p1->~Point()`
4. 메모리 해제: `operator delete(p1)`

위 `operator new()`/`operator delete()`는 메모리를 할당/해제하는 global namespace에 있는 c++ 표준함수이다.
만약 생성자를 호출하지 않고 메모리만 할당하고 싶다면
`p1 = new Point(1, 2)`가 아닌 `void* p1 = operator new(sizeof(Point))`를 사용하면 된다. (이는 C언어의 malloc 함수와 같다)
마찬가지로 소멸자를 호출하지 않고 메모리 해제만 하고 싶다면 `delete p1`이 아닌 `operator delete(p1)`을 해주면 된다.

new를 객체지향으로 사용하려면 (클래스에 대한 동적메모리 할당)생성자를 불러야하는데, 이미 할당된 메모리에 메모리 할당없이 생성자를 명시적으로 호출하는 기법을 Placement new라고 한다.
```cpp
#include <iostream>
#include <new>
#include <memory>
class Point {
	int x, y;
public:
	Point(int a,int b) : x{a},y{b} { std::cout<<"Point(int,int)"<<std::endl; }
	~Point() { std::cout << "~Point" <<std::endl; }
};

int main() {
	void* p1 = operator new(sizeof(Point)); //메모리 할당
	Point* p2 = new(p1) Point(1,2); //void* 타입으로 넣었지만 객체가 생성되며 메모리를 가리키는 포인터가 Point*로 형식이 바뀐다.
	p2 -> ~Point(); //소멸자를 명시적으로 호출
	operator delete(p1);
}
```
객체에 대한 동적메모리를 할당하는 방법은 new/delete를 사용하는 방법이 있고,
 operator new/ operator delete와 replacement new를 사용하면 객체에 대한 동적 메모리 할당 시에도 메모리 할당과 생성자 호출을 분리할 수 있다.

이처럼 메모리 할당과 생성자 호출을 분리하면 생성자 초기화, 메모리 효율화 등에 유리하다.

생성자 초기화를 편리하게 할 수 있는 예시에 대해 알아보자.