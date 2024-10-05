
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

만일 벡터였다면 원소가 존재하는지 아닌지 확인하기 위해 벡터의 처음부터 끝 까지 하나씩 비교해가면서 찾았어야 했을 것이다. 만일 원소가 없었더라면 벡터에 있는 모든 원소를 확인하였을 것이다 (즉 벡터에서 [find](https://modoocode.com/261) 는 $O(N)$이라 볼 수 있다).

하지만 셋의 경우 놀랍게도 $O(logN)$ 으로 원소가 존재하는지 확인할 수 있습니다. 이것이 가능한 이유는 셋 내부적으로 원소들이 정렬된 상태를 유지하기 때문에 비교적 빠르게 원소의 존재 여무를 확인할 수 있습니다.

따서 20 을 찾았을 때 `Yes` 가 나오고 셋에 없는 원소인 25 를 찾는다면 `No` 가 출력됩니다.

셋이 이러한 방식으로 작업을 수행할 수 있는 이유는 바로 내부적으로 트리 구조로 구성되어 있기 때문입니다.