iptables는 시스템 관리자가 리눅스 커널 방화벽이 제공하는 테이블들과 그것을 저장하는 체인, 규칙들을 구성할 수 있게 해주는 사용자 공간 응용 프로그램이다.
각기 다른 커널 모듈과 프로그램들은 현재 다른 프로토콜을 위해 사용되는데, iptables는 IPv4에 ip6tables는 IPv6에, arptables는 ARP에 ebtables는 이더넷 프레임에 적용된다.

즉,
1. netfilter라는 네트워크 관련 API의 Wrapper이다.
2. 방화벽 테이블, 체인, 규칙을 쉽게 쓸 수 있게 해준다.
3. ip6tables, arptables, ebtables 등이 있다.
4. 리눅스 최신 버전은 nftables를 쓰고, 이것은 위의 것들을 전부 통합한 것이다.