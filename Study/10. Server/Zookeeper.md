
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
