## 이벤트 기반 아키텍처

분산된 애플리케이션 서비스들이 이벤트를 기반으로 통신하고 서로의 동작을 야기하는 패턴을 이벤트 기반 아키텍처라고 한다.

### 이벤트
- 이벤트는 사이트 방문, 주문, 결제 등 비즈니스 내외부에서 발생한 주목할만한 사건을 의미한다.
- 이미 일어난 일에 대한 기록이기 때문에 이벤트는 변경되지 않는다는 특징이 있다.

### 이벤트의 포맷
- 이벤트는 일반적으로 json, xml 등의 포맷으로 교환된다.
- 이벤트의 내용은 header와 body로 구성되는데, header는 이벤트의 아이디, 타입, 타임스탬프 등 메타데이터에 해당되는 정보를 담고 있고, body는 어떤 일이 발생했는지에 대한 설명을 담고 있다.
- 이벤트는 다양한 형식으로 구성될 수 있는데, 이벤트 포맷의 통일성을 위해 CloudEvents와 같은 이벤트 명세를 사용하기도 한다.
```json
{
    "specversion" : "1.0",
    "type" : "com.github.pull_request.opened",
    "source" : "https://github.com/cloudevents/spec/pull",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "text/xml",
    "data" : "<much wow=\"xml\"/>"
}
```

## MSA와 이벤트 기반 아키텍처

MSA는 하나의 애플리케이션을 여러 서비스로 나누어 운영한다.

### IPC
MSA와 같은 분산 아키텍처에서는 서비스들 간의 협업을 위해 IPC가 중요한 요소가 된다.

IPC는 REST API, gRPC, 공유 메모리, 이벤트 기반 통신 등 다양한 방법으로 구현될 수 있다.

이 중 이벤트 기반 통신의 경우 비동기 방식이기 때문에 동기 방식에 비해 서비스들 간의 결합을 느슨하게 할 수 있다는 장점이 있다.

### 트랜잭션 관리
MSA에서 각 서비스는 별도의 DB를 가지고 있다.

데이터가 여러 DB에 분산되어 있다보니 데이터 간의 일관성을 유지하는 것이 단일 DB에 비해 까다로운 편이다.

마이크로 서비스 아키텍처는 데이터 일관성을 유지하기 위해 일반적으로 사가(Saga) 패턴을 사용하는데, 이 패턴에서 이벤트가 활용된다.

Saga 패턴은 아래와 같이 각 서비스의 로컬 트랜잭션은 데이터베이스를 업데이트한 다음에 메세지 또는 이벤트를 발행하여 다른 서비스의 로컬 트랜잭션 작업을 야기한다.
![[Pasted image 20250507180038.png]]

## Pub/Sub 모델

이벤트 기반 아키텍처는 일반적으로 Event Producer, Broker(Manager), Consumer로 구성되는데, Pub/Sub 모델은 전형적인 이벤트 기반 아키텍처의 형태를 띤다.

### Pub/Sub 모델과 메세지 큐 모델
이벤트 기반 Pub/Sub 모델은 메세지 큐 모델과 유사하지만 몇 가지 차이점을 가지고 있다.

메세지 큐 모델에서 Publisher는 전달하고자 하는 Subscriber에 지정되어 있는 메세지 큐에 메세지를 전송해야 한다.

반면 Pub/Sub 모델에서는 Broker가 이벤트를 관리하므로 Publisher는 어떤 Subscriber로 보내야 하는지 알 필요 없이 이벤트를 발행하면 된다.

메세지 큐 모델에서는 Subscriber가 추가될 경우 새로운 Que 인스턴스를 추가해야 하지만, Pub/Sub 모델에서는 Publisher와 Subscriber가 Decoupling되어 있기 때문에 Publisher를 업데이트하거나 새로운 인스턴스를 추가할 필요가 없다.

따라서 Pub/Sub 모델은 시스템을 유연하게 확장할 수 있다는 이점이 있다.

## 메세지 큐와 이벤트 기반 아키텍처 장점

메시지 큐와 이벤트 드리븐 아키텍처는 다양한 장점을 가지고 있다.
- 시스템의 결합도를 낮출 수 있다.
- 시스템의 확장성을 높일 수 있다. 메시지 큐를 사용하면 메시지를 비동기적으로 처리할 수 있으며, 이벤트 드리븐 아키텍처를 사용하면 시스템의 확장성을 높일 수 있다.
- 시스템의 유연성을 높일 수 있다.
- 시스템의 반응성을 높일 수 있다.

## 메세지 큐와 이벤트 기반 아키텍처 단점

메시지 큐와 이벤트 드리븐 아키텍처는 다양한 장점을 가지고 있지만, 몇 가지 단점도 존재한다.
- 시스템의 복잡도가 증가할 수 있다.
- 디버깅이 어려울 수 있다.
- 메시지 손실이 발생할 수 있다.
- 시스템의 성능이 저하될 수 있다.