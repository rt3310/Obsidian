
## vector

### 맨 뒤에 원소를 추가하는 작업
맨 뒤에 원소를 추가하는 작업은 엄밀히 말하자면 amortized $O(1)$이라고 한다. (amortized: 분할상환)

왜냐면 보통은 `vector`의 경우 현재 가지고 있는 원소의 개수보다 더 많은 공간을 할당해 놓고 있는다. 예를 들어 현재 `vector`에 있는 원소의 개수가 10개라면 이미 20개를 저장할 수 있는 공간을 미리 할당해 놓게 된다. 따라서 만약에 뒤에 새로운 원소를 추가하게 된다면 새롭게 메모리를 할당할 필요 없이, 그냥 이미 할당된 공간에 그 원소를 쓰기만 하면 된다. 따라서 대부분의 경우 $O(1)$로 `vector` 맨 뒤에 새로운 원소를 추가하거나 지울 수 있다.

문제가 되는 상황은 할당된 공간을 다 채웠을 때이다. 이 때는 어쩔 수 없이, 새로운 큰 공간을 다시 할당하고 기존의 원소들을 복사하는 수 밖에 없다. 따라서 이 경우 $n$개의 원소를 모두 복사해야 하기 때문에 $O(n)$으로 수행된다. 하지만 이 $O(n)$으로 수행되는 경우가 매우 드물기 때문에, 전체적으로 평균을 내보았을 때 $O(1)$로 수행됨을 알 수 있다. 이렇기에 amortized $O(1)$이라고 부르게 된다.

### 임의의 위치에 원소를 추가하는 작업
물론 `vector`라고 만능은 아니다. 맨 뒤에 원소를 추가하거나 제거하는 것은 빠르지만, 임의의 위치에 원소를 추가하거나 제거하는 것은 $O(n)$으로 느리다. 왜냐하면 어떤 자리에 새로운 원소를 추가하거나 뺄 경우 그 뒤에 오는 원소들을 한 칸씩 이동시켜 주어야 하기 때문이다. 따라서 이는 n번의 복사가 필요하게 된다.

- 임의의 위치 원소 접근(`[]`, `at`): $O(1)$
- 맨 뒤에 원소 추가 및 제거(`push_back`, `pop_back`): amortized $O(1)$, 평균적으로 $O(1)$이지만 최악의 경우 $O(n)$
- 임의의 위치 원소 추가 및 제거(`insert`, `erase`): $O(n)$

위처럼 어떠한 작업을 하냐에 따라서 속도 차가 매우 크기 때문에, C++ 표준 라이브러리를 잘 사욯아기 위해서는 내가 이 컨테이너를 어떠한 작업을 위해 사용하는지 정확히 인지하고, 적절한 컨테이너를 골라야 한다.

## iterator

반복자는 포인터처럼 사용할 수 있다. 실제로 현재 반복자가 가리키는 원소의 값을 보고 싶다면
```cpp
std::cout << *itr << std::endl;
```
포인터로 `*` 를 해서 가리키는 주소 값의 값을 보았던 것처럼, `*` 연산자를 이용해서 `itr` 이 가리키는 원소를 볼 수 있다. **물론 `itr` 은 실제 포인터가 아니고 `*` 연산자를 오버로딩해서 마치 포인터처럼 동작하게 만든 것**이다. `*` 연산자는 `itr` 이 가리키는 원소의 레퍼런스를 리턴한다.

```cpp
std::vector<int>::iterator itr = vec.begin() + 2;
std::cout << "3 번째 원소 :: " << *itr << std::endl;
```
또한 반복자 역시 `+` 연산자를 통해서 그 만큼 떨어져 있는 원소를 가리키게 할 수도 있다. (그냥 배열을 가리키는 포인터와 정확히 똑같이 동작한다고 생각하면 된다)

