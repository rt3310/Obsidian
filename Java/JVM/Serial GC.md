- 서버의 CPU 코어가 1개일 때 사용하기 위해 개발된 가장 단순한 GC
- GC를 처리하는 **쓰레드가 1개** (싱글 쓰레드)여서 **가장 stop-the-world 시간이 길다**.
- Minor GC에는 **Mark-Sweep**을 사용하고, Major GC에는 **Mark-Sweep-Compact**를 사용한다.
- 보통 실무에서 사용하는 경우는 없다. (디바이스 성능이 안좋아서 CPU 코어가 1개인 경우에만 사용)

![[Serial GC.png]]