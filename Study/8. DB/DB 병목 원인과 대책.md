DB 서버 병목의 경우 각각의 원인은 다음과 같이 분류할 수 있다.
- DB 설계 문제
- DB 사용 애플리케이션 문제
- 서버 리소스 부족

어느 경우에서도 먼저 **실제 실행 시간이 걸리는 쿼리를 찾아내는 것이 가장 중요**하다.

## DB 설계 문제

실행 속도를 떨어뜨리기 쉬운 패턴 3가지를 꼽으라면 다음과 같다.
### 인덱스 문제
인덱스 설정이 잘못되면 RDB는 매우 비효율적으로 실행된다.

MySQL의 경우 SHOW CREATE INDEX로 테이블에 생성된 인덱스를 확인할 수 있다. 인덱스를 생성하는 컬럼의 기준은 다음과 같이 내용이 있지만, 실제 쿼리 실행 계획을 EXPLAIN으로 확인하면서 필요한 인덱스를 추가한다.
- WHERE 절 조건으로 자주 사용되는 컬럼
- JOIN 키가 되는 컬럼
- Cardinality가 높은 컬럼

Cardinality가 낮은 컬럼에 인덱스를 사용하면 추가적인 메모리와 스토리지 리소스 사용이 발생하여 데이터 갱신 시 인덱스 생성에 불필요한 리소스를 사용하게 되고 검색 시 인덱스를 사용하지 않는 검색으로 인해 느려지는 경우도 발생한다.

또 실제 이 컬럼들에 대해 사용 가능한 인덱스를 생성한다고 해도 상태에 따라 해당 인덱스를 활용할 수 없어 효율이 떨어지는 쿼리를 실행할 수도 있다. 이런 경우에는 인덱스 히트 구문을 사용하여 SQL 안에서 사용할 인덱스를 지정할 수 있다.

### 적절한 테이블 분리가 되지 않음
테이블이 올바르게 정규화되어 있는지 아닌지는 실행 시 속도에 영향을 미친다.
예를 들어 5,6개 혹은 그 이상의 테이블을 JOIN해야 하는 상황에서 쿼리 실행은 느려진다.
각 테이블이 비대화 되지 않는 범위 또는 인덱스가 적절하게 사용 가능한 범위에서 테이블을 정규화할 필요가 있다.

### 적절한 스토리지 엔진 미사용
MySQL의 경우지만 MySQL에서는 테이블 별로 스토리지 엔진을 선택할 수 있다.
스토리지 엔진 별로 사용 가능한 기능에는 특징이 있고 스토리지 엔진으로 **MyISAM**을 사용한 경우 테이블 갱신 시에 테이블 행의 락이 아닌 **테이블에 락**이 걸린다. 그래서 **테이블 갱신이 많이 발생하는 경우**에는 **전체 성능 저하**가 발생한다.

## DB 사용 애플리케이션 문제

### 지속적인 연결 미사용
캐시 사용과 같이 DB 접속에서도 지속적인 연결을 사용하지 않으면 DB에 접속할 때 마다 연결해야 하고 부하가 많을 때는 접속 Latency를 무시할 수 없다. 상용 RDBMS 제품도 부하가 많은 상황에서는 지속적인 연결 사용 여부에 따라 테스트 결과가 크게 차이날 수 있다.

### 적절하지 않은 쿼리 사용
RDB에서는 같은 실행 결과를 가진 쿼리도 여러가지 방법으로 실행할 수 있어 쿼리 작성 방법에 따라 내부 실행 계획이 바뀌고 성능에 영향을 줄 수 있다.

특히 문제가 발생하기 쉬운 경우는 다음과 같다.
- JOIN 해야 하는 부분에 **JOIN을 하지 않고 여러 번 쿼리**를 실행
- **같은 쿼리 안에 JOIN 하는 횟수가 너무 많아짐**
- 서브 쿼리를 사용했을 대 **서브 쿼리 실행 결과가 너무 많음**

이러 상황이 발생 했을 때 쿼리 최적화를 해야 한다. O/RM으로 쿼리 빌드를 하고 있을 대 특히 이런 부적절한 SQL이 생성되어 버리는 경우가 있어 주의가 필요하다.

