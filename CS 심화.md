# 자료구조/알고리즘

## 1. 시간복잡도와 공간복잡도

### 핵심 개념
알고리즘의 성능을 **입력 크기 N에 대한 함수**로 표현하는 방법입니다.

|표기법|의미|용도|
|---|---|---|
|Big-O (O)|점근적 **상한**|최악의 경우 보장|
|Big-Omega (Ω)|점근적 **하한**|최선의 경우|
|Big-Theta (Θ)|점근적 **정확한 경계**|평균적 동작|
#### 왜 Big-O만 쓰는가?
실무와 면접에서 Big-O를 선호하는 이유는 **보수적 예측**이 시스템 설계에 더 안전하기 때문입니다. 서비스 SLA(Service Level Agreement)를 맞추려면 최악의 케이스를 기준으로 인프라를 설계해야 합니다. Ω는 알고리즘 이론 연구에서, Θ는 더 정밀한 분석이 필요할 때 씁니다.
#### O(1)이 O(N²)보다 무조건 빠른가?
아닙니다. Big-O는 N → ∞일 때의 **점근적 분석**입니다.

```
O(1) 실제 시간: 10,000ms (상수 계수가 엄청 큰 경우)
O(N²) 실제 시간: N=5일 때 25ms
```

N이 충분히 작으면 O(N²)이 실제로 빠를 수 있습니다. 실무에서는 **상수 계수(constant factor)** 와 **캐시 지역성(cache locality)** 도 반드시 고려해야 합니다. 예를 들어 HashMap의 조회는 O(1)이지만 해시 계산 비용과 캐시 미스가 발생하면, 정렬된 소규모 배열을 선형 탐색하는 것보다 느릴 수 있습니다.

#### 공간복잡도 실무 트레이드오프
시간-공간 트레이드오프(Time-Space Tradeoff)는 실무에서 매우 중요합니다. 예를 들어 캐싱은 공간을 희생해 시간을 얻는 전략입니다. AWS Lambda 같은 환경에서는 메모리 비용이 직접 과금되므로, 공간복잡도 최적화가 비용 절감과 직결됩니다.

#### 대안
알고리즘 분석 시 Big-O 외에 **Amortized Analysis(분할 상환 분석)** 도 중요합니다. ArrayList의 동적 확장은 단일 연산은 O(N)이지만, 분할 상환 시 O(1)로 분석합니다.

## 2. 링크드 리스트

### 배열 vs 링크드 리스트 비교

| 연산         | 배열             | 링크드 리스트           |
| ---------- | -------------- | ----------------- |
| 인덱스 접근     | O(1)           | O(N)              |
| 삽입/삭제 (중간) | O(N)           | O(1) (노드 참조 시)    |
| 삽입/삭제 (끝)  | O(1) amortized | O(1) (tail 포인터 시) |
| 메모리        | 연속 할당, 캐시 친화적  | 비연속, 포인터 오버헤드     |

#### 캐시 지역성(Cache Locality) 
관점이 핵심입니다. 배열은 메모리에 연속으로 저장되어 CPU 캐시 라인을 효율적으로 활용합니다. 링크드 리스트는 노드가 힙 여기저기 흩어져 캐시 미스가 빈번하게 발생합니다. 실제로 소규모 데이터에서는 O(N)인 배열 탐색이 O(1)인 HashMap보다 빠른 경우도 있습니다.

#### 링크드 리스트로 구현 가능한 자료구조
스택(head에서 push/pop), 큐(tail enqueue, head dequeue), 데크(Doubly Linked List), 해시 테이블의 체이닝 충돌 처리, LRU 캐시(HashMap + Doubly Linked List)

#### 대안
Java에서는 `LinkedList<E>` 대신 **ArrayDeque**를 권장합니다. 캐시 효율성이 높고 포인터 오버헤드가 없으며, 대부분의 연산에서 실제 성능이 더 우수합니다.

## 3. 스택과 큐

### 스택 2개로 큐 구현
#### java
```java
public class QueueUsingTwoStacks<T> {
    private final Deque<T> inStack = new ArrayDeque<>();
    private final Deque<T> outStack = new ArrayDeque<>();

    public void enqueue(T item) {
        inStack.push(item);  // O(1)
    }

    public T dequeue() {
        if (outStack.isEmpty()) {
            // inStack의 모든 요소를 outStack으로 이동
            while (!inStack.isEmpty()) {
                outStack.push(inStack.pop());
            }
        }
        if (outStack.isEmpty()) throw new NoSuchElementException();
        return outStack.pop();  // Amortized O(1)
    }
}
```
#### 시간복잡도
enqueue O(1), dequeue **amortized O(1)** (각 요소는 최대 2번 이동)

