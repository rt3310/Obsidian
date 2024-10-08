- 시간 제한: 1 초

정점 N개의 트리에서 두 사람이 게임을 진행하려 한다.
각 정점은 1번부터 N번 까지 번호가 매겨져 있고 루트노드는 1번 노드이다.
게임은 서로 턴을 번갈아 가며 진행되고 트리 위에 놓을 수 있는 말과 함께 진행된다.
두 사람의 점수는 모두 0점으로 시작한다.

각 턴마다 두 사람은 다음과 같은 작업을 반복한다.
- 현재 말이 놓여 있는 정점의 번호만큼 자신의 점수에 더한다.
- 현재 말이 놓여 있는 정점의 자식 정점이 없다면 그대로 게임을 종료한다. <br> 자식 정점이 존재한다면 자식 정점 중 원하는 자식 정점으로 말을 옮긴다.

게임이 종료되었을 때 선공의 점수가 후공의 점수보다 높거나 같다면 선공이 승리하고 아니라면 후공이 승리한다.
두 사람이 최적으로 플레이할 때, 처음 말이 놓여져 있는 정점의 번호에 따라 선공이 이기는지 후공이 이기는지 구해보자.

### 입력
- 첫째 줄에 정점의 수 N이 주어진다.
$1≤N≤100000$
- 둘째 줄부터 N−1개의 줄에 간선을 나타내는 정수 u,v가 주어진다.
$1≤u,v≤N$
- 이는 u번 정점과 v번 정점을 잇는 간선이 존재한다는 뜻이다.
### 출력
- N개의 줄에 걸쳐 정답을 출력한다.
- i번째 줄에는 말의 시작위치가 i번 정점일 때의 결과를 출력한다.
- 선공이 이긴다면 1을 후공이 이긴다면 0을 출력한다.

---
### 예시 1
입력
```
5
1 3
2 1
3 4
5 1
```
출력
```
1  
1  
0  
1  
1
```

### 예시 2
입력
```
6  
1 3  
1 2  
3 5  
3 6  
2 4
```

```
1  
0  
0  
1  
1  
1
```

## 중점

자신의 이익을 최대화하면서 동시에 상대방의 이익을 최소화하는 것이 최적의 전략이다.
때문에 현재 노드 점수, 자식 노드들 중 최소 점수를 다 고려함으로써 현재 얻을 수 있는 이익과 미래에 상대방에게 줄 수 있는 이익을 동시에 고려한다.
이를 위해, (상대의 최적 점수) - (나의 최적 점수)가 최소가 되는 노드를 선택함으로써, 상대방에게 넘겨주는 점수와 그 후 상대방이 얻을 수 있는 최대 점수의 차이를 최소화한다.

```cpp
#include <iostream>
#include <vector>
using namespace std;

int n;
vector<int> nodes[100001];
int parent[100001];
int maxScores[100001];

pair<int, int> initScores(int cur, int depth) {
    if (nodes[cur].empty()) {
        maxScores[cur] = cur;
        if (depth % 2 == 1) {
            return {cur, 0};
        }
        return {0, cur};
    }

    int mn = 100001;
    int st = 0;
    int nd = 0;
    int node = 0;
    pair<int, int> result;
    for (int next : nodes[cur]) {
        result = initScores(next, depth + 1);
        if (depth % 2 == 1) {
            if (result.second - result.first < mn) {
                mn = result.second - result.first;
                st = result.second;
                nd = result.first;
                node = next;
            }
        } else {
            if (result.first - result.second < mn) {
                mn = result.first - result.second;
                st = result.first;
                nd = result.second;
                node = next;
            }
        }
    }

    maxScores[cur] = node;
    if (depth % 2 == 1) {
        return {nd + cur, st};
    } else {
        return {st, nd + cur};
    }    
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(NULL);

    parent[1] = 1;

    cin >> n;

    if (n == 1) {
        cout << 0;
        return 0;
    }

    for (int i = 0; i < n - 1; i++) {
        int a, b;
        cin >> a >> b;

        if (parent[a] != 0) {
            nodes[a].push_back(b);
            parent[b] = a;
        } else {
            nodes[b].push_back(a);
            parent[a] = b;
        }
    }

    initScores(1, 1);

    for (int i = 1; i <= n; i++) {
        int cur = i;
        int depth = 1;
        int a = cur, b = 0;
        while (cur != maxScores[cur]) {
            cur = maxScores[cur];
            depth++;

            if (depth % 2 == 1) {
                a += cur;
            } else {
                b += cur;
            }
        }

        if (a >= b) {
            cout << "1\n";
        } else {
            cout << "0\n";
        }
    }

    return 0;
}

```