
## ZooKeeper

Apache ZooKeeper는 분산 환경에서 여러 서버(노드)들이 동일한 상태와 설정을 공유할 수 있도록 돕는 중앙 집중형 코디네이터(coordinator) 서비스이다.

즉, 분산 시스템의 두뇌 역할을 하는 시스템이다.

## 왜 필요한가?

분산 시스템에서는 서버가 여러 대이기 때문에 다음과 같은 문제들이 발생한다.
- 어떤 서버가 리더(Leader)인지 알아야 한다.
- 여러 서버가 같은 설정(Configuration)을 공유해야 한다.
- 특정 서버가 죽었을 때 빠르게 감지해야 한다.
- 여러 서버가 동시에 같은 자원(Resource)을 수정하려 하면 충돌이 발생한다.
이런 합의(Consensus)와 동기화(Synchronization) 문제를 해결하기 위해 Zookeeper가 사용된다.

## 분산 코디네이션 서비스

분산 시스템에서 시스템 간의 정보 공유, 상태 체크, 서버들 간의 동기화를 위한 락 등을 처리해주는 서비스
![[Pasted image 20251015201226.png]]
ZooKeeper는 분산 시스템의 일부분이기 때문에 동작을 멈춘다면 분산 시스템이 멈출 수도 있다. 그래서 안정성을 확보하기 위해 클러스터로 구축한다.
클러스터는 홀수로 구축한다. 어떤 서버에 문제가 생겼을 경우 과반수 이상의 데이터를 기준으로 일관성을 맞추기 때문이다. 살아있는 노드가 과반수 이상이라면 지속적인 서비스를 제공한다 (위 그림에서 Server는 Zookeeper, Client는 Kafka가 된다)
서버 여러 대를 앙상블(클러스터)로 구성하고, 분산 애플리케이션들이 각각 클라이언트가 되어 Zookeeper 서버들과 커넥션을 맺을 후 상태 정보 등을 주고 받는다.

상태 정보들은 ZooKeeper의 znode라고 불리는 곳에 Key-Value 형태로 저장하며, znode에 저장된 것을 이용하여 분산 애플리케이션들은 서로 데이터를 주고받게 된다. (znode를 일반 컴퓨터의 파일이나 폴더 개념으로 생각하면 쉽다)
znode는 우리가 알고 있는 일반적인 디렉토리와 비슷한 형태로서 자식노드를 가지고 있는 계층형 구조로 구성되어 있다.

구성
- Request Processor: Write 요청 처리
- Zab(Zookeeper Atomic Broadcast Protocol): Request Processor에서 처리한 요청을 트랜잭션을 생성하여 모든 서버에게 전파한다.
	- \[Leader-Propose]→\[Follower-Accept]→\[Leader-Commit] 단계로 구성된다.
- In-memory DB: Znode의 정보가 저장되며, 로컬 파일시스템에 Replication을 구성할 수 있다.

## ZNode

ZooKeeper는 트리 형태의 데이터 구조를 가지는데, 여기서 각 노드를 ZNode라고 부른다.

![[Pasted image 20251015211413.png]]

- Persistent Node: 영구 저장소
- Ephermeral Node: Client가 종료되면 사라진다.
- Sequence Node: 생성 시 뒤에 숫자가 붙는다.

## Watcher

ZooKeeper는 ZNode에 변화를 감지할 수 있는 Watcher를 클라이언트가 설정할 수 있도록 한다.
Watcher는 자신이 감시하고 있는 ZNode에 수정이 발생했을 때, 클라이언트로 callback 호출을 전송하는 알림 기능을 제공한다.

## Quorum

Leader가 새로운 트랜잭션을 수행하기 위해서는 자신을 포함하여 과반수 이상의 서버의 합의를 얻어야 한다. 이때, 과반수의 합의를 위해 필요한 서버들을 Quorum이라고 한다.
Ensemble을 구성하는 서버의 수가 5개라면, Quorum은 3개의 서버로 구성이 된다.

### 트랜잭션 처리
1. Leader에게 Request 전달
	- 새로운 트랜잭션 요청이 Follower에게 도착했을 경우, Follower는 Leader에게 요청을 전달한다.
2. Propose
	- Propose는 Leader가 Quorum을 구성하는 서버들에게 트랜잭션을 수행해도 되는지 여부를 요청하는 과정을 의미한다.
3. Ack
	- Quorum을 구성하는 서버들은 Leader로 부터 Propose 요청을 받으면, 트랜잭션을 수행해도 된다는 Ack 응답을 Leader에게 전송한다.
4. Commit
	- 모든 Quorum으로 부터 Ack를 받으면, Leader는 트랜잭션을 처리하라는 Commit 명령을 broadcast 형태로 모든 Follower에 전파한다.
	- ZooKeeper에서는 Commit 명령을 전달할 때, ZAB(ZooKeeper Atomic Broadcast) 알고리즘을 사용한다.
	- Atomic Broadcast는 broadcast 방식 중 하나로, 멀티 프로세스 시스템에서 모든 프로세스에게 동일한 순서로 메시지가 전달된다는 것을 의미한다.

## ZooKeeper를 짝수로 구성한 경우 생기는 문제점

크게 문제는 없으나, 짝수로 구성한 경우 쿼럼(Quorum)을 형성할 때 비례적으로 노드 수가 더 필요하므로 잘 사용되지 않는다.(4대로 구성한 경우 3대 필요)