### 큐 2개로 스택 구현
**방법 A - push 비용:** push 시 q2에 넣고, q1 전체를 q2로 이동 후 swap → push O(N), pop O(1) **방법 B - pop 비용:** pop 시 q1에서 N-1개를 q2로 이동, 마지막 요소 반환 → push O(1), pop O(N)

### 배열로 스택과 큐 구현
스택은 배열 + top 인덱스로 O(1) 완벽하게 가능합니다.

큐는 **Circular Buffer(환형 버퍼)** 로 구현해야 O(1)을 유지합니다.

#### java
```java
public class CircularQueue<T> {
    private final Object[] data;
    private int head = 0, tail = 0, size = 0;
    private final int capacity;

    public CircularQueue(int capacity) {
        this.capacity = capacity;
        this.data = new Object[capacity];
    }

    public void enqueue(T item) {
        if (size == capacity) throw new IllegalStateException("Queue is full");
        data[tail] = item;
        tail = (tail + 1) % capacity;  // 핵심: 모듈러 연산
        size++;
    }

    @SuppressWarnings("unchecked")
    public T dequeue() {
        if (size == 0) throw new NoSuchElementException();
        T item = (T) data[head];
        head = (head + 1) % capacity;
        size--;
        return item;
    }
}
```

### Prefix / Infix / Postfix

| 표기법 | 예시 | 특징 |
|---|---|---|
| Infix | `3 + 4 * 2` | 사람이 읽기 쉬움, 괄호/우선순위 필요 |
| Prefix | `+ 3 * 4 2` | 괄호 불필요, 재귀적 계산 용이 |
| Postfix | `3 4 2 * +` | 스택으로 O(N) 계산 가능 |

#### Postfix 스택 계산
```
토큰을 순서대로 읽으며:
- 숫자 → 스택 push
- 연산자 → 스택에서 2개 pop, 계산 후 push
```
**Infix → Postfix 변환 (Shunting-yard 알고리즘):** 연산자 스택 활용, 우선순위 비교

### Deque 구현 및 C++ Deque의 O(1) Random Access
Deque는 **Doubly Linked List** 또는 **Circular Buffer**로 구현합니다.

C++ `std::deque`는 **고정 크기 청크(chunk) 배열들의 배열**로 구현됩니다. 중앙 인덱스 테이블을 가리키는 포인터 배열이 있고, 각 청크는 고정 크기 배열입니다.
인덱스 i에 접근 시: `chunk_index = i / chunk_size`, `offset = i % chunk_size` 로 두 번의 포인터 역참조만에 O(1) 달성합니다.

#### 대안
Java에서는 `ArrayDeque`가 단일 circular array로 deque를 구현합니다.

## 4. 해시 자료구조

### 좋은 해시 함수 설계
좋은 해시 함수의 조건은 **균일 분포(Uniform Distribution)**, **계산 속도**, **눈사태 효과(Avalanche Effect: 입력 1비트 변화 → 출력 50% 비트 변화)** 입니다.
#### 실무에서 사용되는 기법
- **소수(Prime Number) 활용:** 모듈러 연산 시 소수를 사용하면 분포가 균일해짐
- **Polynomial Hashing:** `hash = s[0]*p^n + s[1]*p^(n-1) + ... ` (Java String.hashCode 방식)
- **MurmurHash, FNV, xxHash:** 빠른 속도와 좋은 분산을 위한 실무용 해시

### 해시 충돌 처리 방법
#### Chaining(체이닝)
같은 버킷에 LinkedList/Tree로 연결
#### Open Addressing(개방 주소법)
- Linear Probing: 다음 빈 슬롯 탐색 → 클러스터링 문제
- Quadratic Probing: 제곱수로 탐색 간격 증가
- Double Hashing: 두 번째 해시 함수로 간격 결정
#### Java HashMap의 충돌 처리
Java 8부터 버킷 내 요소가 **8개 이상**이면 LinkedList → **Red-Black Tree**로 전환합니다. 다시 6개 이하로 줄면 LinkedList로 복구합니다. 이를 **Treeification**이라고 합니다.
```
Java HashMap 동작:
- 초기: 버킷별 LinkedList (O(N) 최악)
- 8개 초과 시: Red-Black Tree 전환 (O(log N) 보장)
- Load Factor 0.75 초과 시: Rehashing (버킷 2배 확장)
```

### Double Hashing 장단점
#### 장점
- 클러스터링 최소화, 분산 우수
#### 단점
- 두 번째 해시 함수가 0을 반환하면 무한루프, 계산 비용 증가, 캐시 지역성 저하

**해결책:** 두 번째 해시 함수를 `h2(k) = prime - (k % prime)` 형태로 설계해 0 방지

