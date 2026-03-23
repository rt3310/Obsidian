## RAID?

RAID는 Redundant Array of Independent Disk(독립된 디스크의 복수 배열) 혹은 Redundant Array of Inexpensive Disk(저렴한 디스크의 복수 배열)의 약자이다.

말 그대로 RAID는 여러 개의 디스크를 묶어 하나의 디스크처럼 사용하는 기술이다.

RAID를 사용했을 때 기대 효과는
- 대용량의 단일 불륨을 사용하는 효과
- 디스크 I/O 병렬화로 인한 성능 향상 (RAID 0, RAID 5, RAID 6 등)
- 데이터 복제로 인한 안정성 향상 (RAID 1 등)
이 있다.

RAID는 컴퓨터를 구성하는 여러 부품 중 기계적인 특성 때문에 상대적으로 속도가 (많이) 느린 하드디스크를 보완하기 위해 만든 기술이다.

RAID를 구성하는 디스크의 개수가 같아도 RAID의 구성 방식에 따라 성능, 용량이 바뀌게 된다.
이 구성 방식을 RAID Level(레이드 레벨)이라고 부른다.

## Standard RAID Level

먼저 기본적인 RAID Level 이다.
RAID 0 ~ RAID 6까지 있지만, 최근 출시되는 RAID 컨트롤러에서 사용가능 한 RAID Level은 RAID 0, RAID 1, RAID 5, RAID 6이다.

RAID를 구성하는 디스크의 종류와 크기는 같다고 가정한다.
- 성능의 경우 RAID 컨트롤러의 연산으로 인한 성능 저하는 제외하고, Sequential I/O 시만 가정한다.

### RAID 0
![[Pasted image 20260323023854.jpg]]