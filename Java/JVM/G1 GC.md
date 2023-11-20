- CMS GC를 대체하기 위해 jdk 7 버전에서 최초로 release된 GC
- **Java 9+ 버전의 default GC**로 지정
- **4GB 이상의 힙 메모리, Stop the World 시간이 0.5초 정도 필요한 상황**에 사용 (**Heap이 너무 작을 경우 미사용 권장**)
- 기존의 GC 알고리즘에서는 Heap 영역을 물리적으로 고정된 Young / Old 영역으로 나누어 사용하였지만, G1 GC는 이러한 개념을 뒤엎는 **Region**이라는 개념을 새로 도입하여 사용.
	- 전체 Heap 영역을 Region이라는 영역으로 **체스같이 분할아여 상황에 따라 Eden, Survivor, Old 등 역할을 고정이 아닌 동적으로 부여**
- Garbage로 가득찬 영역을 빠르게 회수하여 빈 공간을 확보하므로, 결국 GC 빈도가 줄어드는 효과를 얻게 되는 원리

![[Pasted image 20231120144302.png]]
 ![[Pasted image 20231120144351.png]]