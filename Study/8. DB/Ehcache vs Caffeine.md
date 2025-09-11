## Ehcache

- 가장 널리 사용되는 Java 기반 캐시
- 직렬화된 데이터 객체를 저장하는 메모리 블럭
- 3개의 스토리지 저장 가능 (메모리 / off Heap(GC 적용하지 않아 매우 큰 캐시 생성 가능) / 디스크)
![[Pasted image 20250911114212.png]]
- LRU / LFU / FIFO 제거 알고리즘 제공

## Caffeine cache

- high performance + 최적의 캐싱 라이브러리라고 소개
- Google 오픈소스 Guava Cache / ConcurrentLinkedHashMap을 바탕으로 만들어짐
- 캐시 제거 전략에 우선순위를 부여 가능
- 최적의 적중률을 제공하는 Window TinyLfu 제거 정책 사용

Caffeine cache를 보다 잘 이해하기 위해서는 eviction 정책을 좀 더 이해해야 한다.
아래 eviction 동작 원리를 간략히 보면
![[Pasted image 20250911114811.png]]

![[Pasted image 20250911114828.png]]

Main Cache (전체 용량 99%)
- Probation Cache(공간 80%): 자주 사용하는 데이터(제거 시 LRU rule 적용)
- Protected Cache(공간 20%): 자주 사용하지 않는 데이터(제거되지 않음)

Window Cache(전체 용량 1%)
- 새로운 데이터가 Cache에 쓰일 때 가장 먼저 Window Cache에 쓰임
- 공간에 가득 찰 경우 LRU식으로 Window Cache 밖으로 제거 (LRU)
	- Tiny LFU 알고리즘에 의해 제거되거나 Probation Cache 영역에 저장됨
- Probation 영역 데이터에 일정한 횟수 이상 접근되면 Protected Cache 역으로 승격됨
	- Protected Cache 영역이 Full 될 경우 오래된 데이터 밖으로 옮겨짐
		- TinyLFU 알고리즘에 의해 제거되거나 Probation Cache 영역에 저장됨

### TinyLFU 제거 메커니즘
