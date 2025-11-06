## 정적 변수(static variable)

### 지역, 전역 변수
먼저 함수 내부에서 정의된 변수를 지역 변수(local variable)라고 하며, 지역 변수는 블록 스코프가 있고 자동 주기(정의 지점에서 생성, 블록이 종료되면 소멸)가 있다.
함수 외부에서 선언된 변수를 전역 변수(global variable)라고 하며, 전역 변수는 정적 주기(static duration)로, 프로그램이 시작될 때 생성되고 종료될 때 파괴된다. 전역 변수는 파일 스코프(or 전역 스코프)를 가진다.

### static (in global), extern 키워드를 이용한 내부/외부 링크
변수는 스코프와 주기외에도 링크(linkage)라는 속성이 있다. 링크는 같은 이름의 여러 식별자가 같은 식별자를 참조하는지를 결정한다.

링크가 없는 변수는 정의된 제한된 범위에서만 참조할 수 있다. 지역 변수가 링크가 없는 변수의 예이다.
이름은 같지만 다른 함수에서 정의된 지역 변수는 링크가 없다. 각 변수는 독립적이다.

- 내부 링크가 있는 변수를 '**static 변수**'라고 한다. static 변수는 변수가 정의된 소스 파일 내에서 어디서나 접근할 수 있지만, 소스 파일 외부에서는 참조할 수 없다. 즉, 현재 파일 내에서만 사용가능한 변수가 되버린다.
	- 지역 변수가 아닌 **전역 변수에 static을 붙인 경우에 해당**한다.
	- 현재 파일 내에서만 존재하는 파일의 지역변수 취급을 받기 때문에 이름이 같더라도 서로 다른 파일에서 각각 다른 변수로 취급이 된다.
	- 따라서 static 변수를 통해 혹시 모를 전역변수의 이름이 같게 되는 문제를 조금이나마 완화할 수 있다.

- 외부 링크가 있는 변수를 '**extern 변수**'라고 한다. extern 변수는 정의된 소스 파일과 다른 소스 파일 모두에서 접근할 수 있다. 즉, 다른 파일 간의 변수를 공유하고 있다라는 뜻이다.
	- extern 키워드는 전역 변수가 외부에 있다는 것을 표시만 할 뿐이고 선언을 하지 않는다. 따라서 어디엔가 정의되어 있지 않으면 링크 에러가 발생할 것이다.
	- extern은 다른 파일의 함수나 변수를 가져오는 것 이외에도 현재 파일 내에서도 작동이 가능하다.

> [!info] extern 키워드를 통한 변수 전방 선언
> 다른 소스 파일에서 선언된 외부 전역 변수를 사용하려면 '변수 전방 선언'을 해야 한다.
> 
> '`extern` 키워드는 두 가지 다른 의미가 있다. 어떤 상황에서는 extern 키워드가 '외부 링크가 있는 변수를 의미'하고 다른 상황에서는 '다른 어딘가에서 정의된 변수에 대한 전방 선언'을 의미한다.
> 
> 만약 변수 전방 선언이 함수 외부에서 선언되면 소스 파일 전체에 적용되고, 함수 내에서 선언되면 해당 블록 내에서만 적용된다. 

### static (in local)
`static` 키워드를 사용한 지역 변수는 자동 주기에서 정적 주기로 바뀐다. 이를 정적 변수(static variable)이라고 부른다.
정적 변수는 한 번만 초기화되며 프로그램 수명 내내 지속된다. 때문에 자신이 선언된 범위를 벗어나더라도 절대로 파괴되지 않는다.
```cpp
#include <iostream>
using namespace std;

int* function() {
	static int a = 2;
	return &a;
}

int main() {
	int* pa = function();
	cout << *pa << endl; // a가 static 변수가 아니면 scope가 function()까지 이기 때문에 error가 발생한다.
}
```

### static (in class)
클래스의 멤버 변수에 `static` 키워드를 붙이면 `static` 멤버 변수를 선언할 수 있다.
`static` 멤버 변수는 클래스 하나에만 종속되는 변수로, `static` 지역 변수처럼 프로그램이 종료될 때 소멸된다.

또한, `static` 멤버 변수의 경우, 클래스의 모든 객체들이 '공유'하는 변수로써 각 객체 별로 따로 존재하는 멤버 변수들과는 달리 모든 객체들이 '하나의' `static` 멤버 변수를 사용하게 된다.
```cpp
class Test {
	static int value;
public:
	Test();
	void show();
	~Test() { value--; }
}
int Test::value = 0;

Test::Test() {
	value++;
}

void Test::show() {
	cout << value << endl;
}

Test::~Test() {
	value--;
}

void create() {
	Test test3;
	test3.show();
}

int main() {
	Test test1;
	test1.show(); // 1
	
	Test test2;
	test2.show(); // 2
	
	create(); // 3
	
	test1.show(); // 2
	test2.show(); // 2
}
```

`static` 변수들은 정의와 동시에 값이 자동으로 0으로 초기화 되기 때문에 이 경우 굳이 따로 초기화 하지 않아도 되지만, 클래스 `static` 변수들의 경우 아래와 같은 방법으로 초기화한다.
```cpp
int Test::value = 0;
```

> [!info] static inline variable
> C++17부터는 인라인 변수(inline variable)을 제공하여, inline 키워드를 붙이면 다음과 같이 static 변수를 클래스 안에서 초기화할 수 있다.
> ```cpp
> class Test {
> 	static inline int value = 0;
> }
> ```
