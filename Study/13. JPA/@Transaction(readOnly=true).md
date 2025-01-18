
## 성능 이점

- 이는 JPA의 영속성 컨텍스트가 수행하는 **변경 감지(Dirty Checking)** 와 관련이 있다.
    - 영속성 컨텍스트는 Entity 조회 시 초기 상태에 대한 Snapshot을 저장한다.
    - 트랜잭션이 Commit 될 때, 초기 상태의 정보를 가지는 **Snapshot과 Entity의 상태를 비교하여 변경된 내용에 대해 update query를 생성해 쓰기 지연 저장소에 저장**한다.
    - 그 후, **일괄적으로 쓰기 지연 저장소에 저장되어 있는 SQL query를 flush하고 데이터베이스의 트랜잭션을 Commit** 함으로써 우리가 update와 같은 메서드를 사용하지 않고도 Entity의 수정이 이루어진다.
    - 이를 변경 감지(Dirty Checking)라고 한다.
- readOnly=true를 설정하게 되면 스프링 프레임워크는 JPA의 **세션 플러시 모드를 MANUAL로 설정**한다.

> [!info]
> MANUAL모드는 트랜잭션 내에서 사용자가 수동으로 flush를 호출하지 않으면 flush가 자동으로 수행되지 않는 모드이다.
    
- 즉, 트랜잭션 내에서 강제로 flush()를 호출하지 않는 한, 수정 내역에 대해 DB에 적용되지 않는다.
- 이로 인해 트랜잭션 Commit 시 영속성 컨텍스트가 **자동으로 flush 되지 않으므로 조회용으로 가져온 Entity의 예상치 못한 수정을 방지**할 수 있다.
- 또한, readOnly=true를 설정하게 되면 JPA는 해당 트랜잭션 내에서 조회하는 Entity는 조회용임을 인식하고 **변경 감지를 위한 Snapshot을 따로 보관하지 않으므로 메모리가 절약되는 성능 상 이점** 역시 존재한다.
    

## 가독성

- readOnly=true를 붙임으로써 직관적으로 해당 메서드가 조회용 메서드 임을 알기 쉬워진다.
```java
@Transactional
public Member getMember(Long memberId) {
    Optional<Member> member = memberRepository.findById(memberId);
    ...
    return member;
}

@Transactional(readOnly=true)
public Member getMember(Long memberId) {
    Optional<Member> member = memberRepository.findById(memberId);
    ...
    return member;
}
```

## Replication 부하 분산

- 기본적으로 간단한 프로젝트에서는 데이터베이스를 하나만 둔다.
- 하지만 실제 운용되는 서비스에서는 데이터베이스의 장애를 빠르게 복구하고, 트래픽을 분산하기 위해 **실시간 복제본 데이터베이스**를 운용하는 **레플리케이션(Replication) 방식**을 사용할 수 있다.
![[Untitled 1.png]]
- 레플리에키션은 Master-Slave 구조로 복제본 DB를 함께 운용함으로써, **Master DB의 장애 발생 시 Slave DB를 Master DB로 승격시켜 장애를 빠르게 복구**할 수 있으며, **조회 작업은 Slave DB**에서 수행하고 **수정 작업은 Master DB**에서 수행함으로써 트래픽을 분산할 수 있다는 장점이 있다.
- 이러한 데이터베이스 구조를 가져갈 때, **readOnly=true가 설정되어 있는 메서드의 경우 Slave DB에서 데이터를 가져오도록 동작**한다.
- 이를 통해 레플리케이션의 목적에 맞게 트래픽 분산을 온전하게 적용할 수 있다는 추가적인 이점이 존재한다.

# @Transctional 어노테이션을 안붙이면?

그럼 조회용 메서드에는 @Transactional 어노테이션을 안붙이면 되지 않을까 하는 의문이 든다.
    
조회용 메서드에 대해 @transactional 어노테이션 유무의 차이는 **OSIV(Open Session In View)가 꺼져있을 때** 알 수 있다.
    
OSIV는 **영속성 컨텍스트를 View Layer 까지 유지하는 속성**으로, 클라이언트의 요청 시점부터 영속성 컨텍스트를 생성하여 Filter / Interceptor - Controller 에서 부터 영속성 컨텍스트가 생성되어 유지됨으로써 **View Layer에서도 Entity의 Lazy Loading이 가능**하도록 한다.
    
기본적으로 별도의 설정을 하지 않는다면 OSIV는 **true**로 설정되어 있어 @Transactional 어노테이션 유무의 차이를 알 수 없다.
- 실제로, OSIV를 켠 상태에서 @Transactional 어노테이션 유무와 상관없이 다음 Lazy Loading을 수행하는 코드의 동작은 Exception 없이 정상적으로 동작한다.

하지만, OSIV를 false로 설정한다면 영속성 컨텍스트는 **트랜잭션 범위를 벗어나는 순간 Entity는 영속성 컨텍스트의 관리를 받지 않는 준영속 상태**가 되어버린다.    

영속성 컨텍스트의 관리를 받지 않는 준영속 상태가 된다는 말은 곧 **Lazy Loading의 동작이 불가능**하다는 의미이다.
OSIV를 false로 설정하고 @transctional 어노테이션을 제거했을 때, **LazyInitializationException**이 발생함을 확인할 수 있다.
![[Untitled 1.png]]
이는 OSIV가 꺼져있는 상태이므로, @Transactional 어노테이션이 붙어있지 않은 상태에서 member를 조회하는 순간 트랜잭션 범위에 존재하지 않으므로 즉시 준영속 상태에 들어가 Lazy Loading의 동작이 불가능하게 된 것이다.

이렇듯, OSIV가 꺼져있는 상태에서는 @Transactional 어노테이션이 없을 때에 Lazy Loading의 동작을 수행할 수 없다는 문제점이 있으므로 조회용 메서드에 대해서도 @Transactional 어노테이션을 붙여주어야 하는 것이다.

OSIV는 기본적으로 true이지만, OSIV 전략은 클라이언트 요청시점부터 API 응답이 끝날 때까지 영속성 컨텍스트와 데이터베이스 커넥션을 유지하므로 **실시간 트래픽이 중요한 애플리케이션 서비스에서 커넥션 부족으로 이어질 수 있다는 큰 단점**이 있다.
그러므로 무조건적으로 OSIV를 적용하는 것이 좋은 방안은 아니기 때문에, OSIV 전략을 사용하지 않는 상황에 대비하여 기본적으로 조회용 메서드에 대해서도 @Transactional 어노테이션을 붙일 수 있도록 하자.
