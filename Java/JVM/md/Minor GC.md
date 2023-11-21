Young Generation 영역은 **짧게 살아남는 메모리들이 존재하는 공간**이다

![[Minor GC.png]]
모든 객체는 처음에는 Young Generation에 생성되게 된다.
Young Generation의 공간은 **Old Generation에 비해 상대적으로 작기 때문에 메모리 상의 객체를 찾아 제거하는데 적은 시간이 걸린다**.

이 때문에 **Young Generation 영역에서 발생되는 GC**를 **Minor GC**라 불린다.