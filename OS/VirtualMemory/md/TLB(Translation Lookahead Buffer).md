- 근래의 가상기억장치를 사용하는 처리기(CPU)는 **하나 이상의 TLB**를 가지고 있다.
- 최근에 처리된 **페이지 테이블의 정보를 갖는 고속 캐시**이다.
	- TLB를 사용하지 않으면 최소 2번의 메모리 접근 시간(페이지 테이블 + 데이터)이 필요한데 이 중에서 **페이지 테이블 접근을 고속 캐시로 빠르게 처리**하는 것 -> **1번의 주기억장치 접근**
- TLB는 처리기와 처리기 캐시 사이, 처리기 캐시와 주기억장치 사이 등 여러가지 다른 레벨의 캐시들 사이에 존재하여 가상주소를 물리주소로 변환하는데 사