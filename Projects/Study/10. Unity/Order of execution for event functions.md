https://docs.unity3d.com/2022.3/Documentation/Manual/ExecutionOrder.html#InBetweenFrames

이벤트 함수는 MonoBehaviour에 내장된 이벤트 세트다. MonoBehaviour는 적절한 함수를 구현하여 선택적으로 구독할 수 있다 (콜백이라고도 부른다).
콜백은 물리, 렌더링, 사용자 입력과 같은 핵심 Unity 하위 시스템의 이벤트 또는 스크립트 자체의 라이프사이클 단계(e.g. 생성, 활성화, 프레임 종속 및 프레임 독립 업데이트, 파괴)에 해당한다.
이벤트가 발생하면 Unity는 스크립트에서 연관된 콜백을 호출하여 이벤트에 대한 응답으로 로직을 구현할 수 있는 기회를 제공한다.

## 플로우차트의 범위
아래 플로우차트의 범위는 MonoBehaviour scripting reference의 메세지에 문서화 된 적절한 콜백을 구현하여 모든 MonoBehaviour 스크립트에서 구독할 수 있는 내장 이벤트 함수로 제한된다.
이벤트를 발생시키는 하위 시스템의 로컬인 일부 추가 내부 메서드도 컨텍스트를 위해 표시된다.

이러한 내장 이벤트 함수 외에도 스크립트에서 잠재적으로 구독할 수 있는 다른 이벤트가 많이 있다.
[Application](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Application.html), [SceneManager](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SceneManagement.SceneManager.html), [Camera](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/Camera.html) 같은 몇몇 주요 클래스들은 자신의 콜백 메서드를 등록할 수 있는 delegate를 제공한다.
[RuntimeInitializeOnLoadMethodAttribute](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/RuntimeInitializeOnLoadMethodAttribute.html)와 같은 메서드 속성은 scene의 특정 단계에서 메서드를 실행하는 데에도 사용할 수 있다.
![[Pasted image 20241023161752.png]]

## 일반적인 원칙

일반적으로, 서로 다른 GameObject에 대해 동일한 이벤트 함수가 호출되는 순서에 의존해서는 안된다(순서가 명시적으로 문서화되거나 설정 가능한 경우는 제외). 플레이어 루프를 보다 세밀하게 제어해야 할 경우 [PlayerLoop API](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/LowLevel.PlayerLoop.html)를 사용할 수 있다.

동일한 MonoBehaviour 하위 클래스의 여러 인스턴스에 대해 이벤트 함수가 호출되는 순서를 지정할 수 없다. 예를 들어, 한 MonoBehaviour의 `update` 함수는 다른 GameObject의 동일한 MonoBehaviour에 대한 `update` 함수 전후에 호출될 수 있다(자신의 부모 또는 자식 GameObject 포함해서).