### Load Factor
Java HashMap의 기본 Load Factor는 **0.75**입니다. Load Factor = 요소 수 / 버킷 수. 0.75는 메모리 효율과 성능의 균형점으로 실험적으로 결정된 값입니다.
- **Load Factor 높으면**: 충돌 증가, 메모리 절약
- **Load Factor 낮으면**: 충돌 감소, 메모리 낭비

### 멀티스레드 환경의 Race Condition 해결
#### 문제
Java 8 이하 HashMap은 Rehashing 중 Race Condition으로 **무한루프(순환 참조)** 가 발생할 수 있습니다.

#### 해결 방법 비교
- java
```java
// 방법 1: Collections.synchronizedMap - 모든 연산에 synchronized → 성능 최악
Map<K,V> map = Collections.synchronizedMap(new HashMap<>());

// 방법 2: ConcurrentHashMap - 권장
// Java 8+: 버킷 단위 synchronized + CAS(Compare-And-Swap) 연산 활용
// 읽기는 완전히 non-blocking, 쓰기는 버킷 레벨 lock
ConcurrentHashMap<K,V> map = new ConcurrentHashMap<>();
```

#### ConcurrentHashMap 내부 구조
```
ConcurrentHashMap (Java 8+)
├── 읽기: volatile 읽기로 lock-free
├── 쓰기: CAS로 첫 노드 삽입 시도
│   └── 실패 시: synchronized(버킷 head) - 버킷 단위 fine-grained lock
└── Rehashing: 점진적 transfer (ForwardingNode로 다른 스레드 안내)
````

#### 더 높은 성능이 필요하다면
**Striped Lock** 패턴이나 **ReadWriteLock** 도 고려할 수 있습니다.

#### 대안
읽기 극도로 많은 환경에서는 **CopyOnWriteArrayList** 패턴처럼, 쓰기 시 전체 복사하는 방식도 있습니다(읽기 O(1), 쓰기 O(N)이므로 쓰기 빈도가 낮을 때만 유효).

## 5. 트리와 이진탐색트리

### 그래프 vs 트리
트리는 그래프의 특수한 형태입니다: **N개 노드, N-1개 간선, 사이클 없는 연결 그래프**

| 특성    | 그래프     | 트리             |
| ----- | ------- | -------------- |
| 사이클   | 가능      | 불가             |
| 루트    | 없을 수 있음 | 있음             |
| 부모 노드 | 여러 개 가능 | 정확히 1개 (루트 제외) |

### BST 중위 탐색(In-order Traversal)
BST에서 중위 탐색 결과는 **오름차순 정렬된 수열**입니다. 이를 활용해 BST가 올바른지 검증하거나, 정렬된 배열로 변환할 수 있습니다.

### BST 주요 연산 시간복잡도

| 연산  | 평균       | 최악 (편향 시) |
| --- | -------- | --------- |
| 탐색  | O(log N) | O(N)      |
| 삽입  | O(log N) | O(N)      |
| 삭제  | O(log N) | O(N)      |

#### 최악이 O(N)인 이유
**편향 트리(Skewed Tree)** 발생 시 사실상 LinkedList가 됩니다.
- **편향 발생 케이스**: 이미 정렬된 데이터를 순서대로 삽입 시 (1, 2, 3, 4, 5 → 우측으로만 편향)

### BST 삽입/삭제
#### 삭제 케이스
- 리프 노드: 그냥 제거
- 자식 1개: 자식으로 대체
- 자식 2개: **중위 후계자(In-order Successor)** 또는 **중위 전임자(In-order Predecessor)** 로 대체

### 삼진탐색트리(Ternary Search Tree)
BST와 동일한 로직으로 삼진탐색트리를 만들 수 있습니다. 각 노드가 두 개의 분기값을 가져 `< left`, `middle`, `> right` 세 자식을 갖습니다. 단, 이는 이미 **B-Tree의 특수 케이스**에 해당합니다. 각 노드에 값 2개, 자식 3개를 가진 **2-3 Tree**가 이에 해당합니다.

순수한 삼진탐색트리도 구현 가능하지만, 실용적 이점이 없어 잘 사용하지 않습니다. 높이는 log₃N으로 낮아지지만, 각 노드에서 비교 횟수가 2배가 되어 전체 복잡도는 동일합니다.

**대안:** BST 대신 **Skip List**를 사용하면 확률적으로 O(log N)을 보장하면서 구현이 더 단순합니다. Redis의 Sorted Set이 Skip List로 구현되어 있습니다.

## 6. 힙(Heap)

### 배열로 구현
1-indexed 배열에서:
- 노드 i의 왼쪽 자식: `2*i`
- 노드 i의 오른쪽 자식: `2*i + 1`
- 노드 i의 부모: `i/2`

### 삽입/삭제 (Heapify)
#### 삽입
배열 끝에 추가 → **Bubble Up** (부모와 비교하며 위로 이동) → O(log N)
#### 삭제(최댓값/최솟값 추출)
루트 제거 → 마지막 노드를 루트로 이동 → **Bubble Down (Heapify Down)** → O(log N)
#### 편향이 발생하지 않는 이유
힙은 **완전 이진트리(Complete Binary Tree)** 를 항상 유지합니다. 삽입은 항상 가장 마지막 위치에, 삭제는 루트와 마지막 노드를 교환 후 Heapify하므로 구조적으로 편향이 불가능합니다.

### 힙 정렬
#### 시간복잡도
O(N log N) (최선/평균/최악 동일) 공간복잡도: O(1) (in-place) **Stable 하지 않습니다.** 동일한 값의 상대적 순서가 보장되지 않습니다.
#### 대안
실무에서는 `PriorityQueue`를 사용합니다. 하지만 **K번째 최솟값**, **중앙값 스트리밍** 같은 문제에서는 Min-Heap + Max-Heap 조합이 유용합니다.

## 7. BBST (Balanced Binary Search Tree)

### 종류 비교

|BBST|균형 조건|회전|특징|
|---|---|---|---|
|AVL Tree|좌우 높이 차 ≤ 1|많음|엄격한 균형, 탐색 빠름|
|Red-Black Tree|느슨한 균형|적음|삽입/삭제 빠름|
|2-3-4 Tree|모든 리프 같은 깊이|-|RBT와 동형|
|B-Tree|노드당 다수 키|-|디스크 I/O 최적화|

### Red-Black Tree 4가지 성질
1. 모든 노드는 **Red** 또는 **Black**
2. 루트는 **Black**
3. 모든 리프(NIL 노드)는 **Black**
4. **Red 노드의 자식은 반드시 Black** (Red가 연속으로 나올 수 없음)
5. (핵심) 루트에서 임의의 리프까지 경로상 **Black 노드 수가 동일** (Black-height 동일)

이 성질들로 인해 트리 높이가 **$2 \log (N+1)$** 이하로 보장됩니다.

#### 균형 유지 방법
삽입/삭제 시 **색상 변경(Recoloring)** 과 **회전(Rotation)** 을 통해 성질 복구

### Red-Black Tree가 AVL보다 많이 쓰이는 이유
AVL은 더 엄격한 균형을 유지하므로 탐색은 약간 빠르지만, **삽입/삭제 시 더 많은 회전 연산**이 필요합니다.
실무에서는 탐색뿐만 아니라 삽입/삭제도 빈번하므로 Red-Black Tree가 **전반적으로 더 균형 잡힌 성능**을 보입니다.

Java의 `TreeMap`, `TreeSet`, C++ `std::map`이 Red-Black Tree로 구현되어 있습니다.

#### 대안
데이터베이스 인덱스에는 **B+ Tree**가 사용됩니다. 디스크 페이지 단위로 데이터를 저장하고, 리프 노드를 연결해 범위 쿼리에 최적화되어 있습니다.

## 8. 정렬 알고리즘

### Quick Sort vs Merge Sort

|항목|Quick Sort|Merge Sort|
|---|---|---|
|시간복잡도 (평균)|O(N log N)|O(N log N)|
|시간복잡도 (최악)|O(N²)|O(N log N)|
|공간복잡도|O(log N) stack|O(N)|
|Stable|❌|✅|
|캐시 효율|우수 (in-place)|보통|

### Quick Sort O(N²) 케이스와 개선
**O(N²) 발생:** 이미 정렬된 배열에서 첫 번째/마지막 요소를 피벗으로 선택 시

#### 개선 방법
- **Median-of-Three:** 첫 번째, 중간, 마지막 요소 중 중앙값을 피벗으로 선택
- **Random Pivot:** 무작위 피벗 선택
- **Introsort:** Quick Sort + Heap Sort + Insertion Sort 혼합 (C++ std::sort, Java Arrays.sort primitive 배열)

### Stable Sort
동일한 키를 가진 요소들의 **상대적 순서가 유지**되는 정렬
#### Stable
- Merge Sort
- Insertion Sort
- Bubble Sort
- Tim Sort
#### Unstable
- Quick Sort
- Heap Sort
- Selection Sort

### Merge Sort 비재귀 구현 (Bottom-Up)
#### java
```java
public void mergeSortIterative(int[] arr) {
    int n = arr.length;
    // size: 각 서브배열의 크기 (1, 2, 4, 8, ...)
    for (int size = 1; size < n; size *= 2) {
        for (int left = 0; left < n - size; left += 2 * size) {
            int mid = left + size - 1;
            int right = Math.min(left + 2 * size - 1, n - 1);
            merge(arr, left, mid, right);
        }
    }
}
```

### Radix Sort
$O(d * (N + K))$ — $d$: 자릿수, $K$: 기수(radix)

비교 기반이 아니므로 **비교 기반 정렬의 O(N log N) 하한을 깨는 것처럼 보이지만**, 실제로는 정수/문자열처럼 키가 유한한 경우에만 적용 가능합니다.

### Bubble / Selection / Insertion 비교

| 알고리즘      | 평균    | 최선    | 최악    | Stable | 특징           |
| --------- | ----- | ----- | ----- | ------ | ------------ |
| Bubble    | O(N²) | O(N)  | O(N²) | ✅      | 거의 정렬됐을 때 유리 |
| Selection | O(N²) | O(N²) | O(N²) | ❌      | 항상 N²번 비교    |
| Insertion | O(N²) | O(N)  | O(N²) | ✅      | 거의 정렬됐을 때 최강 |

**거의 정렬된 배열:** Insertion Sort가 압도적으로 유리합니다. 이미 정렬된 배열에서 O(N)입니다. 이것이 **Tim Sort**가 소규모 구간에 Insertion Sort를 사용하는 이유입니다.

### Java의 정렬 알고리즘
- **primitive 배열** (`int[]`, `double[]`): **Dual-Pivot Quick Sort** (Java 7+)
- **객체 배열** (`Integer[]`, `Object[]`): **Tim Sort** (Stable 보장 필요)

Tim Sort = Merge Sort + Insertion Sort 하이브리드. 자연적으로 정렬된 런(run)을 찾아 병합합니다.

### 50GB 데이터, 4GB 메모리 정렬: 외부 정렬(External Sort)
```
Phase 1 - Run Generation:
  4GB씩 메모리에 로드 → 내부 정렬 → 디스크에 정렬된 청크 저장
  총 50/4 = ~13개 청크 생성