참고로 템플릿 버전의 경우,
```cpp
for (typename std::vector<T>::iterator itr = vec.begin(); itr != vec.end(); ++itr) {
```
와 같이 앞에 `typename` 을 추가해줘야만 한다. 그 이유는, `iterator` 가 `std::vector<T>` 의 의존 타입이기 때문이다. [의존 타입](https://modoocode.com/222?category=361027)

```cpp
// vec[2] 앞에 15 추가
vec.insert(vec.begin() + 2, 15);
```
insert 시, 위처럼 인자로 반복자를 받고, 그 반복자 앞에 원소를 추가해준다. 위 경우 `vec.begin() + 2` 앞에 `15` 를 추가하므로 `10, 20, 30, 40` 에서 `10, 20, 15, 30, 40` 이 된다.

```cpp
vec.erase(vec.begin() + 3);
print_vector(vec);
```
[erase](https://modoocode.com/240) 도 인자로 반복자를 받고, 그 반복자가 가리키는 원소를 제거한. 위 경우 4번째 원소인 30 이 지워질 것이다. 물론 [insert](https://modoocode.com/238) 과 [erase](https://modoocode.com/240) 함수 모두 `O(n)` 으로 느린편이다.

참고로 `vector` 에서 반복자로 [erase](https://modoocode.com/240) 나 [insert](https://modoocode.com/238) 함수를 사용할 때 주의해야 할 점이 있다.
```cpp
#include <iostream>
#include <vector>

template <typename T>
void print_vector(std::vector<T>& vec) {
	// 전체 벡터를 출력하기
	std::cout << "[ ";
	for (typename std::vector<T>::iterator itr = vec.begin(); itr != vec.end(); ++itr) {
		std::cout << *itr << " ";
	}
	std::cout << "]";
}
int main() {
	std::vector<int> vec;
	vec.push_back(10);
	vec.push_back(20);
	vec.push_back(30);
	vec.push_back(40);
	vec.push_back(20);
	
	std::cout << "처음 벡터 상태" << std::endl;
	print_vector(vec);
	
	std::vector<int>::iterator itr = vec.begin();
	std::vector<int>::iterator end_itr = vec.end();
	
	for (; itr != end_itr; ++itr) {
		if (*itr == 20) {
			vec.erase(itr);
		}
	}
	
	std::cout << "값이 20 인 원소를 지운다!" << std::endl;
	print_vector(vec);
}
```
컴파일 후 실행했다면 아래와 같은 오류가 발생한다.
![[2554D949595B4BB61B3489.webp]]
왜 이런 오류가 발생하는 것일까?

문제는 바로 위 코드에서 발생합니다. 컨테이너에 원소를 추가하거나 제거하게 되면 기존에 사용하였던 모든 반복자들을 사용할 수 없게 된다. 다시 말해 위 경우 `vec.erase(itr)`을 수행하게 되면 더이상 `itr`은 유효한 반복자가 아니게 되는 것이다. 또한 `end_itr` 역시 무효화된다.

따라서 `itr != end_itr` 이 영원히 성립되며 무한루프에 빠지게 되어 위와 같은 오류가 발생한다.

```cpp
std::vector<int>::iterator itr = vec.begin();

for (; itr != vec.end(); ++itr) {
	if (*itr == 20) {
		vec.erase(itr);
		itr = vec.begin();
	}
}
```
성공적으로 컴파일 했다면
![[277A6C33595B4E5831E24D.webp]]
와 같이 제대로 값이 20 인 원소만 지워졌음을 알 수 있다.

사실 생각해보면 위 바뀐 코드는 꽤나 비효율적임을 알 수 있다. 왜냐하면 20인 원소를 지우고, 다시 처음으로 돌아가서 원소들을 찾고 있기 때문이다. 그냥 20 인 원소 바로 다음 위치 부터 찾아나가면 될텐데 말이다.
```cpp
for (std::vector<int>::size_type i = 0; i != vec.size(); i++) {
	if (vec[i] == 20) {
		vec.erase(vec.begin() + i);
		i--;
	}
}
```
그렇다면 아예 위처럼 굳이 반복자를 쓰지 않고 [erase](https://modoocode.com/240) 함수에만 반복자를 바로 만들어서 전달하면 된다.

```cpp
vec.erase(vec.begin() + i);
```
를 하게 되면 `vec[i]` 를 가리키는 반복자를 [erase](https://modoocode.com/240) 에 전달할 수 있다. 하지만 사실 위 방법은 그리 권장하는 방법은 아니다. 기껏 원소에 접근하는 방식을 반복자를 사용하는 것으로 통일했는데, 위 방법은 이를 모두 깨버리고 그냥 기존의 배열처럼 정수형 변수 `i` 로 원소에 접근하는 것이기 때문이다.

하지만 후에 C++ 알고리즘 라이브러리에 대해 배우면서 이 문제를 깔끔하게 해결 하는 방법에 대해 다루도록 할 것입니다. 일단 임시로는 위 방법처럼 처리하도록 하자

### const_iterator
`vector` 에서 지원하는 반복자로 `const_iterator` 가 있다. 이는 마치 `const` 포인터를 생각하면 된다. 즉, `const_iterator` 의 경우 가리키고 있는 원소의 값을 바꿀 수 없다. 예를 들어서
```cpp
#include <iostream>
#include <vector>

template <typename T>
void print_vector(std::vector<T>& vec) {
	// 전체 벡터를 출력하기
	for (typename std::vector<T>::iterator itr = vec.begin(); itr != vec.end(); ++itr) {
		std::cout << *itr << std::endl;
	}
}
int main() {
	std::vector<int> vec;
	vec.push_back(10);
	vec.push_back(20);
	vec.push_back(30);
	vec.push_back(40);
	
	std::cout << "초기 vec 상태" << std::endl;
	print_vector(vec);
	
	// itr 은 vec[2] 를 가리킨다.
	std::vector<int>::iterator itr = vec.begin() + 2;
	
	// vec[2] 의 값을 50으로 바꾼다.
	*itr = 50;
	
	std::cout << "---------------" << std::endl;
	print_vector(vec);
	
	std::vector<int>::const_iterator citr = vec.cbegin() + 2;
	
	// 상수 반복자가 가리키는 값은 바꿀수 없다. 불가능!
	*citr = 30;
}
```
컴파일 했다면
![[Pasted image 20241005171238.png]]
와 같이, `const` 반복자가 가리키고 있는 값은 바꿀 수 없다고 오류가 발생한다. 주의할 점은, `const` 반복자의 경우
```cpp
std::vector<int>::const_iterator citr = vec.cbegin() + 2;
```
와 같이 `cbegin()` 과 `cend()` 함수를 이용하여 얻을 수 있다. 많은 경우 반복자의 값을 바꾸지 않고 참조만 하는 경우가 많으므로, `const_iterator` 를 적절히 이용하는 것이 좋다.

### reverse iterator
`vector` 에서 지원하는 반복자 중 마지막 종류로 역반복자(reverse iterator)가 있다. 이는 반복자와 똑같지만 벡터 뒤에서부터 앞으로, 거꾸로 간다는 특징이 있다.
```cpp
#include <iostream>
#include <vector>

template <typename T>
void print_vector(std::vector<T>& vec) {
	// 전체 벡터를 출력하기
	for (typename std::vector<T>::iterator itr = vec.begin(); itr != vec.end(); ++itr) {
		std::cout << *itr << std::endl;
	}
}
int main() {
	std::vector<int> vec;
	vec.push_back(10);
	vec.push_back(20);
	vec.push_back(30);
	vec.push_back(40);
	
	std::cout << "초기 vec 상태" << std::endl;
	print_vector(vec);
	
	std::cout << "역으로 vec 출력하기!" << std::endl;
	// itr 은 vec[2] 를 가리킨다.
	std::vector<int>::reverse_iterator r_iter = vec.rbegin();
	for (; r_iter != vec.rend(); r_iter++) {
		std::cout << *r_iter << std::endl;
	}
}
```
성공적으로 컴파일 했다면
```
초기 vec 상태
10
20
30
40
역으로 vec 출력하기!
40
30
20
10
```
와 같이 역으로 벡터의 원소들을 출력할 수 있다.
![[275B1D3D595B2F4011531A.webp]]
이전에 반복자의 `end()` 가 맨 마지막 원소의 바로 뒤를 가리켰던 것처럼, 역반복자의 `rend()` 역시 맨 앞 원소의 바로 앞을 가리키게 된다. 또한 반복자의 경우 값이 증가하면 뒤쪽 원소로 가는 것처럼, 역반복자의 경우 값이 증가하면 앞쪽 원소로 가게 된다.

또 반복자가 상수 반복자가 있는 것처럼 역반복자 역시 상수 역반복자가 있다. 그 타입은 `const_reverse_iterator` 타입이고, `crbegin(), crend()` 로 얻을 수 있다.

역반복자를 사용하는 것은 매우 중요하다.
```cpp
#include <iostream>
#include <vector>

int main() {
	std::vector<int> vec;
	vec.push_back(1);
	vec.push_back(2);
	vec.push_back(3);
	
	// 끝에서 부터 출력하기
	for (std::vector<int>::size_type i = vec.size() - 1; i >= 0; i--) {
		std::cout << vec[i] << std::endl;
	}
	
	return 0;
}
```
성공적으로 컴파일 했다면
```
3
2
1
// ... (생략) ...
0
0
0
1
0
593
0
0
[1]    22180 segmentation fault (core dumped)  ./test
```
와 같이 오류가 발생하게 된다.
맨 뒤의 원소부터 제대로 출력하는 코드 같은데 왜 이런 문제가 발생했까?
그 이유는 `vector` 의 `index` 를 담당하는 타입이 부호 없는 정수이기 때문이다. 따라서 `i` 가 0 일 때 `i --` 를 하게 된다면 -1 이 되는 것이 아니라, 해당 타입에서 가장 큰 정수가 돼버리게 된다.
따라서 `for` 문이 영원히 종료할 수 없게 된다.

이 문제를 해결하기 위해서는 부호 있는 정수로 선언해야 하는데, 이 경우 `vector` 의 `index` 타입과 일치하지 않아서 타입 캐스팅을 해야 한다는 문제가 발생하게 된다.
따라서 가장 현명한 선택으로는 역으로 원소를 참조하고 싶다면, 역반복자를 사용하는 것이다.

## 리스트 (list)

리스트(`list`) 의 경우 양방향 연결 구조를 가진 자료형이라 볼 수 있다.
![[246A0A4B595B396939AF3D.webp]]
따라서 `vector` 와는 달리 임의의 위치에 있는 원소에 접근을 바로 할 수 없다. `list` 컨테이너 자체에서는 시작 원소와 마지막 원소의 위치만을 기억하기 때문에, 임의의 위치에 있는 원소에 접근하기 위해서는 하나씩 링크를 따라가야 한다.

그래서 리스트에는 아예 `[]` 나 `at` 함수가 아예 정의되어 있지 않다.

물론 리스트의 장점이 없는 것은 아니다. `vector` 의 경우 맨 뒤를 제외하고는 임의의 위치에 원소를 추가하거나 제거하는 작업이 $O(n)$이였지만 리스트의 경우 $O(1)$로 매우 빠르게 수행될 수 있다. 왜냐하면 원하는 위치 앞과 뒤에 있는 링크값만 바꿔주면 되기 때문이다.

한 가지 재미있는점은 리스트의 반복자의 경우 다음과 같은 연산밖에 수행할 수 없다.
```cpp
itr++
itr-- // --itr도 된다.
```
다시 말해
```cpp
itr + 5 // 불가능
```
와 같이 임의의 위치에 있는 원소를 가리킬 수 없다는 것이다. 반복자는 오직 한 칸씩 밖에 움직일 수 없다.

이와 같은 이유는 `list` 의 구조를 생각해보면 알 수 있다. 앞서 말했듯이 리스트는 왼쪽 혹은 오른쪽을 가리키고 있는 원소들의 모임으로 이루어져 있기 때문에, 한 번에 한 칸씩 밖에 이동할 수 없다. 즉, **메모리 상에서 원소들이 연속적으로 존재하지 않을 수 있다는 뜻**이다. 반면에 **벡터의 경우 메모리 상에서 연속적으로 존재**하기 때문에 쉽게 임의의 위치에 있는 원소를 참조할 수 있다.

이렇게 리스트에서 정의되는 반복자의 타입을 보면 `BidirectionalIterator` 타입임을 알 수 있다. 이름에서도 알 수 있듯이 **양방향으로 이동할 수 있되, 한 칸 씩 밖에 이동할 수 없다**. 반면에 벡터에서 정의되는 반복자의 타입은 `RandomAccessIterator` 타입이다.

즉, 임의의 위치에 접근할 수 있는 반복자이다
(참고로 `RandomAccessIterator` 는 `BidirectionalIterator` 를 상속받고 있다)
```cpp
#include <iostream>
#include <list>

template <typename T>
void print_list(std::list<T>& lst) {
	std::cout << "[ ";
	// 전체 리스트를 출력하기 (이 역시 범위 기반 for 문을 쓸 수 있습니다)
	for (const auto& elem : lst) {
		std::cout << elem << " ";
	}
	std::cout << "]" << std::endl;
}
int main() {
	std::list<int> lst;
	
	lst.push_back(10);
	lst.push_back(20);
	lst.push_back(30);
	lst.push_back(40);
	
	std::cout << "처음 리스트의 상태 " << std::endl;
	print_list(lst);
	
	for (std::list<int>::iterator itr = lst.begin(); itr != lst.end(); ++itr) {
		// 만일 현재 원소가 20 이라면
		// 그 앞에 50 을 집어넣는다.
		if (*itr == 20) {
			lst.insert(itr, 50);
		}
	}
	
	std::cout << "값이 20 인 원소 앞에 50 을 추가 " << std::endl;
	print_list(lst);
	
	for (std::list<int>::iterator itr = lst.begin(); itr != lst.end(); ++itr) {
		// 값이 30 인 원소를 삭제한다.
		if (*itr == 30) {
			lst.erase(itr);
			break;
		}
	}
	
	std::cout << "값이 30 인 원소를 제거한다" << std::endl;
	print_list(lst);
}
```
성공적으로 컴파일 했다면
```
처음 리스트의 상태 
[ 10 20 30 40 ]
값이 20 인 원소 앞에 50 을 추가 
[ 10 50 20 30 40 ]
값이 30 인 원소를 제거한다
[ 10 50 20 40 ]
```

```cpp
for (std::list<int>::iterator itr = lst.begin(); itr != lst.end(); ++itr) {
	// 만일 현재 원소가 20 이라면
	// 그 앞에 50 을 집어넣는다.
	if (*itr == 20) {
		lst.insert(itr, 50);
	}
}
```
앞서 설명했지만 리스트의 반복자는 `BidirectionalIterator` 이기 때문에 `++` 과 `--` 연산만 사용 가능하다. 따라서 위처럼 `for` 문으로 하나 하나 원소를 확인해보는것은 가능하다. `vector` 와는 다르게 [insert](https://modoocode.com/238) 작업은 `O(1)` 로 매우 빠르게 실행된다.

```cpp
for (std::list<int>::iterator itr = lst.begin(); itr != lst.end(); ++itr) {
	// 값이 30 인 원소를 삭제한다.
	if (*itr == 30) {
		lst.erase(itr);
		break;
	}
}
```
마찬가지로 [erase](https://modoocode.com/240) 함수를 이용하여 원하는 위치에 있는 원소를 지울 수도 있다. **리스트의 경우는 벡터와는 다르게, 원소를 지워도 반복자가 무효화되지 않는다. 왜냐하면, 각 원소들의 주소값들은 바뀌지 않기 때문이다**

## 덱 (deque - double ended queue)

마지막으로 살펴볼 컨테이너는 덱(`deque`) 이라고 불리는 자료형이다. 덱은 벡터와 비슷하게 $O(1)$로 임의의 위치의 원소에 접근할 수 있으며 맨 뒤에 원소를 추가/제거 하는 작업도 $O(1)$ 으로 수행할 수 있다. 뿐만 아니라 벡터와는 다르게 맨 앞에 원소를 추가/제거 하는 작업 까지도 $O(1)$ 으로 수행 가능하다.

임의의 위치에 있는 원소를 제거/추가 하는 작업은 벡터와 마찬가지로 $O(n)$ 으로 수행 가능하다. 뿐만 아니라 그 속도도 벡터보다 더 빠르다.

그렇다면 덱이 벡터에 비해 모든 면에서 비교 우위에 있는 걸까? 안타깝게도 벡터와는 다르게 **덱의 경우 원소들이 실제로 메모리 상에서 연속적으로 존재하지는 않는다**. 이 때문에 **원소들이 어디에 저장되어 있는 지에 대한 정보를 보관하기 위해 추가적인 메모리가 더 필요로 한다**. (실제 예로, 64 비트 `libc++` 라이브러리의 경우 1 개의 원소를 보관하는 덱은 그 원소 크기에 비해 8배나 더 많은 메모리를 필요로 합니다).

**즉, 덱은 실행 속도를 위해 메모리를 (많이) 희생하는 컨테이너라 보면 된다**.

![[245FC94C595B5F9B133E4E.webp]]위 그림은 덱이 어떠한 구조를 가지는지 보여준다. 일단, 벡터와는 다르게 원소들이 메모리에 연속되어 존재하는 것이 아니라 일정 크기로 잘려서 각각의 블록 속에 존재한다. 따라서 **이 블록들이 메모리 상에 어느 곳에 위치하여 있는지 저장하기 위해서 각각의 블록들의 주소를 저장하는 벡터가 필요하다**.
참고로 이 벡터는 기존의 벡터와는 조금 다르게, **새로 할당 시에 앞쪽 및 뒤쪽 모두에 공간을 남겨놓게 된다**. (벡터의 경우 뒤쪽에만 공간이 남았다) 따라서 이를 통해 맨 앞과 맨 뒤에 $O(1)$의 속도로 [insert](https://modoocode.com/238) 및 [erase](https://modoocode.com/240) 를 수행할 수 있는 것입니다. 그렇다면 왜 덱이 벡터보다 원소를 삽입하는 작업이 더 빠른 것일까?
![[236D8137595B617B02463F.webp]]
위와 같은 상황에서 `deq.push_back(10)` 을 수행하였다고 생각해보자.
![[244E834B595B642028B977.webp]]
그렇다면 단순히 새로운 블록을 만들어서 뒤에 추가되는 원소를 넣어주면 된다. 즉, 기존의 원소들을 복사할 필요가 전혀 없다는 의미이다. 반면에 벡터의 경우
![[23015636595B647418F5CE.webp]]
위 그림에서도 잘 알 수 있듯이, 만약에 기존에 할당한 메모리가 꽉 차면 모든 원소들을 새로운 공간에 복사해야 한다. 따라서 평균적으로 덱이 벡터보다 더 빠르게 작동한다. (물론 덱의 경우 블록 주소를 보관하는 벡터가 꽉 차게 되면 새로운 공간에 모두 복사해야 한다.

하지만 블록 주소의 개수는 전체 원소 개수보다 적고 (위 경우 N / 5 가 되겠다. 왜냐하면 각 블록에 원소가 5개 씩 있으므로), 대체로 벡터에 저장되는 객체들의 크기가 주소 값의 크기보다 크기 때문에 복사 속도가 훨씬 빠르다.

앞서 말했듯이 덱 역시 벡터 처럼 임의의 위치에 원소에 접근할 수 있으므로 `[]` 와 `at` 함수를 제공하고 있고, 반복자 역시 `RandomAccessIterator` 타입이고 벡터랑 정확히 동일한 방식으로 작동한다.

## 어떤 컨테이너를 사용해야 하는가?

어떠한 컨테이너를 사용할지는 전적으로 이 컨테이너를 가지고 어떠한 작업들을 많이 하냐에 달려있다.
- 일반적인 상황에서는 그냥 벡터를 사용한다 (거의 만능이다!)
- 만약에 맨 끝이 아닌 중간에 원소들을 추가하거나 제거하는 일을 많이 하고, 원소들을 순차적으로만 접근 한다면 리스트를 사용한다.
- 만약에 맨 처음과 끝 모두에 원소들을 추가하는 작업을 많이 하면 덱을 사용한다.

참고적으로 $O(1)$ 으로 작동한다는 것은 언제나 이론적인 결과일 뿐이며 실제로 프로그램을 짜게 된다면, $O(log \space n)$ 이나 $O(n)$ 보다도 느릴 수 있다. ($n$ 의 크기에 따라서) 따라서 속도가 중요한 환경이라면 적절한 벤치마크를 통해서 성능을 가늠해 보는 것도 좋다.