프로젝트 설정 창의 [Script Execution Order](https://docs.unity3d.com/2022.3/Documentation/Manual/class-MonoManager.html) 패널을 사용하여 하나의 MonoBehaviour 하위 클래스의 이벤트 함수가 다른 하위 클래스의 이벤트 함수보다 먼저 호출되도록 지정할 수 있다. 예를 들어, EngineBehaviour와 SteeringBehaviour라는 두 스크립트가 있는 경우 EngineBehaviour가 항상 SteeringBehaviour보다 먼저 업데이트되도록 실행 순서를 설정할 수 있다.

[여러 scene을 추가로 구성하는 경우](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SceneManagement.LoadSceneMode.Additive.html), 구성된 스크립트 실행 순서는 여러 scene에 걸쳐 부분적으로 적용되는 것이 아니라 한 번에 한 scene 전체에 적용되므로, EngineBehaviour와 SteeringBehaviour는 모두 다음 scene에서 업데이트 되기 전에 한 scene에서 업데이트 된다.

## 첫 번째 Scene load

이 함수들은 scene이 시작될 때 호출된다(scene의 object 당 한 번씩).
### Awake
- object의 인스턴스가 새로 생성될 때 호출되는 첫 번째 lifecycle 함수다.
- 시작하는 동안 GameObject가 inactive 상태인 경우, `Awake`는 active될 때까지 호출되지 않는다.
- 항상 `Start` 함수 전에 호출된다.
### OnEnable
- object가 enabled되고 active될 때 호출된다.
- 항상 `Awake` 이후(동일 object에서), `Start` 이전에 호출된다.

scene asset의 일부인 object의 경우, 모든 스크립트에 대한 `Awake` 및 `OnEnable` 함수는 `Start` 이전에 호출되고 후속 함수는 이 중 하나에 대해 호출된다. 하지만, 런타임에 object를 인스턴스화하는 경우에는 이를 적용할 수 없다.

**`Awake`는 각 개별 object의 범위에서 `OnEnable` 이전에만 호출되도록 보장된다**.
즉, 여러 object에 걸쳐, 순서는 결정적이지 않으며 다른 object의 `OnEnable`이 호출되기 전에 한 객체의 `Awake`가 호출된다고 믿을 수 없다. 때문에 scene의 모든 object에 대해 호출된 `Awake`에 의존하는 모든 작업은 `Start`에서 수행되어야 한다.

> [!note] scene load 및 unload 전
> 위 다이어그램에는 scene이 각각 load 및 unload될 때 콜백을 받을 수 있는 [SceneManager.sceneLoaded](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SceneManagement.SceneManager-sceneLoaded.html) 및 [SceneManager.sceneUnloaded](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/SceneManagement.SceneManager-sceneUnloaded.html) 이벤트가 표시되지 않았다. 자세한 내용과 사용법 예시는 관련 scripting reference 페이지를 참고.
> 
> `OnEnable` 이후와 scene의 모든 object에 대한 `Start` 이전에 `sceneLoaded` 알림을 받을 수 있다. 실행 흐름의 일부로 scene load를 포함하는 다이어그램은 [Details of disabling Domain and Scene reload](https://docs.unity3d.com/2022.3/Documentation/Manual/ConfigurableEnterPlayModeDetails.html)를 참고
> 
> [RuntimeInitializeOnLoadMethodAttribute](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/RuntimeInitializeOnLoadMethodAttribute.html)와 해당 유형 [BeforeSceneLoad](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/RuntimeInitializeLoadType.BeforeSceneLoad.html) 및 [AfterSceneLoad](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/RuntimeInitializeLoadType.AfterSceneLoad.html)를 사용하여 각각 scene load 전후에 메서드를 실행할 수도 있다.
> 
> 이러한 유형으로 표시된 메서드에 대한 실행 순서 정보는 [RuntimeInitializeOnLoadMethodAttribute](https://docs.unity3d.com/2022.3/Documentation/ScriptReference/RuntimeInitializeOnLoadMethodAttribute.html) scripting reference 기본 페이지를 참고

## Editor

### Reset
- 스크립트가 object에 처음 연결될 때와 `Reset` 명령이 사용될 때, 스크립트 속성을 초기화하기 위해 호출된다.
### OnValidate
- Editor에서 scene을 열 때나 도메인을 reload한 후 등, 다양한 시점에 발생할 수 있는 object 역직렬화를 포함하여 script의 속성이 설정될 때마다 호출된다.

## 첫 번째 frame 업데이트 전

### Start
- script 인스턴스가 enabled 된 경우에만, 첫 번째 frame 업데이트 전에 호출된다.

scene asset의 일부인 object의 경우, `Update`가 호출되기 전에 모든 script에서 `Start` 함수가 호출된다. 하지만 게임플레이 중에 object를 인스턴스화 할 때는 이를 강제할 수 없다.
예를 들어, 다른 object의 `Update` 함수에서 object를 인스턴스화 하는 경우, 원본 object에서 `Update`가 처음으로 실행될 때까지 인스턴스화 된 object의 `Start`를 호출할 수 없다.

## frame 사이

### OnApplicationPause
- 이는 일시정지가 감지된 frame 끝에서 호출되며, 사실상 일반 frame `Update`사이에 호출된다. 게임이 일시정지된 상태를 나타내는 그래픽을 표시할 수 있도록 `OnApplicationPause`가 호출된 후에 하나의 추가 프레임이 발행된다.

## Update 순서

게임 로직과 상호작용, 애니메이션, 카메라 위치 등을 추적할 때 사용할 수 있는 몇 가지 이벤트가 있다. 일반적인 패턴은 `Update` 함수 내에서 대부분의 작업을 수행하는 것이지만, 사용할 수 있는 다른 기능도 있다.
### FixedUpdate
`FixedUpdate`는 frame 당이 아닌 게임 내 시간의 고정된 간격으로 발생한다. 이런 update는 고정되어있고 frame 속도는 가변적이므로, frame 속도가 높을 때 frame 중에 fixed update가 없고 frame 속도가 낮을 때 frame 당 fixed update가 여러 개 있을 수 있다.
### Update
### LateUpdate
