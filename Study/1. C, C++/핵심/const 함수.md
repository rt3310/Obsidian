C++에서는 변수들의 값을 바꾸지 않고 읽기만 하는, 마치 상수 같은 멤버 함수를 '상수 함수'로써 선언할 수 있다.

상수 함수 내에서는 객체들의 '읽기' 만이 수행되며, 상수 함수 내에서 호출할 수 있는 함수로는 다른 상수 함수밖에 없다.

## mutable

하지만 `mutable` 키워드를 붙여 멤버 변수를 선언하면 해당 멤버 변수는 값을 수정할 수 있다.

그런데, `mutable`을 쓸거면 `const`를 떼어버리는게 낫지 않을까 생각할 수 있다.
먼저 멤버 함수를 왜 `const`로 선언하는지부터 생각해보자.

클래스의 멤버 함수를 `const`로 선언하는 의미는 '이 함수는 객체의 내부 상태에 영향을 주지 않습니다'를 표현하는 방법이다. 대표적인 예로 읽기 작업을 수행하는 함수들을 들 수 있다.

그럼 다음과 같이 DB에서 데이터를 조회하는데, 캐시를 만들어 빠르게 조회할 수 있도록 했다고 하자.
```cpp
class Server {
	Cache cache;
	
	 // 이 함수는 데이터베이스에서 user_id 에 해당하는 유저 정보를 읽어서 반환한다.
	 User GetUserInfo(const int user_id) const {
		// 1. 캐시에서 user_id 를 검색
		Data user_data = cache.find(user_id);
		
		// 2. 하지만 캐시에 데이터가 없다면 데이터베이스에 요청
		if (!user_data) {
			user_data = Database.find(user_id);
			
			// 그 후 캐시에 user_data 등록
			cache.update(user_id, user_data); // <-- 불가능
	    }	
	    // 3. 리턴된 정보로 User 객체 생성
	    return User(user_data);
	}
};
```
여기서 문제는 `GetuserInfo`가 `const` 함수라는 점이다. 따라서
```cpp
cache.update(user_id, user_date);
```
위처럼 캐시를 업데이트하는 작업을 수행할 수 없다. 왜냐하면 캐시를 업데이트 한다는 것은 캐시 내부의 정보를 바꿔야 한다는 뜻이기 때문이다. 따라서 저 `update` 함수는 `const` 함수가 아닐 것이다.

그렇다고 해서 `GetUserInfo`에서 `const`를 떼기도 좀 뭐한게, 이 함수를 사용하는 사용자의 입장에선 당연히 `const`로 정의되어야 하는 함수이기 때문이다. 따라서 이 경우, `Cache`를 `mutable`로 선언해버리면 된다.
```cpp
mutable Cache cache;
```