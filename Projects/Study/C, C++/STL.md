
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