Phase 2 - K-way Merge:
  13개 청크를 동시에 읽으며 MinHeap으로 병합
  각 청크에서 버퍼 유지, MinHeap에서 최솟값 추출 → 출력 버퍼에 쓰기
```
```
[청크1] [청크2] ... [청크13]
   ↓       ↓           ↓
  [     Min-Heap (13개 요소)    ]
              ↓
         [출력 파일]
```
실무에서는 **Apache Spark**의 External Sort, **PostgreSQL**의 External Merge Sort가 이 방식을 사용합니다.

#### 대안
Hadoop MapReduce는 Map 단계에서 로컬 정렬 후 Shuffle-Sort-Merge 과정을 거칩니다.

## 9. 그래프 자료구조

### 인접 행렬 vs 인접 리스트

| | 인접 행렬 | 인접 리스트 |
|---|---|---|
| 두 정점 연결 확인 | O(1) | O(degree) |
| 한 정점의 모든 인접 정점 | O(V) | O(degree) |
| 공간복잡도 | O(V²) | O(V + E) |
| 적합한 경우 | 밀집 그래프 | 희소 그래프 |

**N개 정점, N³개 간선:** 최대 간선 수 V² = N²이므로 N³개 간선은 이론적으로 불가능합니다 (단순 그래프 기준). 현실적으로 밀집 그래프이므로 인접 행렬이 적합합니다.

### 사이클 없는 그래프 = 트리?
아닙니다. **Forest(포레스트)** 가 반례입니다. 여러 개의 독립된 트리들의 집합으로, 사이클이 없지만 연결되어 있지 않습니다.

조건: 사이클 없음 + **연결 그래프**여야 트리입니다.

#### 대안
그래프 표현으로 **CSR(Compressed Sparse Row)** 형식도 있습니다. 희소 행렬 표현에 효율적이며, 그래프 처리 라이브러리(NetworkX, GraphX)에서 사용합니다.

## 10. 최단거리 알고리즘

### 알고리즘 비교

| 알고리즘 | 시간복잡도 | 음수 간선 | 음수 사이클 | 비고 |
|---|---|---|---|---|
| BFS | O(V+E) | ❌ | ❌ | 가중치 없는 그래프 |
| Dijkstra (힙) | O((V+E) log V) | ❌ | ❌ | 양수 가중치 |
| Bellman-Ford | O(VE) | ✅ | 감지 가능 | 음수 허용 |
| Floyd-Warshall | O(V³) | ✅ | 감지 가능 | 모든 쌍 최단거리 |
| A* | O(E log V) (휴리스틱 의존) | ❌ | ❌ | 단일 목적지 탐색 |
### 트리에서 최단거리
트리는 두 노드 사이에 **유일한 경로**가 존재합니다. 단순 BFS/DFS로 O(N)에 구할 수 있습니다. 더 나아가 **LCA(Lowest Common Ancestor)** 와 **Sparse Table**을 활용하면 전처리 O(N log N) 후 쿼리당 O(log N)에 구할 수 있습니다.

### Dijkstra - 힙 없이 구현 시
배열로 구현 시 매 단계에서 최솟값 탐색에 O(V) → 전체 O(V²). 힙 사용 시 O((V+E) log V). **정점 수가 많고 간선이 적은 희소 그래프**에서 힙이 훨씬 유리합니다.

### N개 정점, N³개 간선
앞서 언급했듯 단순 그래프에서 N³개 간선은 불가능하지만, 매우 밀집된 경우라면 **Floyd-Warshall O(V³)** 이 Dijkstra를 V번 반복하는 것과 동일한 복잡도를 가지며 구현이 더 단순합니다.

### A* 알고리즘
Dijkstra + 휴리스틱 함수 h(n): 목적지까지의 예상 비용

`f(n) = g(n) + h(n)` (g: 출발지부터 현재까지 실제 비용, h: 예상 비용)

h(n)이 실제 비용을 절대 과대추정하지 않으면(Admissible) 최적해를 보장합니다. 게임 AI 경로탐색, 내비게이션에 사용됩니다. 올바른 휴리스틱이 있을 때 Dijkstra보다 훨씬 빠르지만, 나쁜 휴리스틱이면 오히려 느릴 수 있습니다.

**대안:** 최단거리 대신 **근사 최단거리**가 필요하면 Greedy Best-First Search가 더 빠릅니다.

## 11. 재귀함수

### Call Stack을 활용한 동작 과정
```
factorial(3) 호출
├── [Stack Frame 3] n=3, 대기 중
│   └── factorial(2) 호출
│       ├── [Stack Frame 2] n=2, 대기 중
│       │   └── factorial(1) 호출
│       │       └── [Stack Frame 1] return 1
│       └── return 2 * 1 = 2
└── return 3 * 2 = 6
````
각 Stack Frame에는 **지역 변수, 매개변수, 반환 주소**가 저장됩니다. JVM의 기본 스택 크기는 약 512KB~1MB로, 깊은 재귀는 **StackOverflowError** 를 유발합니다.

