
## Buffer Pool Flushing 메커니즘

### Flushing
Buffer Pool에 쌓인 더테 페이지(변경된 데이터)들을 디스크에 주기적으로 플러시하는 과정

### Page Cleaner Thread
Page Cleaner Thread가 더티 페이지들을 플러쉬하는 역할을 담당
- **`innodb_page_cleaners`** 값을 통해 조정할 수 있다.
- 기본 값은 4이다.

## 최적화 전략

### Page Cleaner Thread 조정
데이터 변경 작업이 많은 환경에서 Thread 수를 늘려 플러싱 성능 향상
- 설정할 수 있는 최대값은 **`innodb_buffer_pool_instance`** 수와 같다.

### 플러싱 인접 페이지
**`innodb_flush_neighbors`** 값을 조절하여 플러싱할 때 이웃 페이지들을 함께 플러쉬하는 방식 설정 (특히 HDD 스토리지에서 유용하다)
- **`innodb_flush_neighbors`** 값이 0이면 이웃 페이지들과 같이 플러쉬 되지 않는다.
- **`innodb_flush_neighbors`** 값이 1이면 연속된 더티 페이지들은 같이 디스크에 플러쉬 된다.
- **`innodb_flush_neighbors`** 값이 2이면 같은 Extent에 있는 더티 페이지들이 같이 디스크에 플러쉬 된다.

### LRU 리스트 스캔 최소화
**`innodb_lru_scan_depth`** 값을 조절하여 더티 페이지 스캔을 자주하지 않도록 하는 방식 설정 (특히 Write Intensive 한 어플리케이션에서 유의미 할 수 있다)
- **`innodb_lru_scan_depth`** 로 명시된 값만큼 1초마다 LRU 리스트를 스캔한다.