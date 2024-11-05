https://docs.unity3d.com/2022.3/Documentation/Manual/ExecutionOrder.html#InBetweenFrames

이벤트 함수는 MonoBehaviour에 내장된 이벤트 세트다. MonoBehaviour는 적절한 함수를 구현하여 선택적으로 구독할 수 있다 (콜백이라고도 부른다).
콜백은 물리, 렌더링, 사용자 입력과 같은 핵심 Unity 하위 시스템의 이벤트 또는 스크립트 자체의 라이프사이클 단계(e.g. 생성, 활성화, 프레임 종속 및 프레임 독립 업데이트, 파괴)에 해당한다.
이벤트가 발생하면 Unity는 스크립트에서 연관된 콜백을 호출하여 이벤트에 대한 응답으로 로직을 구현할 수 있는 기회를 제공한다.

## 플로우차트의 범위
아래 플로우차트의 범위는 MonoBehaviour 스크립팅 참조의 메세지에 문서화 된 적절한 콜백을 구현하여 모든 MonoBehaviour 스크립트에서 구독할 수 있는 내장 이벤트 함수로 제한된다.
이벤트를 발생시키는 하위 시스템의 로컬인 일부 추가 내부 메서드도 컨텍스트를 위해 표시된다.

![[Pasted image 20241023161752.png]]
## 일반적인 원칙

일반적으로, you should not rely