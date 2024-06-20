
## 1. 성능의 문제
`System.out.println()`

![[Pasted image 20240620190654.png]]
synchronized block을 감싸 thread-safe하게 처리하기 때문에 성능 저하가 올 수 있다.

> [!QUESTION]
> 그럼 logging 라이브러리들은 thread safe 하지 않은건가?
> 
> 스프링에서 기본적으로 추가되는 logging 라이브러리들(log4j2, logback)은 thread safe하게 구현되어 있다.
> 다만 synchroinzed block을 사용하는 경우는 매우 드물며, 내부적으로 로깅 이벤트를 처리


## 2. 로그 레벨 관리 문제

## 3. 유지보수성 저하
