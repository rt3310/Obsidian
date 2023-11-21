Old Generation 영역은 **길게 살아남는 메모리들이 존재하는 공간**이다.

![[Major GC.png]]
Old Generation의 객체들은 거슬러 올라가면 처음에는 Young Generation에 의해 시작되었으나, GC 과정 중에 제거되지 않은 경우 age 임계값이 차게되어 이동된 녀석들이다.
그리고 Major GC는 객체들이 **계속 Promotion되어 Old 영역의 메모리가 부족해지면 발생**하게 된다.