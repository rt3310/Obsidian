Connector는 **애플리케이션을 클라이언트와 연결**한다.
Connector는 어느 접점(point)에서 클라이언트로부터 요청이 받아지는지 나타내며, 서버 상의 포트를 할당한다.

Nonsecure HTTP 애플리케이션을 위한 default port는 기본 웹 서버의 80과 충돌을 피하기 위해 8080이지만, 설정을 통해 쉽게 변경할 수 있다.
여러 개의 connector가 하나의 엔진 또는 엔진-레벨 컴포넌트에 설정될 수 있으나, 각각 유니크한 포트 숫자를 지녀야 한다.

default connector는 Coyote이다.

Connector는 아래와 같이 생성자에서 ProtocolHandler의 `create()` 메서드를 통해 프로토콜에 맞는 ProtocolHandler를 설정하여 클라이언트의 요청을 컨테이너에 연결한다.

```java
public interface ProtocolHandler {
	// ...
	public static ProtocolHandler create(String protocol) /* throws ... */ {
		if (protocol == null || "HTTP/1.1".equals(protocol)
			|| org.apache.coyote.http11.Http11NioProtocol.class.getName().equals(protocol)) {
			return new org.apache.coyote.http11.Http11NioProtocol();
		} else if ("AJP/1.3".equals(protocol)
			|| org.apache.coyote.ajp.AjpNioProtocol.class.getName().equals(protocol)) {
			return new org.apache.coyote.ajp.AjpNioProtocol();
		} else {
			Class<?> clazz = Class.forName(protocol);
			return (ProtocolHandler) clazz.getConstructor().newInstance();
		}
	}
}

public class Connector extends LifeCycleMBeanBase {
	// ...
	public Connector(String procotol) {
		configuredProtocol = protocol;
		ProtocolHandler p = null;
		try {
			p = ProtocolHandler.create(protocol);
		}
		// ...
	}
	// ...
}
```