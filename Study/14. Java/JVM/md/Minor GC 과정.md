1. 처음 생성된 객체는 Young Generation 영역의 일부인 **Eden 영역**에 위치
   ![[minor gc 1.png]]
   ![[minor gc 2.png]]
2. 객체가 계속 생성되어 Eden 영역이 꽉차게 되고 **Minor GC**가 실행
   ![[minor gc 3.png]]
3. **Mark** 동작을 통해 **reachable 객체를 탐색**
   ![[minor gc 4.png]]
4. Eden 영역에서 살아남은 객체는 1개의 Survivor 영역으로 이동
   ![[minor gc 5.png]]
5. Eden 영역에서 **사용되지 않는 객체(unreachable)의 메모리를 해제(sweep)**
   ![[minor gc 6.png]]
6. 살아남은 모든 객체들은 **age 값이 1씩 증가**
   ![[minor gc 7.png]]
7. 또다시 Eden 영역에 **신규 객체들로 가득 차게 되면 다시 한 번 minor GC가 발생**하고 mark 한다.
   ![[minor gc 8.png]]
   ![[minor gc 9.png]]
   ![[minor gc 10.png]]
8. marking한 객체들을 비어있는 survivor 1로 이동하고 sweep
   ![[minor gc 11.png]]
   ![[minor gc 12.png]]
9. 다시 살아남은 모든 객체들은 age가 1씩 증가
   ![[minor gc 13.png]]
10. 이러한 과정을 반복
    ![[minor gc 14.gif]]