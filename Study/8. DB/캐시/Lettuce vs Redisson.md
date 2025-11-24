## Lettuce

Lettuce는 Java용 Redis 클라이언트 라이브러리로, Netty 기반의 비동기/동시성 친화적 클라이언트다.
동기, 비동기, 리액티브 API를 모두 제공해 다양한 사용 패턴(블로킹/논블로킹/리액티브)에 맞춰 쓸 수 있다.

### 기본 원리
- Netty 이벤트 루프를 사용해 논블로킹 I/O를 구현한다.
- 내부적으로는 하나의 RedisClient를 애플리케이션 전체에서 재사용하고, 필요에 따라 `StatefulRedisconection`(단일 connection)이나 `StatefulRedisClusterConnection`을 얻어 `RedisCommands`(sync), `RedisAsyncCommands`(async), `RedisReactiveCommands`(reactive)로 작업한다.
- 명령은 Redis 프로토콜에 맞춰 프레임화되어 전송되고, 비동기 호출은 `RedisFuture`를 반환한다.
### 필요성
- 고성능, 낮은 Latency, 많은 동시성(수천~수만 커넥션이 아닌 수천의 동시 명령) 요구
- 리액티브 스택(ex: Reactor, WebFlux)과 자연스럽게 통합해야 할 때
- 스레드에 안전한(singleton) 클라이언트를 원할 때
### 장단점
#### 장점
- 성능: Netty 기반, 논블로킹으로 높은 처리량
- 다양한 API: sync/async/reactive 모두 제공
- Thread Safe: `RedisClient`와 `StatefulRedisConnection`은 재사용 권장
- Cluster / Sentinel 지원: Redis Cluster/Sentinel 환경 지원
- 경량: 낮은 오버헤드, Latency 민감한 작업에 적합
#### 단점/주의점
- 저수준: Redis 원시 명령 중심. 분산 락, 분산 컬레션 같은 고수준 추상은 직접 구현하거나 별도 라이브러리 필요
- 관리 요구: connection lifecycle, pooling, 재시도/backoff, failover 처리 등을 직접 설계해야 함
- 블로킹 sync 사용 시 주의: sync API는 호출 스레드를 블로킹하므로 비동기 환경에서는 피해야 함
### 실무 팁 / 운영 주의
- `RedisClient`는 애플리케이션 단위로 싱글톤으로 두고 재사용
- 커넥션은 비용 있으니 빈번하게 만들지 말 것.
- 클러스터 환경에서는 `StatefulRedisClusterConnection` 사용, 노드 이동(resharding)과 명령 라우팅을 고려
- 재접속/재시도 정책, backoff, timeout을 명확히 설정
- 모니터링: command latency, pending futures, connection 수, event loop latency

## Redisson

Redisson은 Redis를 기반으로 하는 고수준 Java 라이브러리로, Redis를 분산 데이터 구조(Distributed Java Objects), 동기화(Distributed Locks), 메시징(Topics), 캐시(RMapCache), 세마포어, 스케줄링 등 다양한 추상(추상화된 분산 primitive)으로 제공한다.
즉, Redis를 분산 Java 라이브러리/플랫폼처럼 손쉽게 쓰게 해준다.

### 기본 원리
- Redis 명령을 사용하되, Redisson이 자주 쓰이는 분산 패턴(lock, map, semaphore 등)을 Redis 데이터 구조 + Lua 스크립트 + 내부 프로토콜로 구현한다.
- 내부적으로 Netty 기반의 커넥터로 Redis와 통신한다(성능 최적화).
- 객체 직렬화(Codec)를 통해 Java 객체를 Redis에 저장/전달
- 분산 락은 보통 `SET NX PX` + watchdog(락 연장) 같은 기법을 사용하거나 Lua 스크립트로 원자성을 보장.
### 필요성
- Redis를 이용해 분산 락, 분산 컬렉션(Map/Set/List), 분산 스케줄러, 세마포어, 원자적 카운터 등 고수준 분산 primitive를 빠르게 도입하고 싶을 때
- 개발자가 분산 동시성 문제(lock, semaphore, reader-writer 등)를 직접 구현하고 검증할 여력이 없을 때
- Java 네이티브 객체와 유사한 API로 빠르게 개발 생산성을 올리고 싶을 때
### 동작 과정
1. `Config`를 만들고 `Redisson.create(config)`로 `RedissonClient` 획득.
2. `RMap`, `RLock`, `RTopic`, `RAtomicLong` 등 분산 객체 가져오기
3. 해당 객체의 메서드를 호출하면 Redisson이 내부적으로 Redis 명령/스크립트/트랜잭션 등을 사용해 연산을 수행
4. 종료 시 `redisson.shutdown()`
### 장단점
#### 장점
- 생산성: 복잡한 분산 동작을 몇 줄로 해결(RLock, RSemaphore, RMapCache, RExecutorService 등)
- 안전한 구현: 재시도/watchdog 등 락 관련 많은 엣지케이스를 라이브러리가 처리
- 풍부한 기능: 분산 Collections, 비동기 API, 스케줄러, ExecutorService 스타일 분산 작업, Live Object, 해시맵 캐시 등
- 클러스터 / Sentinel / Replicated 모드 지원
#### 단점 / 주의점
- 추상화 오버헤드: 편리함 대가로 성능/메모리 오버헤드가 존재. 원시 Redis 명령을 직접 사용하는 것보다 비용이 듦
- 블랙박스성: 내부에서 어떤 Redis 명령/스크립트를 쓰는지 완전히 이해하지 못하면 성능 이슈 원인 파악이 더 어려울 수 있음
- 직렬화 비용: Java 객체 직렬화(CODEC 선택)에 따라 네트워크 바이트 수/성능 영향 큼
- 잠재적 락 오용: 분산 락을 쉽게 제공하므로 잘못 쓰면 전체 서비스 병목 발생
### 실무 팁 / 운영 주의
- 가능한 경우 직렬화 codec을 명확히 지정(JSON, Kryo, FST 등), 기본 직렬화(ex: Java Serialization)는 성능/호환성 측면에서 피하는 편이 낫다.
- RLock 사용 시 leaseTime(auto-unlock)과 watchdog 동작을 이해하고 통제
- Redisson이 내부적으로 Lua 스크립트나 여러 명령을 연속 호출할 수 있으니 명령 모니터링으로 비용 파악
- 고성능 요구에는 Redisson 대신 Lettuce로 핵심 hot path를 직접 구현하는 하이브리드 전략을 고려

## Lettuce vs Redisson

### 선택 기준
- 성능·레이턴시가 최우선이고, Redis 명령을 직접 제어하고 싶다 → Lettuce. (특히 리액티브 스택 사용 시 자연스럽다)
- 분산 락, 분산 컬렉션, 분산 스케줄러 등 고수준 추상화가 필요하고 개발 생산성을 빨리 올리고 싶다 → Redisson.
- 혼합 전략 권장: 핵심 핫패스(예: 조회·카운터)는 Lettuce로 최적화, 분산 락·세마포어·작업 스케줄러 등은 Redisson으로 처리.

### 성능/오버헤드 관점
- Lettuce: 낮은 오버헤드, 직렬화 부담 적음(보통 String/바이트 사용).
- Redisson: 객체 직렬화/추상화 오버헤드 존재 → throughput/latency 영향 가능.

### 기능성 관점
- Lettuce: low-level Redis features 접근에 유리(Streams, Modules 명령 등)
- Redisson: 편리한 분산 프리미티브(RExecutorService, RLiveObject, RMapCache 등)