### 꼬리 재귀 최적화 (Tail Call Optimization, TCO)
**꼬리 재귀:** 재귀 호출이 함수의 **마지막 연산**인 경우
#### kotlin
```kotlin
// 일반 재귀 (TCO 불가)
fun factorial(n: Int): Long = if (n <= 1) 1 else n * factorial(n - 1)

// 꼬리 재귀 (TCO 가능)
tailrec fun factorial(n: Int, acc: Long = 1): Long =
    if (n <= 1) acc else factorial(n - 1, n * acc)
```

Kotlin은 `tailrec` 키워드로 컴파일 시 반복문으로 변환합니다. **Java/JVM은 TCO를 지원하지 않습니다** (JVM 스펙 이슈). Scala는 `@tailrec` 어노테이션을 제공합니다.

**TCO 원리:** 마지막 호출이므로 현재 Stack Frame을 재사용 → O(N) 스택이 O(1)로 감소

**대안:** 재귀 대신 **명시적 스택(Explicit Stack)** 을 사용해 반복문으로 변환하면 JVM에서도 스택 오버플로우 없이 깊은 재귀를 처리할 수 있습니다.

## 12. MST (Minimum Spanning Tree)

### MST 정의
V개 정점, 사이클 없이 모든 정점을 연결하는 V-1개 간선의 부분 그래프 중 **가중치 합이 최소인 것**

