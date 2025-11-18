## JVM

![[Pasted image 20251118123727.png]]
### JVM Heap
![[Pasted image 20251118123741.png]]위 그림은 JDK 1.7 버전 이전의 Heap 메모리 구조이다.

- Eden: 새로 생성한 대부분의 객체가 위치하는 곳
- S0, S1: Eden 영역에서 GC가 한 번 발생한 후 살아남은 객체들이 존재하는 곳
- Old Memory: Young Generation에 대한 GC가 반복되는 과정 속에 살아남은 객체가 있는 곳
	- 특정 횟수 이상 참조되어 Old 영역으로 가기 위한 age를 달성했을 때 이동하게 된다.
- Perm: Class / Method의 Meta 정보, static 변수/상수들이 저장되는 곳

우리가 아는 Garbage Collection도 위와 같은 가정을 두고 언급한 것들이 많다. 하지만 지금도 그럴까?

![[Pasted image 20251118124000.png]]
JDK 1.8 이후, Perm이 Metaspace로 바뀌었다. 또 어떤 부분이 더 바뀌었을까?

| 구분     | Perm                                | Metaspace                                  |
| ------ | ----------------------------------- | ------------------------------------------ |
| 저장 정보  | 클래스 meta / 메소드 meta / static 변수, 상수 | 클래스 meta / 메소드 meta                        |
| 관리 포인트 | Heap 영역 튜닝 + Perm 영역 별도             | Native 영역 동적 조정                            |
| GC     | Full GC                             | Full GC                                    |
| 메모리 측면 | -XX: PermSize / -XX: MaxPermSize    | -XX: MetaSpaceSize / -XX: MaxMetaspaceSize |

가장 중요한 핵심은 Perm영역이 Heap이 아니라 **Native** 영역으로 바뀌었다는 것이다.
Native 영역의 가장 큰 특징 중 하나는 Native 영역은 JVM에 의해서 크기가 강제되지 않고, 프로세스가 이용할 수 있는 메모리 자원을 최대로 활용할 수 있다.
만약 메모리 leak이 Classloader를 동작하는 코드에 발생하는 것으로 의심된다면, 이는 최대 메모리를 설정하지 않았기 때문이다.

## Garbage Colleciton

Garbage Collection은 객체가 접근 불가능한 상태(Unreachable)가 되었을 때, 메모리가 누적되므로 이를 수거하는 작업을 의미한다.
이 때, 작업(GC)를 실행하는 스레드를 제외한 나머지 스레드는 모두 작업을 멈춘다. 이를 **stop the world**라고 한다.

우리가 알고 있는 GC 튜닝이란, 이 stop the world 시간을 줄이는 것이다.

먼저, Heap 영역은 크게 2가지로 구성이 되어있다.
- Young Generation 영역: 대부분의 객체가 GC되는 영역, 이 영역에서 객체가 사라질 때 **Minor GC**가 발생
- Old Generation 영역: Young 영역보다 크게 할당되지만, GC는 적게 발생. 이 때는 **Full GC**라고 일컫음

이 특징으로 인해 Full GC는 stop the world 시간이 길 수 밖에 없다. 기본적으로 메모리가 크고, 처리해야 될 양이 많기 때문이다.

이 Old 영역에 대한 GC를 다르게 하기 위해서 많은 알고리즘들이 존재한다.
### GC 알고리즘
- **Serial GC**: Heap이 앞부분부터 확인하여 살아있는 것만 남기고(Sweep), 객체들이 연속되도록 Compaction하는 작업
	- 기본적으로 Mark-Sweep-Compaction 알고리즘에 해당
	- Serial GC는 적은 메모리와 CPU 코어 개수가 적을 때 가장 올바르다(싱글 스레드)
- **Parallel GC**: Serial GC의 멀티스레드 버전
- **Parallel Old GC**: Parallel GC와 다른 점은 Mark-Summary-Compaction 단계를 거쳐서 객체를 식별
	- Summary에 해당하는 작업이 GC를 수행한 영역에 대해서 살아있는 객체를 식별한다는 작업
- Concurrent Mark & Sweep GC = **CMS GC**: CMS GC는 다른 GC와는 다르게 Compaction을 진행하지 않는다.
- G1 GC: 뒤에 설명
- ZGC: 뒤에 설명
### Mark & Sweep
- Initial Mark: 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾는다.
- Concurrent Mark: 위에서 살아있다고 확인한 객체에서 참조되고 있는 객체를 확인한다.
- Remark: 위 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.
- Concurrent Sweep: 쓰레기를 정리한다.

## G1GC(Garbage First GC)

G1GC는 위 GC 알고리즘들과 메커니즘이 많이 다르다. 위에서 언급한 Young 영역과 Old 영역에 대한 GC는 잠시 잊는게 좋다.
G1GC는 JDK 11 부터 공식적인 GC 알고리즘으로 적용되었고, 하드웨어가 점점 발전하면서 대용량 메모리에 적합한 솔루션을 제공하기 위해 나타났다.

G1GC는 앞서 언급했던 Eden, Survivor, Old 영역이 존재하지만, 해당 영역은 **고정된 크기가 아니며 전체 Heap 메모리 영역을 Region이라는 특정한 크기로 나눈 것**이고 Region의 상태에 따라 그 Region의 역할(Eden, Survivor, Old)가 **동적으로 변동**한다.
Region은 기본적으로 $(전체 Heap 메모리) / 2048$로 default 값이 지정되어 있다.

이를 그림으로 나타내면 아래와 같다.
![[Pasted image 20251118130022.png]]
여기서 다음과 같은 새롭게 보이는 영역들이 존재한다.
- Humonogous: Region 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간
- Available/Unused: 아직 사용되지 않은 Region

G1GC에서도 마찬가지로 **MinorGC**가 존재하며, 이 과정에서 살아남은 객체들을 Survivor Region으로 옮기고, Eden에 대한 영역을 사용가능한(Available) Region으로 돌리는 형태로 과정이 일어나게 된다.

G1GC에는 Full GC와 유사한 **Concurrent Cycle** 이라는 과정이 존재하는데, 해당 과정은 **IHOP(InitiatingHeapOccupancyPercent)에서 정한 수치를 초과하면 실행**하게 된다.

### G1GC 과정
![[Pasted image 20251118130310.png]]
- **Initial Mark**: Old Region에 존재하는 객체들이 참조하는 Survivor Region을 찾는다(STW)
- **Root Region Scan**: 위에서 찾은 Survivor 객체들에 대한 스캔 작업을 실시한다.
- **Concurrent Mark**: 전체 Heap의 Scan 작업을 실시하고, GC 대상 객체가 발견되지 않은 Region은 이후 단계를 제외한다.
- **Remark**: 애플리케이션을 멈추고(STW) 최종적으로 GC 대상에서 제외할 객체를 식별한다.
- **Cleanup**: 애플리케이션을 멈추고(STW) 살아있는 객체가 가장 적은 Region에 대한 미사용 객체를 제거한다.
- **Copy**: GC 대상의 Region이었지만, Cleanup 과정에서 완전히 비워지지 않은 Region의 살아남은 객체들을 새로운 Region(Avaliable/Unused)에 복사하여 Compaction을 수행한다.
- 살아있는 객체가 아주 적은 Old 영역에 대해 [GC pause(mixed)]를 로그로 표시하고, Young GC가 이루어질 때 수집되도록 한다.