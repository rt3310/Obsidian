엔진은 top-level 컨테이너로 다른 컨테이너가 포함할 수 없다(parent 컨테이너를 가지지 못한다).
**이 레벨에서부터 객체들이 child 컴포넌트들을 가지기 시작**한다.

컨테이너는 꼭 엔진일 필요는 없고, 위에서 제시된 서블릿 컨테이너의 명세를 만족하면 된다. 하지만, 보통 이 레벨의 컨테이너는 엔진이기에 엔진이라는 가정을 두고 다음을 기술한다.

엔진(Engine)은 하나의 컨테이너로 **전체 Catalina 서블릿 엔진**을 나타낸다.
엔진은 **HTTP 헤더를 체크하여 특정 요청이 어느 가상 호스트 또는 어느 컨텍스트에 연결되어야 하는지 판단**한다.

특정 설정 변경 없이 실행된다면, default enging을 사용한다.
이 엔진은 위에서 언급된 체크를 진행한다. 톰캣이 웹 서버를 위해서 Java Servlet 지원을 제공하도록 설정된 경우에는, 보통 웹 서버가 요청에 대한 연결을 진행하기 때문에 요청에 사용하도록 default로 설정된 클래스가 overriden 된다.

Catalina에서 top-level 컨테이너인 엔진의 Standard 구현체는 StandardEngine 클래스로 아래와 같은 상속 관계를 지닌다.
```java
public interface Lifecycle  { ... }

public interface Container extends Lifecycle { ... }

public interface Engine extends Container { ... }

public class StandardEngine extends ContainerBase implements Engine { ... }
```