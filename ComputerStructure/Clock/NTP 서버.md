NTP 서버는 계층으로 이루어져 있으며 그 계층을 Stratum이라고 부르는데 최상위 계층인 Stratum 0을 **PRC(Primary Reference Clock)**라고 부른다.
- 시간 원천에 직접 연결된 서버들은 Stratum 1이라고 한다.
	- Stratum 1은 Stratum 0에 동기화시킨 시간 서버이며 전세계적으로 수백 개 이상의 공식적인 1차 타임 서버가 운용중이다.
	- 이런식으로 Stratum 2 ~ 15까지 계층적 트리 구조를 형성하면서 다수의 컴퓨터에게 시간 정보를 전송해준다.