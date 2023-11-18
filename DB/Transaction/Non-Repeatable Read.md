READ COMMITTED는 Non-Repeatable Read(반복 읽기 불가능) 문제가 발생할 수 있다.
예를 들어 사용자 B가 트랜잭션을 시작하고 name="Minkyu"인 레코드를 조회했다고 하자. 해당 조건을 만족하는 레코드는 아직 존재하지 않으므로 아무것도 반환되지 않는다.
![[Pasted image 20231119004800.png]]