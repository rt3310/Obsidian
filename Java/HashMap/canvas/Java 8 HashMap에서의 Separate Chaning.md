Java 2부터 Java 7까지의 HashMap에서 Separate Chaining 구현 코드는 조금씩 다르지만, 구현 알고리즘 자체는 같았다. 만약 객체의 해시 함수 값이 균등 분포(uniform distribution) 상태라고 할 때, get() 메서드 호출에 대한 기댓값은![hashmap5](https://d2.naver.com/content/images/2015/06/helloworld-831311-5.png)이다. 그러나 **Java 8에서는 이보다 더 나은![hashmap6](https://d2.naver.com/content/images/2015/06/helloworld-831311-6.png)을 보장한다. 데이터의 개수가 많아지면, Separate Chaining에서 링크드 리스트 대신 트리를 사용하기 때문**이다.

데이터의 개수가 많아지면![hashmap7](https://d2.naver.com/content/images/2015/06/helloworld-831311-7.png)과![hashmap8](https://d2.naver.com/content/images/2015/06/helloworld-831311-8.png)의 차이는 무시할 수 없다. 게다가 실제 해시 값은 균등 분포가 아닐뿐더러, 설사 균등 분포를 따른다고 하더라도 birthday problem이 설명하듯 일부 해시 버킷 몇 개에 데이터가 집중될 수 있다. 그래서 데이터의 개수가 일정 이상일 때에는 링크드 리스트 대신 트리를 사용하는 것이 성능상 이점이 있다.

링크드 리스트를 사용할 것인가 트리를 사용할 것인가에 대한 기준은 하나의 해시 버킷에 할당된 키-값 쌍의 개수이다. 예제 5에서 보듯 Java 8 HashMap에서는 상수 형태로 기준을 정하고 있다. 즉 **하나의 해시 버킷에 8개의 키-값 쌍이 모이면 링크드 리스트를 트리로 변경**한다. 만약 **해당 버킷에 있는 데이터를 삭제하여 개수가 6개에 이르면 다시 링크드 리스트로 변경**한다. **트리는 링크드 리스트보다 메모리 사용량이 많고, 데이터의 개수가 적을 때 트리와 링크드 리스트의 Worst Case 수행 시간 차이 비교는 의미가 없기 때문**이다. 8과 6으로 2 이상의 차이를 둔 것은, 만약 **차이가 1이라면 어떤 한 키-값 쌍이 반복되어 삽입/삭제되는 경우 불필요하게 트리와 링크드 리스트를 변경하는 일이 반복되어 성능 저하가 발생할 수 있기 때문**이다.
```java
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
```
Java 8 HashMap에서는 Entry 클래스 대신 Node 클래스를 사용한다. Node 클래스 자체는 사실상 Java 7의 Entry 클래스와 내용이 같지만, **링크드 리스트 대신 트리를 사용할 수 있도록 하위 클래스인 TreeNode가 있다는 것이 Java 7 HashMap과 다르다**.

이때 사용하는 트리는 **Red-Black Tree**인데, Java Collections Framework의 **TreeMap과 구현이 거의 같다**. 트리 순회 시 사용하는 **대소 판단 기준은 해시 함수 값**이다. 해시 값을 대소 판단 기준으로 사용하면 **Total Ordering**에 문제가 생기는데, Java 8 HashMap에서는 이를 **tieBreakOrder() 메서드로 해결**한다.

```java
transient Node<K, V>[] table;

static class Node<K, V> implements Map.Entry<K, V> {
	// 클래스 이름은 다르지만, Java 7의 Entry 클래스와 구현 내용은 같다.
}

// LinkedHashMap.Entry는 HashMap.Node를 상속한 클래스다.
// 따라서 TreeNode 객체를 table 배열에 저장할 수 있다.
static final class TreeNode<K, V> extends LinkedHashMap.Entry<K, V> {
	TreeNode<K, V> parent;
	TreeNode<K, V> left;
	TreeNode<K, V> right;
	TreeNode<K, V> prev;

	boolean red; // Red Black Tree에서 노드는 Red이거나 Black이다.

	TreeNode(int hash, K key, V val, Node<K, V> next) {
		super(hash, key, val, next);
	}

	final TreeNode<K, V> root() {
		// Tree 노드의 root를 반환한다.
	}

	static <K, V> void moveRootToFront(Node<K, V>[] tab, TreeNode<K, V> root) {
		// 해시 버킷에 트리를 저장할 때에는, root 노드에 가장 먼저 접근해야 한다.
	}

	// 순회하며 트리 노드 조회
	final TreeNode<K, V> find(int h, Object k, Class<?> kc) {}
	final TreeNode<K, V> getTreeNode(int h, Object k) {}

	static int tieBreakOrder(Object a, Object b) {
		// TreeNode에서 어떤 두 키의 comparator 값이 같다면 서로 동등하게 취급한다.
		// 그런데 어떤 두 개의 키 hash 값이 서로 같아도 이 둘은 서로 동등하지 않을 수 있다.
		// 따라서 어떤 두 개의 키에 대한 해시 함수 값이 같을 경우, 임의로 대소 관계를 지정할 필요가 있는 경우가 있다.
	}

	final void treeify(Node<K, V>[] tab) {
		// LinkedList로 트리를 변환한다.
	}

	final Node<K, V> untreeify(HashMap<K, V> map) {
		// 트리를 링크드 리스트로 변환한다.
	}

	// 다음 두 개 메서드의 역할은 메서드 이름만 읽어도 알 수 있다.
	final TreeNode<K, V> putTreeVal(HashMap<K, V> map, Node<K, V>[] tab,
								int h, K k, V v) { }
	final void removeTreeNode(HashMap<K, V> map, Node<K, V>[] tab, boolean movable) { }

	// Red Black 구성 규칙에 따라 균형을 유지하기 위한 것이다.
	final void split(...)
	static <K, V> TreeNode<K, V> rotateLeft(...)
	static <K, V> TreeNode<K, V> rotateRight(...)
	static <K, V> TreeNode<K, V> balanceInsertion(...)
	static <K, V> TreeNode<K, V> balanceDeletion(...)

	static <K, V> boolean checkInvariants(TreeNode<K, V> t) {
		// Tree가 규칙에 맞게 잘 생성된 것인지 판단하는 메서드다.
	}
}
```