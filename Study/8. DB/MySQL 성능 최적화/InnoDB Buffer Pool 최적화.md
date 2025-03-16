
## InnoDB Pool 이해

### 역할과 중요성
테이블 데이터와 인덱스 데이터를 메모리에서 캐싱하고, 변경된 데이터를 디스크에 플러시하는 역할을 한다.
![[KakaoTalk_20250316_202137181.png]]

### Buffer Pool은 Instance로 나눠져있다
- Instance 단위로 Flush 리스트와 LRU 리스트가 나눠져있다.
- Instance 의 수가 많을수록 동시성 수준의 성능이 증가한다.

### Chunk는 Buffer Pool이 resizing 될 때 비용을 최소화하기 위해서 설계됐다

## Buffer Pool 최적화 전략

### **`innodb_buffer_pool_size`** 조절하기
- **`innodb_buffer_pool_size`** 는 OS 메모리의 70~80% 정도로 할당해주는 것이 좋다.
- 메모리 크기가 8GB 미만이라면 OS 메모리의 50%만 쓰는 것을 권장
- 기본 값 128MB

### Buffer Pool Instances 조절하기
- **`innodb_buffer_pool_size`** 사이즈에 맞게 **`innodb_buffer_pool_instance`** 의 수도 조절해서 동시성 성능 향상
- 메모리가 40GB 밑이라면 **`innodb_buffer_pool_instance`** 는  기본 값으로 사용. (기본 값: 8)
- 메모리가 40GB보다 많아진다면 5GB 마다 **`innodb_buffer_pool_instance`** 의 수를 하나씩 증가

### Read-Ahead 트리거링 조절
- Read-Ahead 전략
	- 테이블이나 인덱스의 연속된 데이터 블록을 미리 비동기로 가져오는 최적화 방식
- **`innodb_read_ahead_threshold`** 설정
	- 이 값보다 많은 연속된 페이지를 읽는다면 Read-Ahead 가 활성화 된다. (기본 값: 56)
- **`innodb_random_read_ahead`** 설정
	- Buffer Pool에 이 값보다 많은 연속된 페이지가 있다면 Read-Ahead가 활성화 된다. (기본 값: 비활성화)