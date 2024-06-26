> [!NOTE]
> https://medium.com/naverfinancial/%EB%8B%88%EB%93%A4%EC%9D%B4-caffeine-%EB%A7%9B%EC%9D%84-%EC%95%8C%EC%95%84-f02f868a6192

### Ehcache
- 가장 널리 사용되는 JAVA 기반 캐시
- 직렬화된 데이터 객체를 저장하는 메모리 블럭
- 3개의 스토리지 저장 가능 (메모리 / off Heap(GC 적용하지 않아 매우 큰 캐시 생성 가능)/ 디스크)
![](https://miro.medium.com/v2/resize:fit:1400/1*2Lfh6ISU-OWWD66kXGV5xw.png)
- LRU / LFU / FIFO 제거 알고리즘 제공

### Caffeine cache
- high performance + 최적의 캐싱 라이브러리라고 소개 (어떤 곳은 극단적으로 local cache의 king이라고 표현 과연?)
- Google 오픈 소스 Guava Cache / ConcurrentLinkedHashMap 을 바탕으로 만들어짐
- 캐시 제거 전략에 우선순위를 부여 가능
- 최적의 적중률을 제공하는 [Window TinyLfu](https://dgraph.io/blog/refs/TinyLFU%20-%20A%20Highly%20Efficient%20Cache%20Admission%20Policy.pdf) 제거 정책 사용

Caffeine cache를 보다 잘 이해하기 위해서는 eviction 정책을 좀 더 이해를 해야 하는데요.

아래의 eviction 동작 원리를 간략히 보면
![](https://miro.medium.com/v2/resize:fit:920/1*GhabPsotTpyLR9JINNKrLg.png)
![](https://miro.medium.com/v2/resize:fit:1400/1*84pKiujtGOpbnpVgIwhpkQ.png)

Main Cache (전체 용량 99%)
- Probation Cache(공간 20%)-자주 사용하지 않는 데이터 (제거 시 LRU rule 적용)
- Protected Cache(공간 80%)-자주 사용하는 데이터 (제거되지 않음)

Window Cache (전체 용량 1%)
- 새로운 데이터가 Cache에 쓰일 때 가장 먼저 Window Cache에 쓰임
- 공간에 가득 찰 경우 LRU 식으로 Window Cache 밖으로 제거 (LRU)
- — Tiny LFU 알고리즘에 의해 제거되거나 Probation Cache 영역에 저장됨
- Probation 영역 데이터에 일정한 횟수 이상 접근되면 Protected Cache 역으로 승격됨
- — Protected Cache 영역이 Full 될 경우 오래된 데이터 밖으로 옮겨짐
- — — TinyLFU 알고리즘에 의해 제거되거나 Probation Cache 영역에 저장됨

[**TinyLFU 제거 메커니즘**](https://www.sobyte.net/post/2022-04/caffeine/)
- Window Cache / Protected Cache로부터 제거되는 데이터 = Candidate
- Probation Cache에서 제거되는 데이터 = Victim
- Candidate Cache 접근 > Victim Cache 접근 : Victim 데이터 제거
- Candidate Cache 접근 < Victim Cache 접근 && Candidate 접근 횟수 5번 이하 : Candidate 데이터 제거
- 둘 중 하나 랜덤하게 제거

Caffeine 캐시 내부 알고리즘은 LFU와 LRU의 장점을 통합
- 서로 다른 캐시 영역에 다른 특성을 가진 캐시 항목을 저장하여 최근에 생성된 캐시 데이터가 Window Cache로 들어가 삭제되지 않음
- 자주 호출되는 데이터 (LFU)은 Protected 영역에 들어가며 LRU에 의해 제거되지 않음
- 호출 횟수 / 호출 시간 두 개의 자원에 대해 밸런스가 잘 되어 있음
- — 자주 호출되고 최근에 생성된 데이터 들은 가능한 캐시에 유지 시킬수 있음
- 전통적인 LRU/LFU 로 처리 할 수 없던 케이스를 보다 잘 처리함

[벤치마크 (정말 좋은가?)](https://github.com/ben-manes/caffeine/wiki/Benchmarks)

![](https://miro.medium.com/v2/resize:fit:1050/1*I_kC3IaGiiLzAE9QunEYXQ.png)

동일/분산 key에서 Throughput도 다른 cache에 비해 압도적으로 좋고

![](https://miro.medium.com/v2/resize:fit:1050/1*CaahlaBBKSbRs82V8gNwjw.png)

Read 처리도 기본 ConcurrentLinkedHashMap/ehcache에 비해 압도적으로 좋고

![](https://miro.medium.com/v2/resize:fit:1050/1*p2Gc0RbRmWAL4s64DW32Rw.png)

Read/Write가 있어도 ConcurrentLinkedHashMap/ehcache에 비해 압도적으로 좋고

![](https://miro.medium.com/v2/resize:fit:1050/1*ycL5vug6XrL1OV8dfwoeAQ.png)

Write만 해도ConcurrentLinkedHashMap/ehcache에 비해 압도적으로 좋고

물론 caffeine cache에서 진행한 벤치마크이기 때문에 어느 정도 감안은 하고 봐야겠지만,

모든 부분에서 벤치마크 결과가 압도적으로 좋게 나오고 있습니다.

대부분의 경우 로컬 캐시를 매우 다양한 처리를 하기 위함이 아니라 단순히 값을 임시로 저장/조회하는 용도로 사용하기에도 벤치마크 결과만 보면 Caffeine Cache를 사용하지 않을 이유가 딱히 떠오르질 않는 거 같습니다.

## Ehcache vs Caffeine 비교

- Ehcache 가 Caffeine에 비해 제공되는 기능은 더 많음
	- multi-level cache, distributed cache, cahce listener …
- 단순 메모리 캐시 사용하고 높은 퍼포먼스를 원하면 Caffeine이 우선순위가 높음 (Ehcache에 비해 제거 알고리즘이 우월함)
	-  Window TinyLfu eviciton policy로 near-optimal hit rate
	-  Caffeine이 ehcache에 비해 writing 퍼포먼스가 좋다고 함
- Eviction strategy
	-  Caffeine (size-based, time-based, reference-based)
	-  Ehcache (LRU, LFU, FIFO) — 메모리 관련만 가능
- ehcache는 메모리 + 디스크 용량까지 사용 가능
	-  caffeine 도 post-eviction strategy로 커스텀 하게 구출할 수 있음
		-  최근 삭제 캐시를 잡아서 로드 시점에 다시 살리도록 가능(DB가 죽는것 같은 경우)

![](https://miro.medium.com/v2/resize:fit:1010/1*V8KYGvs8aDBOsNvZ53eAnA.png)

- Caffeine 은 캐시 사용 시 3가지 전략을 제공 (manual, synchronous, asynchronous loading)
- — Ehcache는 asynchronous loading 불가
- — Caffeine이 좀 더 사용하기 쉬움