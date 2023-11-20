- Java 15에 release
- 대량의 메모리(8MB ~ 16TB)를 low-latency로 잘 처리하기 위해 디자인 된 GC
- G1의 Region 처럼, ZGC는 ZPage라는 영역을 사용하며, G1의 Region은 크기가 고정인데 비해, ZPage는 2mb 배수로 동적으로 운영됨
	- 큰 객체가 들어오면 2^ 로 영역을 구성해서 처리
- ZGC가 내세우는 최대 장점 중 하나는 힙 크기가 증가하더라도 'stop-the-world'의 시간이 절대 10ms를 넘지 않는다는 것

![[ZGC.png]]