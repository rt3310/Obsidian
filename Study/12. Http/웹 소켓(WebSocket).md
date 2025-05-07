## HTTP vs WebSocket

WebSocket이 HTTP 요청을 시작되는 호환성을 가지고 있지만, 분명하게 두 프로토콜은 다른 방식으로 동작한다.

HTTP는 여러 URL을 기반으로 서버 애플리케이션과 Request/Response 형식으로 상호 작용한다.

WebSocket은 반대로 **오직 초기의 커넥션 수립을 위한 하나의 URL만 있고, 모든 애플리케이션 메시지는 동일한 TCP 커넥션에서 전달된다**.

즉, WebSocket은 HTTP 프로토콜과 다른 `asynchronous`, `event-driven`, `messaging` 아키텍쳐 모델이다.

또한, HTTP 경우에는 서버가 URI, Method, Headers 정보로 적절한 핸들러로 라우팅해 처리할 수 있다.

하지만 WebSocket은 **HTTP 와 다르게 메시지 내용에 의미를 두지 않기 때문에, 클라이언트-서버 간에 임의로 메시지에 의미를 부여하지 않으면 처리할 방법이 마땅히 없다**.

이러한 문제를 STOMP 메시징 프로토콜을 통해서 해결할 수 있는데, 상위 프로토콜이 규정한 협약을 토대로 메시지를 처리할 수 있다.

## 언제 WebSocket을 사용할까

![[Pasted image 20250507182401.png]]

### Traditional Polling
고전적인 Polling 방식은 **새로운 정보가 있는지 확인하기 위해 주기적으로 HTTP 요청**을 보낸다. 이러한 방식은 지속적으로 요청을 보내기 때문에, 매번 커넥션을 생성하기 위한 Handshake 비용이 많아지며 서버에 부담을 주게 된다.

### Long Polling
Long Polling은 Traditional Polling을 개선한 방식이다.

클라이언트는 서버에 요청을 보내고, **서버는 변경 사항이 있는 경우에만 응답하여 커넥션을 종료**한다. 그리고 클라이언트는 바로 다시 서버에 요청을 보내어 변경 사항이 있을 때까지 대기하게 된다.

커넥션은 무한히 대기할 수 없으므로, 브라우저는 약 5분 정도 대기하며 중간 프록시에 따라 더 짧게 커넥션이 종료될 수도 있다.

만약 변경 사항이 불규칙적인 간격으로 일어나는 경우 효율적이나, 변경 사항의 빈도가 잦다면 기존 Traditional Polling과 차이가 없으므로 서버의 부담이 증가하게 된다.

### HTTP Streaming
HTTP Streaming은 Long Polling 과 동일하게 HTTP 요청을 보내지만, 변경 사항을 클라이언트에 응답한 이후에도 커넥션을 종료하지 않고 유지한다. 따라서 매번 새로운 커넥션을 맺고 끊는 것이 아니라 **하나의 커넥션을 유지**하며, 변경 사항을 응답 메시지로 전달한다.

HTTP Streaming은 Long Polling 방식에 비해서 **서버의 부담을 줄일 수 있지만, 여러 건의 변경 사항이 일어난 경우 동시 처리가 어려워진다**. 왜냐하면, **서버가 현재 업데이트된 데이터를 모두 전달해야만, 클라이언트에서 다음 업데이트된 데이터의 시작 위치를 알 수 있기 때문**이다.

뿐만 아니라, HTTP Streaming 방식은 서버가 클라이언트에게 전달하는 메시지에 대한 실시간성을 어느 정도 보장하지만, 클라이언트가 서버에게 보내는 요청은 여전히 새로운 커넥션을 생성해야 한다.

이러한 동시성과 서버 부담이라는 Trade Off 사항에서, HTTP Streaming 보다 Long Polling 방식을 많이 사용한다고 한다.

### WebSocket
위와 같은 HTTP Long Polling, Streaming 방식이 가지고 있는 문제를 해결하고, 서버-클라이언트 간에 양방향 통신이 가능하도록 WebSocket 이라는 기술이 만들어지게 됐다.

WebSocket은 서비스를 동적으로 만들어 주지만, AJAX, HTTP Streaming, HTTP Long Polling 기술이 보다 효과적인 경우도 있다. 예를 들어, 변경 사항의 빈도가 자주 일어나지 않고 데이터의 크기가 작은 경우에는 AJAX, HTTP Streaming, HTTP Long Polling 기술이 효과적일 수 있다.

즉, 실시간성을 보장해야 하고 변경 사항의 빈도가 크다면 WebSocket은 좋은 해결책이 될 수 있다.

### WebSocket Session 동시성
`WebSocketHandler`를 사용하는 경우, 표준 WebSocket session(JSR-356)은 동시 전송을 지원하지 않는다.
따라서 STOMP 메시징 프로토콜을 이용해서 메시지 전송을 동기화하거나, `WebSocketSession`을 `ConcurrentWebSocketSessionDecorator`으로 Wrapping해야 한다.

`ConcurrentWebSocketSessionDecorator`은 오직 하나의 스레드만 메시지를 전송하도록 보장해주기 때문이다.

### WebSocket Handshake
각 `WebSocketHandler`마다 HandShake 전(before)/후(after)로 필요한 작업이 있다면, `HandshakeInterceptor` 인터페이스를 구현해서 등록하면 된다.

