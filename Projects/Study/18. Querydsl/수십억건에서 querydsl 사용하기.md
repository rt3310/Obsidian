## exist 메소드 금지

![[Pasted image 20250112011048.png]]

Querydsl의 exists는 실제로 count() > 0 으로 실행된다.
![[Pasted image 20250112010946.png]]

![[Pasted image 20250112011136.png]]

exists가 빠른 이유는 **조건에 해당하는 row 1개**만 찾으면 바로 쿼리를 종료하기 때문!
이를 직접 구현하자

![[Pasted image 20250112012400.png]]
![[Pasted image 20250112012416.png]]

## Cross Join 회피

![[Pasted image 20250112022321.png]]
![[Pasted image 20250112022401.png]]