### Union-Find 자료구조
#### java

```java
public class UnionFind {
    private final int[] parent, rank;

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }

    // Path Compression: O(α(N)) ≈ O(1)
    public int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    // Union by Rank
    public void union(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return;
        if (rank[px] < rank[py]) { int tmp = px; px = py; py = tmp; }
        parent[py] = px;
        if (rank[px] == rank[py]) rank[px]++;
    }
}
```

**Path Compression + Union by Rank** 조합 시 연산당 **O(α(N))** — α는 역 아커만 함수로 실질적으로 O(1)

### Kruskal vs Prim

||Kruskal|Prim|
|---|---|---|
|방식|간선 중심 (Edge-based)|정점 중심 (Vertex-based)|
|시간복잡도|O(E log E)|O((V+E) log V)|
|적합한 경우|희소 그래프|밀집 그래프|

**언제 무엇이 빠른가:** E ≈ V² (밀집)이면 Prim, E ≈ V (희소)이면 Kruskal

### MST 결과는 항상 트리인가?

**증명:**

1. MST는 V개 정점을 연결하므로 V-1개 간선 (MST 정의)
2. V개 정점, V-1개 간선 + 사이클 없음 = 트리의 정의
3. MST 구성 과정에서 사이클이 생기는 간선을 추가하지 않으므로 사이클 없음
4. 따라서 MST는 항상 트리 ✅

