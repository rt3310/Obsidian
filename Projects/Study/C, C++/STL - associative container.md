
## 셋 (set)

```cpp
#include <iostream>
#include <set>

template <typename T>
void print_set(std::set<T>& s) {
	// 셋의 모든 원소들을 출력하기
	std::cout << "[ ";
	for (typename std::set<T>::iterator itr = s.begin(); itr != s.end(); ++itr) {
		std::cout << *itr << " ";
	}
	std::cout << " ] " << std::endl;
}
int main() {
	std::set<int> s;
	s.insert(10);
	s.insert(50);
	s.insert(20);
	s.insert(40);
	s.insert(30);
	
	std::cout << "순서대로 정렬되서 나온다" << std::endl;
	print_set(s);
	
	std::cout << "20 이 s 의 원소인가요? :: ";
	auto itr = s.find(20);
	if (itr != s.end()) {
		std::cout << "Yes" << std::endl;
	} else {
		std::cout << "No" << std::endl;
	}
	
	std::cout << "25 가 s 의 원소인가요? :: ";
	itr = s.find(25);
	if (itr != s.end()) {
		std::cout << "Yes" << std::endl;
	} else {
		std::cout << "No" << std::endl;
	}
}
```
성공적으로 컴파일 했다면
```
순서대로 정렬되서 나온다
[ 10 20 30 40 50  ] 
20 이 s 의 원소인가요? :: Yes
25 가 s 의 원소인가요? :: No
```