### 갱신 락 발생
**갱신 쿼리 속도가 참조계 쿼리에 비해 느린 경우** 갱신 락이 발생했을 가능성이 있다.

갱신 락이 발생하기 쉬운 경우로 DB에 **같은 레코드를 동시에 여러 사용자가 갱신**하려고 했을 때 락이 발생한다. MyISAM에서 갱신 시에 테이블 전체 락이 걸리는 경우에도 InnoDB와 같이 행에 락이 발생하는 스토리지 엔진에서도 발생한다.

이 락이 발생하게 되면 갱신 트랜잭션은 동시에 하나 밖에 처리할 수 없게 되고 해당 처리에 대한 Throughput은 현저히 저하된다.

특히 counter-increment(증가) 처리나 decrement(감소) 처리가 발생하기 쉽고 다음과 같은 설계를 했을 때 주의가 필요하다.
- 경품 당첨자 수 counter를 DB increment 처리로 설계하는 것
- 기사에 대해 '좋아요' counter를 DB increment 처리로 설계하는 것
- 상품 재고 확보를 DB increment 처리로 설계하는 것

이런 문제를 피하고자 **increment 처리 대신 신규 레코드를 생성**하고 counter는 별도 **KVS와 같은 빠른 시스템을 사용**하는 방법을 고려해야 함

### 적절한 캐시 미사용
캐시되어야 할 데이터가 캐시되지 않아 DB에 불필요한 쿼리가 몇 번이고 실행되는 경우가 있다. 특히 발생하기 쉬운 상황은 **데이터가 존재하지 않은 상태를 캐시하지 않는 것**이다.

예를 들어 상품 DB 조회가 느린 시스템으로 상품 검색 결과를 캐시하는 시스템을 구축했다고 하자. 여기서 '100'이라는 상품이 과거에는 있었지만, 현재는 판매하지 않는다. 이 ID=100이라는 데이터는 존재하지 않아 존재하지 않는다는 정보를 캐시하지 않는다면 ID=100 상품의 조회가 발생하면 데이터를 가지고 있는 느린 시스템에 저장된 후 조회해야 한다.

이것을 방지하기 위해 '상품 ID=100은 존재하지 않는다.'라는 정보를 캐시에서 판단할 수 있어야 한다. 이 캐시를 상품 ID 별로 가지고 있어도 되지만, 조회가 필요한 ID에 이상한 숫자를 넣었을 때는 처리가 어려운 경우가 있으니 상품 수가 적을 때에는 모든 상품 정보를 한번은 캐시해 두는 방법도 있다.

### 부하테스트 시나리오 준비 부족
부하테스트 시나리오에 문제가 있다면 특정 레코드에 참고와 갱신이 집중되어 버린다. 따라서 이런 시나리오를 수정하고 서비스 환경과 흡사하게 대상 레코드가 분산될 수 도 있도록 한다.

## 서버 리소스 부족

### 참조 쿼리가 무겁다.
갱신 쿼리에는 문제가 없지만, 참고 쿼리만 무거운 경우에는 참고 쿼리를 참고 **전용 슬레이브 서버를 이용**하면 스케일 아웃 할 수 있다.

또 쿼리 종류에 따라서 **Read-through 캐시**를 사용하여 **쿼리 실행 횟수를 줄여 전체 throughput 향상을 기대**할 수 있다.

### CPU 리소스 부족
CPU 리소스가 부족할 때 DB 스케일 업 또는 스케일 아웃을 테스트 한다.

그러나 스케일 아웃에 따라 참고 쿼리에 의한 CPU 리소스 부족은 대응할 수 있지만 갱신 쿼리를 스케일 아웃으로 해결하는 것은 어렵다. 일반적으로 갱신 처리 확장은 일정 시간 다운 타임을 가지고 가면서 스케일 업을 해야 한다.

또한 **스케일 업을 해도 DB Throughput이 변화가 없는 경우도 많다**. 그래서 RDB를 사용한 시스템에서 스케일 업/스케일 아웃을 하면서 확장하지만, **최종적인 병목 부분은 DB의 갱신 성능이 되는 경우가 많다**.

