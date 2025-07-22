## 특징

- 1개의 key에 1개의 value가 매칭된다.
- value는 여러 개의 타입을 가질 수 있다.
	- MongoDB 등의 Document 형태보다 더 유연한 구조를 가질 수 있다.
- 메모리를 사용하기(HDD를 사용하지 않기) 때문에 빠른 성능을 보장한다.
	- 필요에 따라서 디스크에 저장을 하는 경우가 있다.
- 고성능의 Redis를 구동 시키려면 고사양의 인스턴스를 사용해야 한다는 비용적인 단점이 있다.
- Redis는 싱글 스레드로 동작한다.
- 영속성을 제공한다.
	- 메모리 기반으로 동작하지만, 스냅샷이나 AOP같은 방법을 사용하여 주기적으로 백업을 할 수 있다.

## Collection

### String Collection

#### SET
데이터를 그냥 문자열로 저장하는 방법, 이진 데이터도 포함되기 때문에 이미지도 저장 가능
#### SETNX
키가 존재하지 않는 경우에 대해서만 새로운 키를 저장, 일반적인 SET에 비해 성능 차이가 조금 더 우수함
#### MSET
Redis에서의 Bulk Write 방법, 네트워크I/O를 줄여주기 때문에 성능적으로 우수
- NX와 M을 조합하여 MSETNX로도 활용 가능
#### INCRBY, DECRBY
데이터가 정수형인 경우에 대해서 유효하게 동작 가능

### List Collection
일반적인 Linked List 형태로 Head와 Tail에 데이터를 삽입 할 때, 압도적인 성능을 보장한다.

많이 사용이 되지는 않지만, Job Queue 또는 Pub/Sub 모델을 구현하는 데 있어서 일부 사용이 되며, BRPOP, BLPOP과 함께 사용이 되기도 한다.

### Set Collection
특정 key에 대한 여러 개의 value를 고유하게 저장할 때 사용이 된다. 이때, Sorted Set이라는 부분도 존재하는데, 해당 타입은 랭킹과 같은 기능을 구현하는 데 있어서 매우 효과적이다.

ZADD, ZRANGE 같은 명령어를 Sorted Set에 활용해보고, SADD, SREM, SSCAN, SMIXMEMBER같은 명령어를 일반 Set에 적용해보자.

### Hashes Collection