셋에 원소를 추가하기 위해서는 시퀀스 컨테이너 처럼 [insert](https://modoocode.com/238) 함수를 사용하면 된다. 한 가지 다른점은, 시퀀스 컨테이너 처럼 '어디에' 추가할지에 대한 정보가 없다는 점이다. 시퀀스 컨테이너가 상자 하나에 원소를 한 개씩 담고, 각 상자에 번호를 매긴 것이라면, 셋은 그냥 큰 상자 안에 모든 원소들을 쑤셔 넣은 것이라 보면 된다. 그 상자 안에 원소가 어디에 있는지는 중요한게 아니고, 그 상자 안에 원소가 '있냐/없냐' 만이 중요한 정보이다.

셋에 원소를 추가하거나 지우는 작업은 $O(logN)$에 처리된다. 시퀀스 컨테이너의 경우 임의의 원소를 지우는 작업이 $O(N)$으로 수행되었다는 점을 생각하면 훨씬 빠르다고 볼 수 있다.

```cpp
template <typename T>
void print_set(std::set<T>& s) {
	// 셋의 모든 원소들을 출력하기
	std::cout << "[ ";
	for (typename std::set<int>::iterator itr = s.begin(); itr != s.end(); ++itr) {
		std::cout << *itr << " ";
	}
	std::cout << " ] " << std::endl;
}
```
셋 역시 셋에 저장되어 있는 원소들에 접근하기 위해 반복자를 제공하며, 이 반복자는 `BidirectionalIterator`이다. 즉, 시퀀스 컨테이너의 리스트처럼 임의의 위치에 있는 원소에 접근하는 것은 불가능하고 순차적으로 하나 씩 접근하는 것 밖에 할 수 없다.

한 가지 흥미로운 점은 셋에 원소를 넣었을 때 `10 -> 50 -> 20 -> 40 -> 30` 으로 넣었지만 실제로 반복자로 원소들을 모두 출력했을 때 나온 순서는 `10 -> 20 -> 30 -> 40 -> 50` 순으로 나왔다는 점이다. 다시 말해 셋의 경우 **내부에 원소를 추가할 때 정렬된 상태를 유지하며 추가**한다.

앞서 셋을 큰 상자라 생각하고 그 안에 원소들을 쑤셔 넣은 것이라 했는데, 실제로 마구 쑤셔넣지는 않고 **순서를 지키면서 쑤셔 넣는다**. 이 때문에 시퀀스 컨테이너와는 다르게 원소를 추가하는 작업이 $O(logN)$으로 진행된다.
또한 **셋의 진가는 앞서 말했듯이 원소가 있냐 없냐를 확인할 때 드러난다**.
```cpp
std::cout << "20 이 s 의 원소인가요? :: ";
auto itr = s.find(20);
if (itr != s.end()) {
	  std::cout << "Yes" << std::endl;
} else {
	  std::cout << "No" << std::endl;
}
```
셋에는 [find](https://modoocode.com/261) 함수가 제공되며, 이 [find](https://modoocode.com/261) 함수를 통해 이 셋에 원소가 존재하는지 아닌지 확인할 수 있다.
만일 해당하는 원소가 존재한다면 이를 가리키는 반복자를 리턴하고 (`std::set<>::iterator` 타입) 만일 존재하지 않는다면 `s.end()` 를 리턴하게 된다.

만일 벡터였다면 원소가 존재하는지 아닌지 확인하기 위해 벡터의 처음부터 끝까지 하나씩 비교해가면서 찾았어야 했을 것이다. 만일 원소가 없었더라면 벡터에 있는 모든 원소를 확인하였을 것이다 (즉 벡터에서 [find](https://modoocode.com/261) 는 $O(N)$이라 볼 수 있다).

하지만 셋의 경우 놀랍게도 $O(logN)$ 으로 원소가 존재하는지 확인할 수 있다. 이것이 가능한 이유는 셋 내부적으로 원소들이 정렬된 상태를 유지하기 때문이다.
따라서 20을 찾았을 때 `Yes` 가 나오고 셋에 없는 원소인 25를 찾는다면 `No` 가 출력됩니다.

셋이 이러한 방식으로 작업을 수행할 수 있는 이유는 바로 내부적으로 트리 구조로 구성되어 있기 때문이다.
![[22108538595C295C210FFE.webp]]
위 그림은 흔히 볼 수 있는 트리 구조를 나타낸다. 각각의 원소들은 트리의 각 노드들에 저장되어 있고, 다음과 같은 규칙을 지키고 있다.
- 왼쪽에 오는 모든 노드들은 나보다 작다
- 오른쪽에 있는 모든 노드들은 나보다 크다
예를 들어 오른쪽의 30을 살펴보자 (위 그림에서 점선으로 표시한 부분). 30 왼쪽에 오는 노드는 25로 30보다 작고, 오른쪽에 오는 노드들은 33, 45, 60 으로 모두 30 보다 큽니다. 어떤 노드들을 살펴보아도 이러한 규칙을 지키고 있음을 알 수 있다.

그렇다면 위 구조에서 25 를 찾으려면 어떻게 할까?
1. 일단 최상위 노드 (루트 노드라 합니다) 와 비교 : 25 > 20 → 오른쪽 노드로 간다
2. 30 과 비교 : 25 < 30 → 왼쪽 노드로 간 30 과 비교 : 25 < 30 → 왼쪽 노드로 간다
3. 25 와 비교 : 25 == 25 → 당첨 25 와 비교 : 25 == 25 → 당첨!
전체 원소 개수는 8개 이지만, 단 3번의 비교로 원소를 정확히 찾을 수 있다.

그렇다면 12 를 찾으려면 어떻게 할까? 참고로 12 는 위 셋에 들어있지 않은 원소이다.
1. 루트 노드와 비교 : 12 < 20 → 왼쪽 노드로 간다
2. 15 와 비교 : 12 < 15 → 왼쪽 노드로 간 15 와 비교 : 12 < 15 → 왼쪽 노드로 간다
3. 10 과 비교 : 12 > 10 → 오른쪽 노드로 가야하지만 오른쪽에 아무것도 없다. 따라서 이 원소는 존재하지 않는다.
만일 벡터였다면 원소들을 처음부터 끝까지 확인해봐야 했지만, 셋의 경우 단 3번의 비교만으로 12 가 셋에 존재하는지 아닌지 여부를 판단할 수 있었다.
![[25566243595C2B600E335B.webp]]
원소를 검색하는데 필요한 횟수는 트리의 높이와 정확히 일치한다. 즉, 15는 단 2번의 비교로 찾아낼 수 있고, 맨 밑에 있는 60 이나 33 의 경우 총 4번의 비교가 필요하다. 따라서, 트리의 경우 최대한 모든 노드들을 꽉 채우는 것이 중요하다. 예를 들어
![[25339935595C2C382057B8.webp]]
어쩌다 보니 트리가 위처럼 되어버렸다면 사실상 시퀀스 컨테이너와 검색 속도가 동일하다. 위와 같이 한쪽으로 아예 치우쳐버린 트리를 균형잡히지 않은 트리(unbalanced tree) 라고 부른다. 실제 셋의 구현을 보면 위와 같은 상황이 발생하지 않도록 앞서 말한 두 개의 단순한 규칙 보다 더 많은 규칙들을 도입해서 트리를 항상 균형 잡히도록 유지하고 있다.

따라서 셋의 구현 상 $O(log \space N)$$으로 원소를 검색할 수 있다는 것이 보장된다.
([대부분의 셋 구현에서 사용하고 있는 트리 구조](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree))

또한 셋의 중요한 특징으로 바로 셋 안에는 중복된 원소들이 없다는 점이 있다.
```cpp
#include <iostream>
#include <set>

template <typename T>
void print_set(std::set<T>& s) {
	// 셋의 모든 원소들을 출력하기
	std::cout << "[ ";
	for (const auto& elem : s) {
		std::cout << elem << " ";
	}
	std::cout << " ] " << std::endl;
}

int main() {
	std::set<int> s;
	s.insert(10);
	s.insert(20);
	s.insert(30);
	s.insert(20);
	s.insert(10);
	
	print_set(s);
}
```
성공적으로 컴파일했다면
```
[ 10 20 30 ]
```
과 같이 나온다. 분명히
```cpp
s.insert(10);
s.insert(20);
s.insert(30);
s.insert(20);
s.insert(10);
```
위와 같이 10 과 20 을 두 번씩 넣었지만 실제로는 한 번씩 밖에 나오지 않는다. 이는 셋 자체적으로 이미 같은 원소가 있다면 이를 [insert](https://modoocode.com/238) 하지 않기 때문이다. 따라서 마지막 두 [insert](https://modoocode.com/238) 작업은 무시되었을 것이다.

참고로 시퀀스 컨테이너들과 마찬가지로 `set` 역시 범위 기반 for 문을 지원한다. 원소들의 접근 순서는 반복자를 이용해서 접근하였을 때와 동일하다.

만약에 중복된 원소를 허락하고 싶다면 멀티셋(multiset)을 사용하면 된다.

### 클래스 객체를 set에 넣고 싶을 때
위와 같이 기본 타입들 말고, 직접 만든 클래스의 객체를 셋의 원소로 사용할 때 한 가지 주의해야 할 점이 있다. 아래는 할 일 (Todo) 목록을 저장하기 위해 셋을 사용하는 예시이다. `Todo` 클래스는 2개를 멤버 변수로 가지는데 하나는 할 일의 중요도이고, 하나는 해야 할 일의 설명이다.
```cpp
#include <iostream>
#include <set>
#include <string>

template <typename T>
void print_set(std::set<T>& s) {
	// 셋의 모든 원소들을 출력하기
	std::cout << "[ ";
	for (const auto& elem : s) {
		std::cout << elem << " " << std::endl;
	}
	std::cout << " ] " << std::endl;
}

class Todo {
	int priority;  // 중요도. 높을 수록 급한것!
	std::string job_desc;

public:
	Todo(int priority, std::string job_desc)
		: priority(priority), job_desc(job_desc) {}
};

int main() {
	std::set<Todo> todos;
	
	todos.insert(Todo(1, "농구 하기"));
	todos.insert(Todo(2, "수학 숙제 하기"));
	todos.insert(Todo(1, "프로그래밍 프로젝트"));
	todos.insert(Todo(3, "친구 만나기"));
	todos.insert(Todo(2, "영화 보기"));
}
```
그런데 컴파일하면 아래와 같은 오류가 발생한다.
![[Pasted image 20241005220612.png]]
왜 발생했을까? 생각을 해보자. 앞서 셋은 원소들을 저장할 때 내부적으로 정렬된 상태를 유지한다고 했다. 즉 정렬을 하기 위해서는 반드시 원소 간의 비교를 수행해야 한다. 하지만, 우리의 `Todo` 클래스에는 `operator<` 가 정의되어 있지 않다. 따라서 컴파일러는 `<` 연산자를 찾을 수 없기에 위와 같은 오류를 뿜어내는 것이다.

그렇다면 직접 `Todo` 클래스에 `operator<` 를 만들어주는 수 밖에 없다.
```cpp
#include <iostream>
#include <set>
#include <string>

template <typename T>
void print_set(std::set<T>& s) {
	// 셋의 모든 원소들을 출력하기
	for (const auto& elem : s) {
		std::cout << elem << " " << std::endl;
	}
}
class Todo {
	int priority;
	std::string job_desc;

public:
	Todo(int priority, std::string job_desc)
	  : priority(priority), job_desc(job_desc) {}
	
	bool operator<(const Todo& t) const {
		if (priority == t.priority) {
			return job_desc < t.job_desc;
		}
		return priority > t.priority;
	}
	
	friend std::ostream& operator<<(std::ostream& o, const Todo& td);
};

std::ostream& operator<<(std::ostream& o, const Todo& td) {
	o << "[ 중요도: " << td.priority << "] " << td.job_desc;
	return o;
}

int main() {
	std::set<Todo> todos;
	
	todos.insert(Todo(1, "농구 하기"));
	todos.insert(Todo(2, "수학 숙제 하기"));
	todos.insert(Todo(1, "프로그래밍 프로젝트"));
	todos.insert(Todo(3, "친구 만나기"));
	todos.insert(Todo(2, "영화 보기"));
	
	print_set(todos);
	
	std::cout << "-------------" << std::endl;
	std::cout << "숙제를 끝냈다면!" << std::endl;
	todos.erase(todos.find(Todo(2, "수학 숙제 하기")));
	print_set(todos);
}
```
성공적으로 컴파일했다면
```
[ 중요도: 3] 친구 만나기 
[ 중요도: 2] 수학 숙제 하기 
[ 중요도: 2] 영화 보기 
[ 중요도: 1] 농구 하기 
[ 중요도: 1] 프로그래밍 프로젝트 
-------------
숙제를 끝냈다면!
[ 중요도: 3] 친구 만나기 
[ 중요도: 2] 영화 보기 
[ 중요도: 1] 농구 하기 
[ 중요도: 1] 프로그래밍 프로젝트
```

먼저 `<` 연산자를 어떻게 구현했는지 살펴보자.
```cpp
bool operator<(const Todo& t) const {
	if (priority == t.priority) {
		return job_desc < t.job_desc;
	}
	return priority > t.priority;
}
```
셋이서 `<` 를 사용하기 위해서는 반드시 위와 같은 형태로 함수를 작성해야 한다. 즉 `const Todo` 를 레퍼런스로 받는 `const` 함수로 말이다.
이를 지켜야 하는 이유는 **셋 내부적으로 정렬 시에 상수 반복자를 사용하기 때문**이다. (**상수 반복자는 상수 함수만을 호출할 수 있다**)

우리의 `Todo <` 연산자는 중요도가 다르면,

## multiset