이를 통해서, HandShake를 막거나 `WebSocketSession`의 속성을 사용할 수 있다.

## SockJS

지금까지는 클라이언트-서버 간에 WebSocket 연결과 메시지 주고 받는 방법에 대해 살펴보았다. 그런데, 클라이언트-서버 WebSocket 통신이 순탄하게만 진행될 수 있을까?

아니다. 그럼, 발생할 수 있는 예외 상황은 어떤 것이 있을지 살펴보자.
```
우선, 모든 클라이언트의 브라우저에서 WebSocket을 지원한다는 보장이 없다.

두 번째로, 클라이언트/서버 중간에 위치한 프록시가 Upgrade 헤더를 해석하지 못해 서버에 전달하지 못할 수 있다. 

마지막으로,  클라이언트/서버 중간에 위치한 프록시가 유휴 상태에서 도중에 커넥션 종료시킬 수도 있다.
```

이러한 문제는 **`WebSocket Emulation`** 을 통해서 해결이 가능하다.

### WebSocket Emulation
우선 `WebSocket`을 첫 번째로 시도하고 `WebSocket` 연결이 실패한 경우에는 HTTP-Streaming, HTTP Long Polling 같은 HTTP 기반의 다른 기술로 전환해 다시 연결을 시도하는 것을 말한다.

즉 **`WebSocket Emulation`** 을 통해서, 위와 같이 `WebSocket` 연결을 할 수 없는 경우에는 다른 HTTP 기반의 기술을 시도하는 방법이다. 이러한, **`WebSocket Emulation`** 을 지원하는 것이 바로 [`SockJS`](https://github.com/sockjs/sockjs-protocol) 프로토콜이다.

Spring Framework는 Servlet 스택 위에서 서버/클라이언트 용도의 SockJS 프로토콜을 모두 지원하고 있다.

즉 `SockJS`의 목표는 "**애플리케이션이 우선적으로 `WebSocket API`를 사용하도록 하지만, 사용할 수 없는 경우에는 런타임 시점에 코드 변경없이 `WebSocket` 이외의 대안으로 대체"** 하도록 하는 것이다.

### 특징
우선 `SockJS`는 브라우저에서 사용하도록 설계가 되었기 때문에, 다양한 브라우저와 버전을 지원하고 있다.
자세한 브라우저 지원 범위는 아래 링크를 참고하길 바란다.
- [sockjs/sockjs-client](https://github.com/sockjs/sockjs-client#supported-transports-by-browser-html-served-from-http-or-https)

또한 `SockJS`는 `WebSocket`, `HTTP Streaming`, `HTTP Long Polling` 등의 크게 세 가지 전송 방법(`Transports`)을 지원하고 있는데, 이외에도 아래와 같이 다양한 방식을 제공하고 있다.
![[Pasted image 20250507185447.png]]

`SockJS`가 지원하는 자세한 전송 방법(`Transports`) 리스트는 아래 링크에서 참고 바란다.
- [sockjs/sockjs-client](https://github.com/sockjs/sockjs-client#supported-transports-by-browser-html-served-from-http-or-https)

### WebSocket Emulation 과정
`SockJS`는 서버로 부터 기본 정보를 획득하기 위해서 `GET /info` 요청을 보내며 시작한다.

클라이언트가 서버에게 `GET /info` 요청을 보내므로써, 서버가 `WebSocket`을 지원하는 지와 전송 과정에서 `Cookies` 지원이 필요한 지 여부, `CORS` 위한 Origin 정보 등의 정보를 응답으로 전달받는다.
![[Pasted image 20250507185944.png]]

이후, 서버가 응답한 메시지를 토대로 앞으로 통신에 사용할 프로토콜을 아래와 같은 방식으로 결정하고 요청을 보낸다.

1. `WebSocket` 사용 가능하다면, `WebSocket` 사용
2. `WebSocket` 사용 불가능하다면,
    1. `Options`의 `Transports` 항목에 `HTTP streaming` 설정이 존재한다면, `HTTP streaming` 사용
    2. `Options`의 `Transports` 항목에 `HTTP streaming` 설정이 없고 `HTTP Long Polling` 존재한다면, `HTTP Long Polling` 사용
```jsx
const sock = new SockJS('http://localhost:8080/test', null, {transports: ["websocket", "xhr-streaming", "xhr-polling"]});
```

### Transports Request URL 형식
모든 `Transports` 요청의 URL 형식은 아래와 같다.
```
http://host:port/myApplication/myEndpoint/{server-id}/{session-id}/{transport}
```
각각의 의미를 하나씩 살펴보자.
- `server-id`는 클러스터 환경에서 요청을 라우팅하는데 유용하게 사용된다.
- `session-id`는 `SockJS` 세션에 속하는 HTTP 요청을 연관시킨다.
- `transport`는 전송 타입을 가리킨다. (ex, `websocket`, `xhr-streaming`, `xhr-polling`)

### Transports Type
`websocket` 타입의 전송 방식은 `WebSocket HandShake`를 하기 위해서 오직 하나의 HTTP 요청만 필요하고, 이후 모든 메시지는 해당 소켓에서 교환된다.
![[Pasted image 20250507190325.png]]