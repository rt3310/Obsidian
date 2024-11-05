https://docs.unity3d.com/2022.3/Documentation/Manual/ExecutionOrder.html#InBetweenFrames

이벤트 함수는 MonoBehaviour에 내장된 이벤트 세트다. MonoBehaviour는 적절한 함수를 구현하여 선택적으로 구독할 수 있다 (콜백이라고도 부른다).
콜백은 물리, 렌더링, 사용자 입력과 같은 핵심 Unity 하위 시스템의 이벤트 또는 스크립트 자체의 라이프사이클 단계(e.g. 생성, 활성화, 프레임 종속 및 프레임 독립 업데이트, 파괴)에 해당한다.
이벤트가 발생하면 Unity는 스크립트에서 연관된 콜백을 호출하여 이벤트에 대한 응답으로 로직을 구현할 수 있는 기회를 제공한다.

## 플로우차트의 범위
아래 플로우차트의 범위는 MonoBehaviour 스크립팅 참조의 메세지에 문서화 된 적절한 콜백을 구현하여 모든 MonoBehaviour 스크립트에서 구독할 수 있는 내장 이벤트 함수로 제한된다.
이벤트를 발생시키는 하위 시스템의 로컬인 일부 추가 내부 메서드도 컨텍스트를 위해 표시된다.

이러한 내장 이벤트 함수 외에도 스크립트에서 잠재적으로 구독할 수 있는 다른 이벤트가 많이 있다.
[Application](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Application.html), [SceneManager](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SceneManagement.SceneManager.html), [Camera](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Camera.html) 같은 몇몇 주요 클래스들은 자신의 콜백 메서드를 등록할 수 있는 delegate를 제공한다.
[RuntimeInitializeOnLoadMethodAttribute](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/RuntimeInitializeOnLoadMethodAttribute.html)와 같은 메서드 속성은 scene의 특정 단계에서 메서드를 실행하는 데에도 사용할 수 있다.
![[Pasted image 20241023161752.png]]
## 일반적인 원칙

일반적으로, 서로 다른 GameObject에 대해 동일한 이벤트 함수가 호출되는 순서에 의존해서는 안된다(순서가 명시적으로 문서화되거나 설정 가능한 경우는 제외). 플레이어 루프를 보다 세밀하게 제어해야 할 경우 [PlayerLoop API](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/LowLevel.PlayerLoop.html)를 사용할 수 있다.
동일한 MonoBehaviour 하위 클래스의 여러 인스턴스에 대해 이벤트 함수가 호출되는 순서를 지정할 수 없다.