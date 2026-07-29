`const` 함수 내부에서는 멤버 변수들의 값을 바꾸는 것이 불가능하다. 하지만, 만약 멤버 변수를 `mutable`로 선언했다면 `const` 함수에서도 이들 값을 바꿀 수 있다.

```cpp
#include <iostream>

class A {
	int data_;

public:
	A(int data) : data_(data) {}
	void DoSomething(int x) const {
		data_ = x;  // 불가능!
	}

	void PrintData() const { std::cout << "data: " << data_ << std::endl; }
};

int main() {
	A a(10);
	a.DoSomething(3);
	a.PrintData();
}
```
위 함수를 컴파일하면 에러가 뜨는 것을 볼 수 있다. `const` 함수 안에서 멤버 변수인 `data_` 를 수정했기 때문이다.

하지만 `data_`를 `mutable`로 선언하면 어떻게 될까?
```cpp
#include <iostream>

class A {
	mutable int data_;

public:
	A(int data) : data_(data) {}
	void DoSomething(int x) const {
		data_ = x;  // 가능!
	}

	void PrintData() const { std::cout << "data: " << data_ << std::endl; }
};

int main() {
	A a(10);
	a.DoSomething(3);
	a.PrintData();
}
```
성공적으로 컴파일 했다면
```
data: 3
```
으로 나오는 것을 볼 수 있다.

위처럼 `data_` 값이 const 함수 안에서 바뀐 것을 볼 수 있다.

그런데 생각해보면 `mutable`을 쓸 바에는 차라리 그냥 `DoSomething()`에서 `const`를 떼어버리는 것이 낫지 않을까? 왜 `mutable` 키워드를 사용할까?

먼저 멤버 함수를 왜 `const`로 선언하는지 부터 생각해보자.
클래스의 멤버 함수들은 '이 객체는 이러한 일을 할 수 있다'라는 의미를 나타내고 있다. 그리고 멤버 함수를 `const`로 선언하는 의미는 '이 함수는 객체의 내부 상태에 영향을 주지 않는다'를 표현하는 방법이다. 대표적인 예로 읽기 작업을 수행하는 함수들을 들 수 있다.

대부분의 경우 의미상 상수 작업을 하는 경우, 실제로도 상수 작업을 하게 된다. 하지만, 실제로 꼭 그렇지만은 않다. 예를 들어, 아래와 같은 서버 프로그램을 만든다고 해보자.
```cpp
class Server {
	// .... (생략) ....
	
	// 이 함수는 DB에서 user_id 에 해당하는 유저 정보를 읽어서 반환한다.
	User GetUserInfo(const int user_id) const {
		// 1. DB에 user_id 를 검색
		Data user_data = Database.find(user_id);
		
		// 2. 리턴된 정보로 User 객체 생성
		return User(user_data);
	}
};
```
이 서버에는 `GetUserInfo`라는 함수가 있는데 입력 받은 `user_id`로 DB에서 해당 유저를 조회해서 그 유저의 정보를 반환하는 함수이다. 당연히도 DB를 업데이트하지도 않고, 무언가를 수정하는 작업도 당연히 없기 때문에 `const` 함수로 선언되어 있다.

그런데 대개 DB에 요청한 후 받아오는 작업은 꽤나 오래 걸린다. 그래서 보통 서버들의 경우 메모리에 캐시(cache)를 만들어서 자주 요청되는 데이터를 굳이 DB까지 가서 찾지 않아도 메모리에서 빠르게 조회할 수 있도록 한다.

물론 캐시는 DB만큼 크지 않기 때문에 일부 유저들 정보 밖에 포함하지 않는다. 따라서 캐시에 해당 유저가 없다면 (이를 캐시 미스라고 한다), DB에 직접 요청해야 한다.
데신 DB에서 유저 정보를 받으면 캐시에 저장해놓아서 다음에 요청할 때는 빠르게 받을 수 있게 된다.

이를 구현하면 아래와 같을 것이다.
```cpp
class Server {
	// .... (생략) ....
	
	Cache cache; // 캐쉬!
	
	// 이 함수는 데이터베이스에서 user_id 에 해당하는 유저 정보를 읽어서 반환한다.
	User GetUserInfo(const int user_id) const {
		// 1. 캐쉬에서 user_id 를 검색
		Data user_data = cache.find(user_id);
		
		// 2. 하지만 캐쉬에 데이터가 없다면 데이터베이스에 요청
		if (!user_data) {
			user_data = Database.find(user_id);
			
			// 그 후 캐쉬에 user_data 등록
			cache.update(user_id, user_data); // <-- 불가능
		}
		
		// 3. 리턴된 정보로 User 객체 생성
		return User(user_data);
	}
};
```
그런데 문제는 `GetUserInfo`가 `const` 함수라는 점이다. 따라서
```cpp
cache.update(user_id, user_data);
```
위처럼 캐시를 업데이트하는 작업을 수행할 수 없다. 왜냐하면 캐시를 업데이트 한다는 것은 캐시 내부의 정보를 바꿔야 된다는 뜻이기 때문이다. 따라서 저 `update` 함수는 `const` 함수가 아닐 것이다.

그렇다고 해서 `GetUserInfo`에서 `const`를 떼기도 좀 뭐한 게, 이 함수를 사용하는 사용자의 입장에서는 당연히 `const`로 정의되어야 하는 함수이기 때문이다.
따라서 이 경우, `Cache`를 `mutable`로 선언해버리면 된다.
```cpp
mutable Cache cache;
```
이렇듯, `mutable` 키워드는 `const` 함수 안에서 해당 멤버 변수에 `const`가 아닌 작업을 할 수 있게 만들어준다.