단, **간선 가중치가 동일한 경우 MST가 유일하지 않을 수 있습니다.** 여러 MST가 존재할 수 있지만 모두 합이 동일합니다.

**대안:** MST 대신 **최대 신장 트리(Maximum Spanning Tree)** 가 필요한 경우도 있습니다 (예: 네트워크 최대 대역폭 경로). 가중치를 음수로 바꿔 MST를 구하면 됩니다.

---

## 13. Thread Safe 자료구조

### Java의 Thread Safe 컬렉션

|컬렉션|Thread Safe 여부|
|---|---|
|ArrayList, HashMap, LinkedList|❌|
|Vector, Hashtable|✅ (but synchronized → 성능 나쁨)|
|ConcurrentHashMap|✅ (버킷 레벨 lock)|
|CopyOnWriteArrayList|✅ (쓰기 시 복사, 읽기 lock-free)|
|ConcurrentLinkedQueue|✅ (CAS 기반 lock-free)|
|BlockingQueue (ArrayBlockingQueue 등)|✅ (생산자-소비자 패턴)|

### 배열 길이를 알 때 더 빠른 Thread Safe 연산

배열을 고정 크기 세그먼트로 분할해 **세그먼트별 Lock** 사용:

java

````java
// Striped Lock 패턴
public class StripedArray<T> {
    private final Object[] data;
    private final ReentrantLock[] locks;
    private final int stripes;

    public StripedArray(int size, int stripes) {
        this.data = new Object[size];
        this.stripes = stripes;
        this.locks = new ReentrantLock[stripes];
        Arrays.fill(locks, new ReentrantLock());
    }

    public void set(int index, T value) {
        int stripe = index % stripes;
        locks[stripe].lock();
        try { data[index] = value; }
        finally { locks[stripe].unlock(); }
    }
}
```

또한 **AtomicIntegerArray**, **AtomicLongArray** 를 사용하면 CAS 기반으로 lock 없이 원자적 연산이 가능합니다.

**대안:** **Disruptor 패턴 (LMAX Disruptor)** — Ring Buffer + CAS로 초고성능 Thread Safe 큐를 구현합니다. 금융 시스템에서 초당 수백만 건 처리에 사용됩니다.

---

## 14. 문자열 자료구조 및 알고리즘

### Trie (트라이)
```
삽입된 단어: ["apple", "app", "apt", "bat"]

       root
      /    \
     a      b
     |      |
     p      a
    / \     |
   p   t    t
   |   |
   l   (end)
   |
   e
   |
  (end)
````

삽입/탐색: O(M) — M은 문자열 길이 공간: O(총 문자 수)

**실무 활용:** 자동완성, 사전, IP 라우팅 테이블

### KMP (Knuth-Morris-Pratt)

패턴 전처리로 **Failure Function(부분 일치 테이블)** 생성, 불일치 시 패턴을 최대한 이동

- 전처리: O(M)
- 탐색: O(N)
- 전체: **O(N + M)** — 단순 탐색 O(NM) 대비 획기적

### Rabin-Karp

**Rolling Hash**를 이용한 패턴 매칭. 해시 값을 O(1)에 갱신하며 탐색

- 평균: O(N + M)
- 최악: O(NM) (해시 충돌 많을 경우)

**다중 패턴 탐색에 유리** (여러 패턴의 해시를 HashSet에 저장하면 O(N)에 다중 탐색)

**대안:** **Aho-Corasick 알고리즘** — Trie + KMP 아이디어를 결합해 O(N + M + 매칭 수)에 다중 패턴 탐색. 검열 시스템, 안티바이러스에 사용됩니다.

---

## 15. 이진탐색

### 시간복잡도 증명

매 단계에서 탐색 범위가 절반으로 줄어듦: N → N/2 → N/4 → ... → 1

단계 수 k에서 N/2^k = 1 → k = **log₂N**

**O(log N)**

### Lower Bound / Upper Bound

java

```java
// Lower Bound: target 이상인 첫 번째 인덱스
public int lowerBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;  // 오버플로우 방지
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

// Upper Bound: target 초과인 첫 번째 인덱스
public int upperBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```

### 삼진탐색의 시간복잡도

범위를 1/3로 줄임: N → N/3 → ... → 1

단계 수 k: N/3^k = 1 → k = log₃N = **log₂N / log₂3 ≈ 0.63 * log₂N**

