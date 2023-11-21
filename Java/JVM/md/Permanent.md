Permanent는 직역하면 영구적인 세대의 의미로서, **생성된 객체들의 정보의 주소 값이 저장된 공간**이다.
클래스 로더에 의해 load되는, **Class, Method 등에 대한 Meta 정보, Static 변수, 상수 정보가 저장되는 영역**이고 **JVM에 의해 사용**된다.

Java 7까지는 힙 영역에 존재했지만 **Java 8 버전 이후에는 Native Method Stack에 편입**되어 Metaspace 영역으로 변경되었다.
(다만, 기존 Perm 영역에 존재하던 Static Object는 Heap 영역으로 옮겨져서 GC의 대상이 최대한 될 수 있도록 하였다)

![[permanent.png]]