### 메모리 리소스 부족
RDS for MySQL에서 DB안에 레코드가 늘어나면 CloudWatch 등에서 모니터링 할 수 있는 Freeable Memory 수치는 낮아지지만, 이 Freeable memory는 일정 값 이하로 내려가지 않는다. 이것은 Freeable Memory가 가용할 때 테이블 내용과 인덱스 데이터를 적극적으로 메모리에 올리기 때문이다.

메모리 리소스가 부족하고 인덱스를 메모리에 올리지 못하는 경우
지금까지 **인덱스를 사용하여 빠르게 처리했던 부분이 급격하게 느려지게 된다**. 이런 상황에서는 **더 많은 메모리를 사용할 수 있는 인스턴스 타입으로 변경**해야 한다.

### 스토리지 I/O 부족
RDB에서 사용중인 스토리지 I/O 성능이 병목이 될 수 있고 전체 Throughput을 떨어드리는 원인이 된다.

AWS의 RDS에서는 스토리지 I/O 성능에도 CPU 크레딧과 같은 버스트 기능이 있어 일정 기간 동안 기준 I/O 성능 보다 높은 성능을 사용할 수 있다. 그러나 버스트가 끝나면 자동으로 기준 I/O 성능 밖에 사용할 수 없어 큰 병목이 발생하는 경우가 있다.

AWS에서는 **PIOPS(Provisioned IOPS)** 를 사용하면 인스턴스 타입과는 별도로 스토리지 I/O 성능만을 구입하여 사용할 수 있어 쓰기 작업이 많은 시스템에서는 PIOPS 사용을 검토하길 바란다.

또 AWS에서 RDS for MySQL을 사용하는 경우 **Aurora**를 사용하면 이러한 문제들을 해결 할 수 도 있다.

## DB 서버 병목 원인과 대책 참고표

|**원인 구분**|**원인 상세**|**주요 증상**|**대책**|
|---|---|---|---|
|DB 설계 문제|인덱스 문제|쿼리 실행에 대한 응답 Latency가 커짐|Slow query log를 확인하고 느린 쿼리에 인덱스 적용 및 수정|
||적절한 테이블 분리|쿼리 실행에 대한 응답 Latency가 커짐|DB 설계 전체를 다시 검토|
||적절한 스토리지 엔진 미사용|테이블 락 발생|스토리지 엔진 선택|
|DB 사용 애플리케이션 문제|지속적인 연결 미사용|DB에 접속할 때 Latency가 커짐|지속적인 연결 사용 검토|
||적절하지 않은 쿼리 사용|쿼리 실행 Latency가 커짐|- 복잡한 쿼리 분리- 적절한 JOIN 사용- O/RM 생성 쿼리에 주의|
||갱신 락이 발생|갱신 쿼리가 무거움|- 갱신이 집중되는 레코드가 있을 경우 DB 이용 정책을 검토- KVS 사용 검토|
||부하테스트 시나리오 준비 부족으로 갱신이 특정 레코드에 집중|- 갱신 쿼리가 무거움- 원래 발생하지 않는 로그 발생|부하 테스트 시나리오에서 특정 사용자만 사용하는 시나리오가 잘못된 경우가 있음. 시나리오 재검토 필요|
|서버 리소스 부족|참고 쿼리가 무거움|DB 응답 Latency가 커짐|-(가능한 범위에서) 참고 전용 슬레이브 서버 이용- 캐시 이용 검토|
||CPU 리소스 부족|- DB 서버 CPU 사용률이 높아짐- DB 서버 메모리 부족 발생|- 스케일 업- (가능한 범위에서) 스케일 아웃|
||메모리 리소스 부족|- Freeable Memory 부족- 데이터양이 적을 때는 빠른 응답을 주던 참고, 갱신 쿼리가 급격하게 느려짐|스케일 업|
||스토리지 I/O 부족|다른 리소스에는 여유가 있지만 스토리지 I/O가 일정 Throughput에서 더 이상 증가하지 않음|- PIOPS 사용 검토- 스케일업- (가능한 범위에서) 스케일 아웃- Aurora 사용 검토|