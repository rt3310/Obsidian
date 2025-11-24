## STOMP

`WebSocket` 프로토콜은 두 가지 유형의 메시지를 정의하고 있지만, 그 메시지의 내용까지는 정의하고 있지 않다.

STOMP는 `WebSocket` 위에서 동작하는 프로토콜로써, 클라이언트와 서버가 전송할 메시지 유형, 형식, 내용들을 정의하는 매커니즘으로, `TCP` 또는 `WebSocket` 같은 양방향 네트워크 프로토콜 기반으로 동작한다.

이름에서도 알 수 있듯이 텍스트 지향 프로토콜이지만 `Message Payload`에는 `Text` 또는 `Binary` 데이터를 포함할 수도 있다.

STOMP는 HTTP 위에서 동작하는 `Frame` 기반의 프로토콜이며, `Frame`은 아래와 같은 형식을 가지고 있다.
```
COMMAND
header1:value1
header2:value2

Body^@
```
[STOMP Protocol Specification, Version 1.2](https://stomp.github.io/stomp-specification-1.2.html#Abstract)

클라이언트는 `Message`를 전송하기 위해 `SEND`, `SUBSCRIBE` `COMMAND`를 사용할 수 있다.
또한, `SEND`, `SUBSCRIBE` `COMMAND` 요청 `Frame`에는 **메시지가 무엇이고 누가 받아서 처리할 지에 대한 `Header` 정보를 함께 포함**한다.

위와 같은 과정을 통해서, Publish-Subscribe(Pub/Sub) 매커니즘을 제공한다. 즉, Broker를 통해서 다른 사용자들에게 메시지를 보내거나 서버가 특정 작업을 수행하도록 메시지를 보낼 수 있게 되는 것이다.
만약 스프링에서 지원하는 STOMP를 사용하게 된다면, 스프링 WebSocket 애플리케이션은 STOMP Broker로 동작한다.

## 스프링에서의 STOMP

스프링에서 지원하는 STOMP는 다양한 기능을 제공한다.

- 메시지를 **`@Controller`의 메시지 핸들링하는 메서드로 라우팅**
- **Simple In-Memory Broker**를 이용해서 **Subscribe중인 다른 클라이언트들에게 메시지를 브로드캐스팅**

### Simple In-Memory Broker 
Simple In-Memory Broker는 **클라이언트의 Subscribe 정보를 자체적으로 메모리에 유지**한다. 뿐만 아니라,  RabbitMQ, ActiveMQ 같은 외부 Messaging System을 STOMP Broker로 사용할 수 있도록 지원하고 있다. 이 경우에 스프링은 외부 STOMP Broker와 TCP 커넥션을 유지하는데, 외부 STOMP Broker는 서버-클라이언트 사이의 매개체로 동작한다.

구체적으로 스프링은 메시지를 외부 브로커에 전달하고, 브로커는 WebSocket으로 연결된 클라이언트에게 메시지를 전달하는 구조이다.

위와 같은 구조 덕분에, 스프링 웹 애플리케이션은 'HTTP 기반의 보안 설정'과 '공통된 검증' 등을 적용할 수 있게 된다.

만약 클라이언트가 특정 경로에 대해서 아래와 같이 Subscribe한다면, 서버는 원할 때마다 클라이언트에게 메시지를 전송할 수 있다.
```
SUBSCRIBE
id:sub-1
destination:/topic/something.*

^@
```


```
SEND
destination:/queue/something
content-type:application/json
content-length:38

{"key1":"value1","key2":"value2", 38}^@
```

STOMP 스펙에서는 의도적으로 `destination` 정보를 불분명하게 정의하였는데, 이는 STOMP 구현체에서 문자열 구문에 따라 직접 의미를 부여하도록 하기 위해서이다. 따라서, `destination` 정보는 STOMP 서버 구현체마다 달라질 수 있기 때문에 각 구현체의 스펙을 살펴봐야 한다.

그러나, 일반적으로 **`/topic` 문자열로 시작하는 구문은 일대다(one-to-many)** 관계의 publish-subscribe를 의미하고, **`/queue` 문자열로 시작하는 구문은 일대일(one-to-one)** 관계의 메시지 교환을 의미한다.


STOMP 서버는 `MESSAGE` COMMAND를 사용해서 모든 Subscriber들에게 메시지를 브로드캐스트할 수 있는데, STOMP Broker는 반드시 애플리케이션이 전달한 메시지를 구독한 클라이언트에게 전달해야 하며, 서버 메시지의 `subscription` 헤더는 클라이언트가 SUBSCRIBE한 `id` 헤더와 일치해야만 한다.
```
MESSAGE
message-id:d4c0d7f6-1
subscription:sub-1
destination:/topic/something

{"key1":"value1","key2":"value2"}^@
```

## STOMP 장점

Spring Framework 및 Spring Security는 STOMP 프로토콜을 사용하여, WebSockets만 이용할 때 보다 더 풍부한 프로그래밍 모델을 제공한다.
- 메시징 프로토콜을 만들고, 메시지 형식을 커스터마이징할 필요가 없다.
- RabbitMQ, ActiveMQ 같은 Message Broker를 이용해서, subscription을 관리하고 메시지를 브로드캐스팅할 수 있다.
- WebSocket 기반으로 각 커넥션마다 WebSocketHandler를 구현하는 것보다, `@Controller`된 객체를 이용해서 조직적으로 관리할 수 있다.
	- 즉 메시지들은 STOMP의 `destination` 헤더를 기반으로, `@Controller` 객체의 `@MethodMapping` 메서드로 라우팅된다.
- STOMP의 `destination` 및 `Message Type`을 기반으로 메시지를 보호하기 위해, Spring Security를 사용할 수 있다.

### Message Flow
일단 STOMP Endpoint를 노출하면, 스프링 애플리케이션은 연결된 클라이언트에 대한 STOMP Broker가 된다.

### 구성 요소
spring-message 모듈은 스프링 프레임워크의 통합된 메시징 애플리케이션을 위한 근본적인 지원을 한다.

다음 목록에서는 몇 가지 사용 가능한 메시징 추상화에 대해 설명한다.
- [`Message`](https://docs.spring.io/spring-framework/docs/5.2.8.RELEASE/javadoc-api/org/springframework/messaging/Message.html): **`headers`와 `payload`를 포함하는 메시지의 `representation`**
- [`MessageHandler`](https://docs.spring.io/spring-framework/docs/5.2.8.RELEASE/javadoc-api/org/springframework/messaging/MessageHandler.html): **`Message` 처리에 대한 계약**
- [`MessageChannel`](https://docs.spring.io/spring-framework/docs/5.2.8.RELEASE/javadoc-api/org/springframework/messaging/MessageChannel.html): Producers과 Consumers의 느슨한 연결을 가능하게 하는 **메시지 전송에 대한 계약**
- [`SubscribableChannel`](https://docs.spring.io/spring-framework/docs/5.2.8.RELEASE/javadoc-api/org/springframework/messaging/SubscribableChannel.html): `MessageHandler` 구독자(Subscribers)를 위한 MessageChannel
	- Subscribers를 관리하고, 해당 채널에 전송된 메시지를 처리할 Subscribers를 호출한다.
- [`ExecutorSubscribableChannel`](https://docs.spring.io/spring-framework/docs/5.2.8.RELEASE/javadoc-api/org/springframework/messaging/support/ExecutorSubscribableChannel.html): `Executor`를 사용해서 메시지를 전달하는 `SubscribableChannel`
    - `ExecutorSubscribableChannel`은 각 구독자(`Subscribers`)에게 메시지를 보내는 `SubscribableChannel`이다.
    
Java 기반의 설정(`@EnableWebSocketMessageBroker`)과 XML 네임스페이스 기반의 설정(`websocket:message-broker`)은 모두 앞선 위의 구성 요소를 사용해서 `message workflow`를 구성한다.

아래의 그림은 내장 메시지 브로커를 사용한 경우의 컴포넌트 구성을 보여준다.
![[Pasted image 20250507193532.png]]
[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-flow](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-flow)

- `clientInboundChannel`은 `WebSocket` 클라이언트로 부터 받은 메시지를 전달한다.
- `clientOutboundChannel`은 `WebSocket` 클라이언트에게 메시지를 전달한다.
- `brokerChannel`은 서버의 애플리케이션 코드 내에서 브로커에게 메시지를 전달한다.

다음 그림은 외부 브로커를 사용해서 `subscriptions`과 `broadcasting` 메시지를 관리하도록 설정한 구성 요소를 보여준다.
![[Pasted image 20250507193542.png]]
[https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-flow](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-flow)

위 두 구성 방식의 주요한 차이점은 **`Broker Relay`**를 사용 여부이다.

**`Broker Relay`의 역할은 다음과 같다.**

- TCP 기반으로 외부 `STOMP Broker`에게 메시지를 전달
- 브로커로부터 받은 메시지를 구독한 클라이언트에게 전달
### 동작 흐름

이제 위 그림에 대한 전체적인 흐름을 살펴보면 다음과 같다.

1. WebSocket 커넥션으로 부터 메시지를 전달받는다.
2. STOMP Frame으로 디코드한다.
3. 스프링에서 제공하는 `Message` Representation으로 변환한다.
4. 추가 처리를 위해, `clientInboundChannel`로 전송한다.
    1. STOMP Message의 Destination 헤더가 `/app`으로 시작한다면, `@MessageMapping` 정보와 매핑된 메서드를 호출한다.
    2. 반면에, Destination 헤더가 `/topic` 또는 `/queue`로 시작한다면, 메시지 브로커로 바로(직접) 라우팅된다.

### Message 처리 과정

`@Controller` 컨트롤러는 클라이언트로 부터 받은 `STOMP Mesaage`를 다룰 수 있을 뿐만 아니라, `brokerChannel`을 통해서 `메시지 브로커`에게 메시지를 보낼 수도 있다.

이후, `메시지 브로커`는 매칭된 구독자들(`subscribers`)에게 `clientOutboundChannel`을 통해서 메시지를 브로드캐스팅한다.

또한, 동일한 컨트롤러의 HTTP 요청에 대한 응답 처리 과정에서 같은 작업을 수행할 수 있다.

예를 들어, 클라이언트가 `HTTP POST` 요청을 보낸다고 가정해보자.

그러면, `@PostMapping` 메서드는 `메시지 브로커`에게 메시지를 보내어 구독자들(`subscribers`)에게 브로드캐스팅할 수도 있다.

다음 예제를 통해서, 메시지 처리 과정을 코드로 살펴보자.