MySQL에는 갭 락이 존재하기 때문에 전의 상황에서 문제가 발생하지 않는다. 사용자 B가 SELECT FOR UPDATE로 데이터를 조회한 경우에 MySQL은 id가 50인 레코드에는 레코드 락, id가 50보다 큰 범위에는 갭 락으로 넥스트 키 락을 건다.
따라서 사용자 A가 id가 51인 member를 INSERT 시도한다면, B의 트랜잭션이 종료(커밋 또는 롤백)될 때까지 기다리다가 대기를 지나치게 오래하면 락 타임아웃이 발생하게 된다.

![[gap lock 잠금 대기.png]]

따라서 일반적으로 MySQL의 REPEATABLE READ에서는 Phantom Read가 발생하지 않는다.
MySQL에서 Phantom Read가 발생하는 거의 유일한 케이스는 다음과 같다.

사용자 B는 트랜잭션을 시작하고, 잠금없는 SELECT 문으로 데이터를 조회하였다. 그리고 사용자 A는 INSERT문을 사용해 데이터를 추가하였다. 이때 잠금이 없으므로 바로 COMMIT 된다. 하지만 사용자 B가 SELECT FOR UPDATE로 조회를 했다면, 언두 로그가 아닌 테이블로부터 레코드를 조회하므로 Phantom Read가 발생한다.

![[MySQL에서의 Phantom Read.png]]
하지만 이런 케이스는 거의 존재하지 않으므로, MySQL의 REPEATABLE READ에서는 Phantom Read가 발생하지 않는다고 봐도 된다.

- **SELECT FOR UPDATE 이후** 
	- **SELECT**: 갭락으로 인해 Phantom Read X
	- **SELECT FOR UPDATE**: 갭락으로 인해 Phantom Read X
- **SELECT 이후**
	- **SELECT**: MVCC로 인해 Phantom Read X
	- **SELECT FOR UPDATE**: Phantom Read O