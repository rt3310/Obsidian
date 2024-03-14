Mark And Sweep 방식은 루트로부터 해당 객체에 접근이 가능한지가 해제의 기준이 된다.
JVM GC에서의 Root Space는 Heap 메모리 영역을 참조하는 method area, static variables, stack, native method stack이 되게 된다.

![[root space.png]]