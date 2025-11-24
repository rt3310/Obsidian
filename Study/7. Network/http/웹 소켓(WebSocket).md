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
`xhr-streaming` 타입의 경우에는 장기 실행 요청을 유지하여 서버에서 클라이언트로 전달하기 위한 메시지를 응답으로 전달받는다.

이후, 클라이언트에서 서버로 새로운 요청을 보내야 할 경우에는 기존의 커넥션을 종료하고 새로운 `HTTP POST` 요청을 보내어 커넥션을 유지한다.
- [동작 화면 보기](https://drive.google.com/file/d/1uFFMw4EEl0BtJIV7QTxiScypseLgrsXJ/view?usp=sharing)

`xhr-polling` 타입 경우에는 서버에서 클라이언트로 응답 메시지가 전달이 될 때마다 기존의 커넥션을 종료하고 새로운 요청을 보내어 커넥션을 생성한다.
- [동작 화면 보기](https://drive.google.com/file/d/1MiLzu-x2xW0uu2b7zsBw5DVvGaezwt-T/view?usp=sharing)

### 메시지 형식
추가적으로, `SockJS`는 `Message Frame` 크기를 최소화하기 위해 노력한다.
예를 들어, `open frame` 경우에는 첫 글자인 `o`를 전송한다.
![[Pasted image 20250507191456.png]]

또한, `Message Frame`의 경우에는 다음과 같은 형태로 전달받는다.
```jsx
a["message1", "message2"]
```
![[Pasted image 20250507191521.png]]

커넥션 유지 여부를 확인하는 `Heartbeat Frame` 경우에는 `h` 로 보낸다.
![[Pasted image 20250507191550.png]]

마지막으로, 커넥션 종료를 의미하는 `Close Frame`은 `c["message"]` 형식으로 보낸다.
![[Pasted image 20250507192212.png]]

### IE 8, 9 호환성
여전히 많은 사용자들은 Internet Explorer 브라우저의 8, 9 버전을 여전히 사용하고 있지만, 해당 버전에서는 `WebSocket`을 지원하고 있지 않다.

이러한 부분에서 `SockJS` 진가가 발휘되는데, IE 8, 9 버전은 `HTTP Streaming` 또는 `HTTP Long Polling` Transports 타입으로 전환되어 호환이 가능하기 때문이다.

`SockJS` 클라이언트는 Microsoft의 [`XDomainRequest(xdr)`](https://developer.mozilla.org/en-US/docs/Web/API/XDomainRequest) 이용해서 `HTTP Streaming`을 지원한다.(10 버전부터는 [`XMLHttpRequest(xhr)`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 사용을 권장하여 [`XDomainRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XDomainRequest) 를 제거)

> XDomainRequest(xdr), XMLHttpRequest(xhr)는 모두 CORS를 지원하기 위한 도구이다.

`XDomainRequest`는 비록 `CORS` 도구로서 잘 동작하지만, `Cookies` 전송을 지원하지 않는다.

`Cookies` 는 종종 Java 애플리케이션에서 필수적이지만, `SockJS` 클라이언트는 여러 유형의 서버와 함께 사용할 수 있기 때문에 큰 문제는 아니다.

따라서, 서버 측의 `Cookies` 필요 여부에 따라 `HTTP Streaming`, `HTTP Long Polling`에서 사용하는 기술이 달라진다.
- `Cookies` 사용 불가능하다면, `XDomainRequest(xdr)`가 사용된다.
- `Cookies` 사용 가능하다면, `iframe` 기반의 기술이 사용된다.

`SockJS` 클라이언트가 첫 번째로 요청한 `GET /info`에 대한 응답 메시지에는 클라이언트가 `Transports` 타입을 선택하는데 영향을 미치는 요소들이 포함되어 있다.
![[Pasted image 20250507192720.png]]
위 그림과 같이, 서버가 `Cookies` 정보가 필요한 지 등의 정보를 클라이언트에게 응답 메시지로 전달한다.

`cookie_needed` 항목은 스프링에서 `setSessionCookieNeeded(bolean)` 메서드로 제어가 가능한데, 아래와 같이 설정할 수 있다.(Java 애플리케이션에서 `JSESSIONID` 쿠키를 많이 사용하기 때문에 디폴트 설정은 `true` 이다.)
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
		webSocketHandlerRegistry
			.addHandler(webSocketHandler(), "/test")
			.withSockJS()
			.setSessionCookieNeeded(false);
	}

	...
}
```

![[Pasted image 20250507192737.png]]
위에서 말했다시피, 만약 서버에서 쿠키가 필요하지 않다면 `SockJS` 클라이언트는 `IE 8, 9` 버전에서 `XDomainRequest`를 사용하게 한다.

또한, `iframe` 기반의 `Transports`를 사용하는 경우에는 브라우저가 `[X-Frame-Options](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Frame-Options)` 응답 헤더에 지정한 `DENY`, `SAMEORIGIN`, `ALLOW-FROM <origin>` 페이지들에 대해서만 `iframe`을 렌더링하도록 방지할 수 있다.
- `DENY`는 모든 `iframe`에서 사용될 수 없다.
- `SAMEORIGIN`은 동일한 출처, 즉 같은 도메인인 경우에만 허용
- `ALLOW-FROM <origin>`는 지정한 도메인 URI에 대해서만 허용

> [!info]
> X-Frame-Options 응답 헤더는 해당 페이지를 `<frame>`, `<iframe>`, `<object>`에서 렌더링할 수 있는 지 여부를 의미한다.

만약 `iframe` 기반의 `Transports`를 사용하고 `X-Frame-Options` 응답 헤더를 포함하려면, 반드시 `SAMEORIGIN` 이거나 `ALLOW-FROM <origin>`에 `SockJS` 클라이언트 도메인을 지정해야만 한다.

즉, `iframe`로 부터 로드되기 위해서는 스프링 서버의 `SockJS`가 클라이언트의 위치를 알고 있어야 한다는 것이다.

따라서, 스프링은 `SAMEORIGIN`을 지원하기 위해서 `SockJS-Client` 접근 경로를 설정할 수 있도록 아래와 같이 제공한다.
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
		webSocketHandlerRegistry
			.addHandler(webSocketHandler(), "/test")
			.withSockJS()
			.setClientLibraryUrl("http://localhost:8080/myApplication/js/sockjs-client.js")
	}

	@Bean
	public WebSocketHandler webSocketHandler() {
		return new Handler();
	}
}
```

### Heartbeat
`SockJS` 프로토콜은 서버가 주기적으로 `Heartbeat Message` 전송하여, 프록시가 커넥션이 끊겼다고 판단하지 않도록 한다.

스프링 `SockJS Configuration`은 `HeartbeatTime`을 사용자가 지정할 수 있도록 `setHeartbeatTime(long)` 메서드를 제공하는데, `HeartbeatTime`의 시작은 마지막 메시지가 전송된 이후부터 카운트된다. (디폴트는 25초이다)
```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
		webSocketHandlerRegistry
			.addHandler(webSocketHandler(), "/test")
			.withSockJS()
			.setHeartbeatTime(30)
	}

	@Bean
	public WebSocketHandler webSocketHandler() {
		return new Handler();
	}
}
```
뿐만 아니라, 스프링 `SockJS`은 `Heartbeat Tasks`를 스케줄링할 수 있도록 `TaskScheduler`를 설정할 수도 있다. `TaskScheduler`는 기본적으로 사용 가능한 프로세서 수만큼 생성되어 Thread Pool에 백업된다.

만약 `STOMP`를 사용해 `Heartbeat`를 주고 받을 경우에는 `SockJS` `Heartbeat` 설정은 비활성화된다.

### Client Disconnects
`SockJS` Transports 타입인 `HTTP Streaming`과 `HTTP Long Polling`는 일반 요청보다 더 긴 커넥션을 요구한다.

이러한 요구 사항은 서블릿 컨테이너에서 `Servlet 3 asynchronous` 지원을 통해 수행된다.

구체적으로, `Servlet 3 asynchronous`는 `Servlet Container Thread`가 종료되고도 요청을 처리하며 다른 스레드가 지속적으로 응답에 Write 할 수 있도록 지원한다.

여기서 문제점은 `Servlet API`가 갑자기 사라진 클라이언트에 대한 알림을 제공하지 않는다는 것이다.

그러나 다행히도, `Servlet Container`는 응답에 Write를 시도하는 경우 예외를 발생시킨다.

뿐만 아니라, 스프링의 `SockJS`는 서버 측에서 주기적으로 `Heartbeat`를 전송하기 때문에 클라이언트의 연결 여부를 일정 시간 안에 파악할 수 있다.

### SockJS와 CORS
만약 Cross-Origin Requests(CORS)를 허용한다면, `SockJS` 프로토콜은 `HTTP Streaming`, `HTTP Long Polling` 과정에서 해당 CORS를 사용한다.

따라서, 스프링은 응답 헤더에서 CORS 헤더를 발견되지 않는다면 `SockJS` CORS에서 설정한 정보를 기반으로 헤더를 추가한다.

만약 `Servlet Filter`를 통해서 이미 CORS 설정한 경우에는 스프링의 `SockJsService`에서의 CORS 설정은 건너뛴다. 또한, 각 핸들러에서는 `setSupressCors(boolean)` 메서드를 이용해서 `SockJsService`를 통한 CORS 헤더 추가 여부를 설정할 수 있다.

`SockJsService`에서 CORS 헤더를 추가하도록 설정하고 싶은 경우에는 `SockJS` Endpoint의 Prefix에 대해서는 `Servlet Filter`를 제외하도록 설정한다.

`SockJS`에서는 CORS 헤더에 아래와 같은 값이 필요하다.
- `Access-Control-Allow-Origin`는 `Origin` 요청 헤더의 값으로 초기화된다.
- `Access-Control-Allow-Credentials`는 항상 `True`로 설정된다.
- [`Access-Control-Request-Headers`](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Access-Control-Request-Headers)는 실제 요청이 만들어질 때 클라이언트가 보낼 수도 있는 HTTP headers를 서버에게 알리는 용도로, 브라우저가 [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/preflight_request)를 보내는 경우에 사용된다. `SockJS`에서는 Request와 동일한 헤더로 설정한다.
- `Access-Control-Allow-Methods`는 서버가 지원하는 Transports 타입의 HTTP METHOD를 설정한다.
- `Access-Control-Max-Age`는 preflight request 결과를 얼마나 캐시할 지를 나타내고, 31536000(1년)으로 설정된다.
![[Pasted image 20250507193050.png]]

