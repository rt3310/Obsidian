밸브는 **하나의 특정 컨테이너와 관련있는 요청 처리 컴포넌트**이다.
밸브는 서블릿 명세의 필터 메커니즘과 유사하지만, 톰캣의 고유한 것이다.
호스트, 컨텍스트, 엔진은 밸브를 가진다: StandardHostValve, StandardContextValve, StandardEngineValve, StandardWrapperValve, ...

밸브 인터페이스는 아래와 같이 간단한 기본 구조를 가지며, 각 밸브의 로직은 invoke 메서드에 다양한 형태로 구현되게 된다.
```java
public interface Valve {
	public Valve getNext();
	
	public void setNext(Valve valve);
	
	public void invoke(Request request, Response Response) throws IOException, ServletException;
	
	public boolean isAsyncSupported();
}
```

예시로 아래와 같은 StandardEngineValve는 요청에서 호스트 객체를 꺼내고 없으면 에러를, 있으면 호스트의 파이프라인에서 가장 처음의 밸브를 꺼내어 `invoke(request, response)`를 실행한다.
```java
final class StandardEngineValve extends ValveBase {

	@Override
	public final void invoke(Request request, Response response)
		throws IOException, ServletException {

		Host host = request.getHost();
		if (host == null) {
			// HTTP 0.9 or HTTP 1.0 request without a host when no default host is defined.
			// Don't overwrite an existing error
			if (!response.isError()) {
				response.sendError(404);
			}
			return;
		}
		if (request.isAsyncSupported()) {
			request.setAsyncSupported(host.getPipeline().isAsyncSupported());
		}

		// Ask this Host to process this request
		host.getPipeline().getFirst().invoke(request, response);
	}
}
```