이진탐색보다 단계 수는 적지만, 매 단계 비교 횟수가 2배이므로 실제 성능은 동일하거나 약간 나쁩니다. 점근적으로 **O(log N)**으로 동일합니다.

### 부등호 범위 변경 시 결과

`<=`를 `<`로 바꾸면 범위 처리 방식이 바뀌어 결과가 달라질 수 있습니다. 특히 **경계 조건(Boundary Condition)** 이 변경되어 off-by-one 오류가 발생합니다. 이는 실무에서 가장 자주 실수하는 부분으로, 항상 **불변식(Loop Invariant)** 을 명확히 정의하고 구현해야 합니다.

**대안:** 이진탐색의 단조성 조건을 파라메트릭 서치(Parametric Search)로 확장하면 "최솟값을 최대화" 류 최적화 문제를 이진탐색으로 풀 수 있습니다.

---

## 16. 그리디 알고리즘 vs 동적 계획법

### 비교

||그리디|동적 계획법|
|---|---|---|
|방식|현재 최선 선택|모든 하위 문제 해결|
|최적 부분 구조|필요|필요|
|탐욕적 선택 속성|필요 (핵심 차이)|불필요|
|시간복잡도|대체로 빠름|대체로 느림|
|구현|단순|복잡|

**탐욕적 선택 속성(Greedy Choice Property):** 현재 단계의 최선 선택이 전체 최적해에 포함됨이 보장되는 성질. **증명이 매우 중요합니다.**

### 언제 무엇을 사용하는가

**그리디 적용 조건 (둘 다 만족해야 함):**

- 최적 부분 구조
- 탐욕적 선택 속성

예: 다익스트라, 크루스칼, 활동 선택 문제, 허프만 코딩

**DP 적용 조건:**

- 최적 부분 구조
- 중복 부분 문제 (Overlapping Subproblems)

예: 배낭 문제, LCS, 편집 거리, 행렬 체인 곱셈

### DP 문제 = 재귀로 변환 가능?

**가능합니다.** DP의 두 가지 구현 방식:

- **Top-Down (Memoization):** 재귀 + 캐시. 필요한 하위 문제만 계산
- **Bottom-Up (Tabulation):** 반복문. 모든 하위 문제를 순서대로 계산

모든 DP 문제는 재귀(Top-Down)로 변환 가능하며, 반대도 마찬가지입니다. 단, JVM에서 재귀 깊이 제한 때문에 Bottom-Up이 실용적으로 더 안전합니다.

**대안:** 일부 DP는 **공간 최적화**가 가능합니다. 예를 들어 1D DP 배열에서 이전 행만 필요한 경우 2D → 1D로 줄일 수 있습니다 (배낭 문제 O(N*W) → O(W)).

---

## 압박 면접 질문

자, 이제 배운 내용을 바탕으로 답해봐요. 실제 면접처럼 생각해야 합니다.

**Q1.** Java의 `HashMap`은 멀티스레드 환경에서 Java 7까지 무한루프가 발생할 수 있었습니다. 이 현상이 **왜** 발생하는지 내부 동작 수준에서 설명하고, Java 8에서 이를 어떻게 개선했는지 설명해보세요. 그리고 Java 8 이후에도 `ConcurrentHashMap`을 써야 하는 이유가 있나요?

**Q2.** Quick Sort의 평균 시간복잡도가 O(N log N)임을 **직관적으로** 설명해보세요. 단순히 "분할 정복이라서"가 아니라, 피벗 선택과 파티셔닝 과정이 어떻게 O(N log N)으로 이어지는지 설명해야 합니다. 그리고 실제 Java의 `Arrays.sort()`가 primitive와 Object 배열에서 다른 알고리즘을 쓰는 이유는 무엇인가요?

**Q3.** 대용량 로그 파일(100GB)에서 가장 많이 등장하는 URL Top 10을 찾아야 합니다. 메모리는 4GB만 사용 가능하고, 처리 시간도 최소화해야 합니다. 이 문제를 어떻게 설계하시겠습니까? 사용할 자료구조와 알고리즘, 그리고 분산 처리 확장 방안까지 포함해서 설명해보세요.

**Q4.** Red-Black Tree가 DB 인덱스에 사용되지 않고 B+ Tree가 사용되는 이유를 **디스크 I/O와 캐시 구조** 관점에서 설명해보세요. 그리고 만약 모든 데이터가 메모리에 올라간다면 어떤 자료구조가 더 적합할까요?

**Q5.** 당신이 설계하는 서비스에서 사용자 세션 저장소로 `ConcurrentHashMap`을 사용하고 있습니다. 서버가 여러 대로 확장되는 시점에 어떤 문제가 발생하고, 어떻게 해결할 수 있나요? 해결책의 트레이드오프도 설명해주세요.