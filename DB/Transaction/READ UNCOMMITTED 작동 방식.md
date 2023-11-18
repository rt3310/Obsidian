예를 들어 사용자 A의 트랜잭션에서 INSERT를 통해 데이터를 추가했다고 하자. 아직 커밋 또는 롤백이 되지 않는 상태임에도 불구하고 READ UNCOMMITTED는 변경된 데이터에 접근할 수 있다.
![[read uncommitted 1.png]]