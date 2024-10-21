## sort

[sort](https://modoocode.com/272) 에 들어가는 반복자의 경우 반드시 임의접근 반복자(RandomAccessIterator) 타입을 만족해야 하므로, 우리가 봐왔던 컨테이너들 중에선 벡터와 데크만 가능하고 나머지 컨테이너는 [sort](https://modoocode.com/272) 함수를 적용할 수 없다. (예를 들어 리스트의 경우 반복자 타입이 양방향 반복자(BidirectionalIterator) 이므로 안된다)
```cpp
list<int> l;
sort(l.begin(), l.end());
```
만약에 위 처럼 리스트를 정렬하려고 했다간
![[Pasted image 20241006015717.png]]
위와 같은 무시무시한 컴파일 오류를 맛보게 될 것이다!

[sort](https://modoocode.com/272) 함수는 기본적으로 오름차순으로 정렬을 해준다. 그렇다면 만약에 내림 차순으로 정렬하고 싶다면 어떻게 할까? 만약 직접 만든 타입이였다면 단순히 `operator<` 를 반대로 바꿔준다면 오름차순에서 내림차순이 되었겠지만, 이 경우 `int` 이기 때문에 이는 불가능하다.

하지만 앞서 대부분의 알고리즘은 3번째 인자로 특정한 조건을 전달한다고 했는데, 여기에 우리가 비교를 어떻게 수행할 것인지에 대해 알려주면 된다.
```cpp
#include <algorithm>
#include <iostream>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << *begin << " ";
		begin++;
	}
	std::cout << std::endl;
}
struct int_compare {
	bool operator()(const int& a, const int& b) const { return a > b; }
};
int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(6);
	vec.push_back(4);
	vec.push_back(7);
	vec.push_back(2);
	
	std::cout << "정렬 전 ----" << std::endl;
	print(vec.begin(), vec.end());
	std::sort(vec.begin(), vec.end(), int_compare());
	
	std::cout << "정렬 후 ----" << std::endl;
	print(vec.begin(), vec.end());
}
```

## 원소 제거 (remove, remove_if)

**[remove](https://modoocode.com/266) 함수는 원소의 이동 만을 수행하지 실제로 원소를 삭제하는 연산을 수행하지는 않는다**. 따라서 벡터에서 실제로 원소를 지우기 위해서는 반드시 [erase](https://modoocode.com/240) 함수를 호출하여 실제로 원소를 지워줘야만 한다.
```cpp
vec.erase(remove(vec.begin(), vec.end(), 3), vec.end());
```
따라서 위 처럼 [remove](https://modoocode.com/266) 함수를 이용해서 값이 3 인 원소들을 뒤로 보내버리고, 그 원소들을 벡터에서 삭제해버리게 된다.

참고로 말하자면 [remove](https://modoocode.com/266) 함수의 경우 반복자의 타입이 `ForwardIterator`다. 즉, 벡터 뿐만이 아니라, 리스트, 혹은 셋이나 맵에서도 모두 사용할 수 있다!

그렇다면 이번에는 값이 딱 얼마로 정해진 것이 아니라 특정한 조건을 만족하는 원소들을 제거하려면 어떻게 해야 할까? 당연히 이 원소가 그 조건을 만족하는지 아닌지를 판단할 함수를 전달해야 된다. 이를 위해선 [remove_if](https://modoocode.com/266) 함수를 사용해야 한다.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}
struct is_odd {
	bool operator()(const int& i) { return i % 2 == 1; }
};

int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	std::cout << "처음 vec 상태 ------" << std::endl;
	print(vec.begin(), vec.end());
	
	std::cout << "벡터에서 홀수 인 원소 제거 ---" << std::endl;
	vec.erase(std::remove_if(vec.begin(), vec.end(), is_odd()), vec.end());
	print(vec.begin(), vec.end());
}
```

### remove_if에 조건 추가
우리의 [remove_if](https://modoocode.com/266) 함수는 함수 객체가 인자를 딱 1개 만 받는다고 가정한다. 따라서 호출되는 `operator()` 을 통해선 원소에 대한 정보 말고는 추가적인 정보를 전달하기는 어렵다.

하지만 예를 들어서 홀수인 원소들을 삭제하되 처음 2개만 삭제한다고 해보자. 함수 객체의 경우 사실 클래스의 객체이기 때문에 멤버 변수를 생각할 수 있다.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}
struct is_odd {
	int num_delete;
	
	is_odd() : num_delete(0) {}
	
	bool operator()(const int& i) {
		if (num_delete >= 2) return false;
		
		if (i % 2 == 1) {
			num_delete++;
			return true;
		}
		
		return false;
	}
};

int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	std::cout << "처음 vec 상태 ------" << std::endl;
	print(vec.begin(), vec.end());
	
	std::cout << "벡터에서 홀수인 원소 앞의 2개 제거 ---" << std::endl;
	vec.erase(std::remove_if(vec.begin(), vec.end(), is_odd()), vec.end());
	print(vec.begin(), vec.end());
}
```
성공적으로 컴파일했다면
```
처음 vec 상태 ------
[5] [3] [1] [2] [3] [4] 
벡터에서 홀수인 원소 앞의 2개 제거 ---
[2] [3] [4]
```
예상과는 사뭇 다른 결과가 나왔다. 홀수 원소 2 개가 아니라 3 개가 삭제됐다. 분명히 우리는 2개 이상 되면 `false` 를 리턴하라고 명시했는데도 말이다.

사실 C++ 표준에 따르면 [remove_if](https://modoocode.com/266) 에 전달되는 함수 객체의 경우 이전의 호출에 의해 내부 상태가 달라지면 안된다. 다시 말해, 위처럼 함수 객체 안에 인스턴스 변수 (`num_delete`) 를 넣는 것은 원칙상 안된다는 것이다.

그 이유는 [remove_if](https://modoocode.com/266) 를 실제로 구현 했을 때, 해당 **함수 객체가 여러 번 복사 될 수 있기 때문**이다. 물론, 이는 어떻게 구현하냐에 따라서 달라진다. 예를 들어 아래 버전을 살펴보자.
```cpp
template <class ForwardIterator, class UnaryPredicate>
ForwardIterator remove_if(ForwardIterator first, ForwardIterator last,
                          UnaryPredicate pred) {
	ForwardIterator result = first;
	while (first != last) {
		if (!pred(*first)) {  // <-- 함수 객체 pred 를 실행하는 부분
			*result = std::move(*first);
			++result;
		}
		++first;
	}
	return result;
}
```
위 버전에 경우 인자로 전달된 함수 객체 `pred` 는 복사되지 않고 계속 호출된다. 따라서 사실 우리의 원래 코드가 위 [remove_if](https://modoocode.com/266) 를 바탕으로 실행됬더라면 2개만 정확히 지워질 수 있다.

하지만 문제는 C++ 표준은 [remove_if](https://modoocode.com/266) 함수를 어떤 방식으로 구현하라고 정해 놓지 않는다. 어떤 라이브러리들의 경우 아래와 같은 방식으로 구현되었다 (사실 대부분의 라이브러리들이 아래와 비슷하다)
```cpp
template <class ForwardIt, class UnaryPredicate>
ForwardIt remove_if(ForwardIt first, ForwardIt last, UnaryPredicate pred) {
	first = std::find_if(first, last, pred);  // <- pred 한 번 복사됨
	if (first != last)
	// 아래 for 문은 first + 1 부터 시작한다고 봐도 된다 (++i != last)
	for (ForwardIt i = first; ++i != last;)
		if (!pred(*i))  // <-- pred 호출 하는 부분
			*first++ = std::move(*i);
	return first;
}
```
참고로 [find_if](https://modoocode.com/263) 함수의 경우 인자로 전달된 조건 `pred` 가 참인 첫 번째 원소를 리턴한다. 그런데 문제는 [find_if](https://modoocode.com/263) 가 함수 객체 `pred` 의 레퍼런스를 받는 것이 아니라, 복사 생성된 버전을 받는다는 점이다. 따라서, [find_if](https://modoocode.com/263) 호출 후에 아래 `for` 문에서 이미 한 개 원소를 지웠다는 정보가 소멸되게 된다. 후에 호출되는 `pred` 들은 이미 `num_delete` 가 1인지 모른 채 0부터 다시 시작하게 된다.

다시 한 번 말하자면, 함수 객체에는 절대로 특정 상태를 저장해서 이에 따라 결과가 달라지는 루틴을 짜면 안된다. 위처럼 이해하기 힘든 오류가 발생할 수도 있다.

그렇다면 위 문제를 어떻게 하면 해결할 수 있을까? 한 가지 방법은 `num_delete` 를 객체 내부 변수가 아니라 외부 변수로 빼는 방법이다.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}

struct is_odd {
	int* num_delete;
	
	is_odd(int* num_delete) : num_delete(num_delete) {}
	
	bool operator()(const int& i) {
		if (*num_delete >= 2) return false;
	
		if (i % 2 == 1) {
			(*num_delete)++;
			return true;
		}
		
		return false;
	}
};
int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	std::cout << "처음 vec 상태 ------" << std::endl;
	print(vec.begin(), vec.end());
	
	std::cout << "벡터에서 홀수인 원소 앞의 2개 제거 ---" << std::endl;
	int num_delete = 0;
	vec.erase(std::remove_if(vec.begin(), vec.end(), is_odd(&num_delete)),
			vec.end());
	print(vec.begin(), vec.end());
}
```
성공적으로 컴파일했다면
```
처음 vec 상태 ------
[5] [3] [1] [2] [3] [4] 
벡터에서 홀수인 원소 앞의 2개 제거 ---
[1] [2] [3] [4]
```
와 같이 제대로 나온다. 위 경우, `num_delete` 에 관한 정보를 아예 함수 객체 밖으로 빼서 보관해버렸다. 물론 함수 객체에 내부 상태인 `num_delete` 의 주소 값은 변하지 않으므로 문제될 것이 없다.

그런데 한 가지 안좋은 점은 이렇게 `STL` 을 사용할 때 마다 외부에 클래스나 함수를 하나씩 만들어줘야 된다는 점이다. 물론 프로젝트의 크기가 작다면 크게 문제가 되지는 않겠지만 프로젝트의 크기가 커진다면, 만약 다른 사람이 코드를 읽을 때 '이 클래스는 뭐하는 거지?' 혹은 '이 함수는 뭐하는 거지?' 와 같은 궁금증이 생길 수도 있고 심지어 잘못 사용할 수도 있다.

따라서 가장 이상적인 방법은 `STL` 알고리즘을 사용할 때 그 안에 직접 써놓는 것이다. 마치
```cpp
vec.erase(std::remove_if(vec.begin(), vec.end(), bool is_odd(int i) { return i % 2 == 1; }), vec.end());
```
이런 식으로 말이다. 문제는 위 문법이 C++ 에서 허용되지 않다는 점이다. 하지만 놀랍게도 `C++ 11` 부터 위 문제를 해결할 방법이 나타났다.

## 람다 함수(lambda function)

람다 함수는 C++ 에서는 `C++ 11` 에서 처음으로 도입되었다. 람다 함수를 통해 쉽게 이름이 없는 함수 객체를 만들 수 있게 되었습니다. 

람다 함수를 사용한 예제부터 먼저 살펴보자.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}
int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	std::cout << "처음 vec 상태 ------" << std::endl;
	print(vec.begin(), vec.end());
	
	std::cout << "벡터에서 홀수인 원소 제거 ---" << std::endl;
	vec.erase(std::remove_if(vec.begin(), vec.end(), [](int i) -> bool { return i % 2 == 1; }), vec.end());
	print(vec.begin(), vec.end());
}
```
성공적으로 컴파일했다면
```
처음 vec 상태 ------
[5] [3] [1] [2] [3] [4] 
벡터에서 홀수인 원소 제거 ---
[2] [4]
```

```cpp
[](int i) -> bool { return i % 2 == 1; }
```
람다 함수는 위와 같은 꼴로 정의된다. 일반적인 꼴을 살펴보자면
![[Pasted image 20241006021041.png]]
와 같은 형태이다. 위 함수 꼴을 살펴보자면 인자로 `int i` 를 받고, `bool` 을 리턴하는 람다 함수를 정의한 것이다. 리턴 타입을 생략한다면 컴파일러가 알아서 함수 본체에서 `return` 문을 보고 리턴 타입을 추측해준다. (만약에 `return` 경로가 여러 군데여서 추측할 수 없다면 컴파일 오류가 발생할 것이다)

리턴 타입을 생략할 경우
![[Pasted image 20241006021143.png]]
이런 식으로 더 간단히 쓸 수 있습니다. 위 예제의 경우
```cpp
[](int i) { return i % 2 == 1; }
```
로 쓴다면 알아서 "아 `bool` 타입을 리턴하는 함수구나" 라고 컴파일러가 만들어준다.

앞서 람다 함수가 이름이 없는 함수라 했는데 실제로 위를 보면 함수에 이름이 붙어 있지 않다! 즉 임시적으로 함수를 생성한 것이다. 만약에 이 함수를 사용하고 싶다면
```cpp
](int i) { return i % 2 == 1; }(3);  // true
```
와 같이 그냥 바로 호출할 수도 있고
```cpp
auto func = [](int i) { return i % 2 == 1; };
func(4);  // false;
```
람다 함수로 `func` 이라는 함수 객체를 생성한 후에 호출할 수도 있다.

하지만 람다 함수도 말 그대로 함수 이기 때문에 자기 자신만의 스코프를 가진다. 따라서 일반적인 상황이라면 함수 외부에서 정의된 변수들을 사용할 수 없다. 예를 들어서 최대 2 개 원소만 지우고 싶은 경우
```cpp
std::cout << "벡터에서 홀수인 원소 최대 2 개 제거 ---" << std::endl;
int num_erased = 0;
vec.erase(std::remove_if(vec.begin(), vec.end(),
									[](int i) {
										if (num_erased >= 2)
											return false;
										else if (i % 2 == 1) {
											num_erased++;
											return true;
										}
										return false;
									}),
			vec.end());
print(vec.begin(), vec.end());
```
위와 같이 람다 함수 외부에 몇 개를 지웠는지 변수를 정의한 뒤에 사용해야만 하는데 (함수 안에 정의하면 함수 호출될 때 마다 새로 생성되니까!) 문제는 그 변수에 접근할 수 없다는 점이다. 하지만 놀랍게도 람다 함수의 경우 그 변수에 접근할 수 있다. 바로 **캡쳐 목록(capture list)** 을 사용하는 것이다.

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}
int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	std::cout << "처음 vec 상태 ------" << std::endl;
	print(vec.begin(), vec.end());
	
	std::cout << "벡터에서 홀수인 원소 ---" << std::endl;
	int num_erased = 0;
	vec.erase(std::remove_if(vec.begin(), vec.end(),
						[&num_erased](int i) {
							if (num_erased >= 2)
								return false;
							else if (i % 2 == 1) {
								num_erased++;
								return true;
							}
							return false;
						}),
			vec.end());
	print(vec.begin(), vec.end());
}
```
성공적으로 컴파일했다면
```
처음 vec 상태 ------
[5] [3] [1] [2] [3] [4] 
벡터에서 홀수인 원소 ---
[1] [2] [3] [4]
```
위와 같이 캡쳐 목록에는 어떤 변수를 캡쳐할 지 써주면 된다. 위 경우 `num_erased` 를 캡쳐했다. 즉 람다 함수 내에서 `num_erased` 를 마치 같은 스코프 안에 있는 것처럼 사용할 수 있게 된다.

이 때 `num_erased` 앞에 `&` 가 붙어있는데 이는 실제 `num_erased`의 레퍼런스를 캡쳐한다는 의미이다. 즉 함수 내부에서 `num_erased` 의 값을 바꿀 수 있게 된다. 만약에 아래처럼
```cpp
[num_erased](int i){
	if (num_erased >= 2)
		return false;
	else if (i % 2 == 1) {
		num_erased++;
		return true;
	}
	return false;
})
```
`&` 를 앞에 붙이지 않는다면 `num_erased` 의 복사본을 얻게 되는데, 그 복사본의 형태는 `const` 이다. 따라서 위 처럼 함수 내부에서 `num_erased` 의 값을 바꿀 수 없게 된다.
그렇다면 클래스의 멤버 함수 안에서 람다를 사용할 때 멤버 변수들을 참조하려면 어떻게 해야 할까?
```cpp
class SomeClass {
	std::vector<int> vec;
	
	int num_erased;

public:
	SomeClass() {
		vec.push_back(5);
		vec.push_back(3);
		vec.push_back(1);
		vec.push_back(2);
		vec.push_back(3);
		vec.push_back(4);
		
		num_erased = 1;
		
		vec.erase(std::remove_if(vec.begin(), vec.end(),
								[&num_erased](int i) {
									if (num_erased >= 2)
										return false;
									else if (i % 2 == 1) {
										num_erased++;
										return true;
									}
									return false;
								}),
					vec.end());
	}
};
```
예를 들어 위와 같은 예제를 생각해보자. 쉽게 생각해보면 그냥 똑같이 `num_erased` 를 `&` 로 캡쳐해서 람다 함수 안에서 사용할 수 있을 것 같지만 실제로는 컴파일 되지 않는다. 왜냐하면 `num_erased` 가 일반 변수가 아니라 객체에 종속되어 있는 멤버 변수이기 때문이다. 즉 람다 함수는 `num_erased` 를 캡쳐해! 라고 하면 이 `num_erased` 가 이 객체의 멤버 변수가 아니라 그냥 일반 변수라고 생각하게 된다.

이를 해결하기 위해선 직접 멤버 변수를 전달하기 보다는 **`this` 를 전달해주면 된다**.
```cpp
num_erased = 0;

vec.erase(std::remove_if(vec.begin(), vec.end(),
						[this](int i) {
							if (this->num_erased >= 2)
								return false;
							else if (i % 2 == 1) {
								this->num_erased++;
								return true;
							}
							return false;
						}),
			vec.end());
```
위와 같이 `this` 를 복사본으로 전달해서 (참고로 `this` 는 레퍼런스로 전달할 수 없습니다) 함수 안에서 `this` 를 이용해서 멤버 변수들을 참조해서 사용하면 된다.

위에 설명한 경우 말고도 캡쳐 리스트의 사용 방법은 꽤나 많다.
- `[]` : 아무것도 캡쳐 안함
- `[&a, b]` : `a` 는 레퍼런스로 캡쳐하고 `b` 는 (변경 불가능한) 복사본으로 캡쳐
- `[&]` : 외부의 모든 변수들을 레퍼런스로 캡쳐
- `[=]` : 외부의 모든 변수들을 복사본으로 캡쳐

## 원소 수정하기 (transform)

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}

int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	std::cout << "처음 vec 상태 ------" << std::endl;
	print(vec.begin(), vec.end());
	
	std::cout << "벡터 전체에 1 을 더한다" << std::endl;
	std::transform(vec.begin(), vec.end(), vec.begin(), [](int i) { return i + 1; });
	print(vec.begin(), vec.end());
}
```
성공적으로 컴파일했다면
```
처음 vec 상태 ------
[5] [3] [1] [2] [3] [4] 
벡터 전체에 1 을 더한다
[6] [4] [2] [3] [4] [5]
```

transform 함수는 다음과 같은 꼴로 생겼다.
![[Pasted image 20241006021950.png]]

우리가 사용한 예의 경우
```cpp
transform(vec.begin(), vec.end(), vec.begin(), [](int i) { return i + 1; });
```
로 하였으므로 `vec` 의 시작(begin) 부터 끝(end) 까지 각 원소에 `[] (int i) {return i + 1}` 함수를 적용시킨 결과를 `vec.begin()` 부터 저장하게 된다. 즉 결과적으로 각 원소에 1 을 더한 결과로 덮어 씌우게 되는 것이다. 상당히 간단합니다.
한 가지 주의할 점은 값을 저장하는 컨테이너의 크기가 원래의 컨테이너보다 최소한 같거나 커야 된다는 점이다. 예를 들어서 단순하게
```cpp
std::transform(vec.begin(), vec.end(), vec.begin() + 1, [](int i) { return i + 1; });
```
이렇게 썻다고 해보자. [transform](https://modoocode.com/275) 함수는 `vec` 의 처음부터 끝까지 쭈르륵 순회하지만 저장하는 쪽의 반복자는 `vec` 의 두 번째 원소부터 저장하기 때문에 결과적으로 마지막에 한 칸이 모자라서
![[24E5C63359718B1B071A2B.webp]]
위와 같은 오류가 발생하게 된다.

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}

int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	// vec2 에는 6 개의 0 으로 초기화 한다.
	std::vector<int> vec2(6, 0);
	
	std::cout << "처음 vec 과 vec2 상태 ------" << std::endl;
	print(vec.begin(), vec.end());
	print(vec2.begin(), vec2.end());
	
	std::cout << "vec 전체에 1 을 더한 것을 vec2 에 저장 -- " << std::endl;
	std::transform(vec.begin(), vec.end(), vec2.begin(), [](int i) { return i + 1; });
	print(vec.begin(), vec.end());
	print(vec2.begin(), vec2.end());
}
```
성공적으로 컴파일했으면
```
처음 vec 과 vec2 상태 ------
[5] [3] [1] [2] [3] [4] 
[0] [0] [0] [0] [0] [0] 
vec 전체에 1 을 더한 것을 vec2 에 저장 -- 
[5] [3] [1] [2] [3] [4] 
[6] [4] [2] [3] [4] [5]
```

```cpp
std::transform(vec.begin(), vec.end(), vec2.begin(), [](int i) { return i + 1; });
```
위와 같이 `vec` 의 처음부터 끝까지 읽으면서 1씩 더한 결과를 `vec2` 에 저장하게 된다. 간단하다! 물론 저 [transform](https://modoocode.com/275) 함수 하나 덕분에 귀찮게 `for` 문을 쓸 필요도 없어질 뿐더러, 내가 이 코드에서 무슨 일을 하는지 더 간단 명료하게 나타낼 수도 있다.

## 원소를 탐색하는 함수(find, find_if, any_of, all_of 등)

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}

int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	auto result = std::find(vec.begin(), vec.end(), 3);
	std::cout << "3 은 " << std::distance(vec.begin(), result) + 1 << " 번째 원소"
			<< std::endl;
}
```
성공적으로 컴파일했으면
```
3 은 2 번째 원소
```

find 함수는 단순히
```cpp
template <class InputIt, class T>
InputIt find(InputIt first, InputIt last, const T& value)
```
와 같이 생겼는데, `first`부터 `last`까지 쭈르륵 순회하면서 `value` 와 같은 원소가 있는지 확인하고 있으면 이를 가리키는 반복자를 리턴한다. 위 경우
```cpp
auto result = std::find(vec.begin(), vec.end(), 3);
```
`vec` 에서 값이 3과 같은 원소를 찾아서 리턴하게 된다. 반복자에 따라서 `forward_iterator`면 앞에서부터 찾고, `reverse_iterator` 이면 뒤에서부터 거꾸로 찾게 된다. 물론 컨테이너에 중복되는 값이 있더라도 가장 먼저 찾은 것을 리턴한다. 만약에 위 `vec` 에서 모든 3을 찾고 싶다면 아래와 같이 하면 된다.
```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}

int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	auto current = vec.begin();
	while (true) {
		current = std::find(current, vec.end(), 3);
		if (current == vec.end()) break;
		std::cout << "3 은 " << std::distance(vec.begin(), current) + 1
				  << " 번째 원소" << std::endl;
		current++;
	}
}
```
성공적으로 컴파일했다면
```
3 은 2 번째 원소
3 은 5 번째 원소
```

위 처럼 마지막으로 찾은 위치 바로 다음부터 계속 순차적으로 탐색해 나간다면 컨테이너에서 값이 3인 원소들을 모두 찾을 수 있게 된다.

다만 [find](https://modoocode.com/261) 계열의 함수들을 사용할 때 한 가지 주의해야 할 점은, **만약에 컨테이너에서 기본적으로 [find](https://modoocode.com/261) 함수를 지원한다면 이를 사용하는 것이 훨씬 빠르다**. 왜냐하면 알고리즘 라이브러리에서의 [find](https://modoocode.com/261) 함수는 그 컨테이너가 어떠한 구조를 가지고 있는지에 대한 정보가 하나도 없기 때문이다.

예를 들어 `set` 의 경우, `set` 에서 사용하는 [find](https://modoocode.com/261) 함수의 경우 $O(log \space n)$으로 수행될 수 있는데 그 이유는 셋 내부에서 원소들이 정렬되어 있기 때문이다. 또 `unordered_set` 의 경우 [find](https://modoocode.com/261) 함수가 $O(1)$로 수행될 수 있는데 그 이유는 `unordered_set` 내부에서 자체적으로 해시 테이블을 이용해서 원소들을 빠르게 탐색해 나갈 수 있기 때문이다.

하지만 그냥 알고리즘 라이브러리의 [find](https://modoocode.com/261) 함수의 경우 이러한 추가 정보가 있는 것을 하나도 모른채 우직하게 처음부터 하나씩 확인해 나가므로 평범한 $O(n)$으로 처리된다. 따라서 알고리즘 라이브러리의 [find](https://modoocode.com/261) 함수를 사용할 경우 **벡터와 같이 기본적으로 [find](https://modoocode.com/261) 함수를 지원하지 않는 컨테이너에 사용하자**.

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}

int main() {
	std::vector<int> vec;
	vec.push_back(5);
	vec.push_back(3);
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	vec.push_back(4);
	
	auto current = vec.begin();
	while (true) {
		current = std::find_if(current, vec.end(), [](int i) { return i % 3 == 2; });
		if (current == vec.end()) break;
		std::cout << "3 으로 나눈 나머지가 2 인 원소는 : " << *current << " 이다 "
				  << std::endl;
		current++;
	}
}
```
성공적으로 컴파일했다면
```
3 으로 나눈 나머지가 2 인 원소는 : 5 이다 
3 으로 나눈 나머지가 2 인 원소는 : 2 이다
```

[find](https://modoocode.com/261) 함수가 단순한 값을 받았다면 [find_if](https://modoocode.com/263) 함수의 경우 **함수 객체**를 인자로 받아서 그 결과가 참인 원소들을 찾게 된다. 위 경우 3으로 나눈 나머지가 2인 원소들을 컨테이너에서 탐색했다.

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string>
#include <vector>

template <typename Iter>
void print(Iter begin, Iter end) {
	while (begin != end) {
		std::cout << "[" << *begin << "] ";
		begin++;
	}
	std::cout << std::endl;
}

struct User {
	std::string name;
	int level;
	
	User(std::string name, int level) : name(name), level(level) {}
	bool operator==(const User& user) const {
		if (name == user.name && level == user.level) return true;
		return false;
	}
};

class Party {
	std::vector<User> users;

public:
	bool add_user(std::string name, int level) {
		User new_user(name, level);
		if (std::find(users.begin(), users.end(), new_user) != users.end()) {
			return false;
		}
		users.push_back(new_user);
		return true;
	}

	// 파티원 모두가 15 레벨 이상이여야지 던전 입장 가능
	bool can_join_dungeon() {
		return std::all_of(users.begin(), users.end(),
						   [](User& user) { return user.level >= 15; });
	}
	
	// 파티원 중 한명 이라도 19렙 이상이면 특별 아이템 사용 가능
	bool can_use_special_item() {
		return std::any_of(users.begin(), users.end(),
						   [](User& user) { return user.level >= 19; });
	}
};
int main() {
	Party party;
	party.add_user("철수", 15);
	party.add_user("영희", 18);
	party.add_user("민수", 12);
	party.add_user("수빈", 19);
	
	std::cout << std::boolalpha;
	std::cout << "던전 입장 가능 ? " << party.can_join_dungeon() << std::endl;
	std::cout << "특별 아이템 사용 가능 ? " << party.can_use_special_item()
			<< std::endl;
}
```
성공적으로 컴파일했다면
```
던전 입장 가능 ? false
특별 아이템 사용 가능 ? true
```

마지막으로 살펴볼 함수들은 [any_of](https://modoocode.com/258) 와 [all_of](https://modoocode.com/257) 이다. [any_of](https://modoocode.com/258) 는 인자로 받은 범위 안의 모든 원소들 중에서 조건을 하나라도 충족하는 것이 있다면 `true` 를 리턴하고 [all_of](https://modoocode.com/257) 의 경우 모든 원소들이 전부 조건을 충족해야 `true` 를 리턴한다. 즉 [any_of](https://modoocode.com/258) 는 [OR](https://modoocode.com/or) 연산과 비슷하고 [all_of](https://modoocode.com/257) 는 [AND](https://modoocode.com/and) 연산과 비슷하다고 볼 수 있다.

```cpp
bool add_user(std::string name, int level) {
	User new_user(name, level);
	if (std::find(users.begin(), users.end(), new_user) != users.end()) {
		return false;
	}
	users.push_back(new_user);
	return true;
}
```
먼저 간단히 유저들의 정보를 담고 있는 `User` 구조체를 정의했고, 그 `User` 들이 파티를 이룰 때 만들어지는 `Party` 클래스를 정의했다. 그리고 위 `add_user` 함수를 사용하면 파티원을 추가할 수 있다. 물론 중복되는 파티원이 없도록 벡터에 원소를 추가하기 전에 확인한다.

```cpp
// 파티원 모두가 15 레벨 이상이여야지 던전 입장 가능
bool can_join_dungeon() {
	return std::all_of(users.begin(), users.end(),
						[](User& user) { return user.level >= 15; });
}
```
따라서 이 파티가 어떤 던전에 참가하고 싶은 경우 모든 파티원의 레벨이 `15` 이상 이어야 하므로 위와 같이 [all_of](https://modoocode.com/257) 함수를 사용해서 모든 원소들이 조건에 만족하는지 확인할 수 있다. 위 경우 민수가 `12` 레벨이여서 `false` 가 리턴될 것이다.

```cpp
// 파티원 중 한명 이라도 19렙 이상이면 특별 아이템 사용 가능
bool can_use_special_item() {
	return std::any_of(users.begin(), users.end(),
						[](User& user) { return user.level >= 19; });
}
```
비슷하게도 한 명만 조건을 만족해도 되는 경우 위와 같이 [any_of](https://modoocode.com/258) 함수를 사용하면 간단히 처리할 수 있다.