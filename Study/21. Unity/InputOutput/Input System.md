- VR, 모바일, 콘솔 등 여러 입력에 유연하게 대응하기 위해 기존의 `Input Manager`를 대체하는 시스템
- Package Manager에서 `Input System`을 install하여 사용할 수 있다.
### Input Actions
- 게임 내 모든 입력을 일괄적으로 매핑하게 해주는 시스템
- **Action Maps**에 입력 값을 받을 대상 지정
- **Actions**에 지정한 대상이 실시할 액션을 추가
- **Properties**에 액션을 실행시킬 입력 값을 할당
### Actions Mapping
#### 입력받는 방법
- **Value**: 기본값, 컨트롤 상태에 대한 지속적인 변경을 추적해야 하는 모든 입력에 사용
- **Button**: 누를 때마다 한 번씩 동작을 트리거하는 입력에 사용
- **Pass Through**: 바인딩 된 컨트롤을 변경하면 해당 컨트롤 값으로 콜백이 트리거 된다.
	- 액션 집합의 모든 입력을 처리하려는 경우 유용하다.
### Bind With Code
![[Pasted image 20241128173256.png]]
- **Send Message**: 키가 입력되면 해당 오브젝트의 모든 컴포넌트에 메시지 함수를 보낸다.
- **BroadCast Message**: Send Message와 동일하지만 자식 컴포넌트들에게도 메시지가 보내진다.
- **Invoke Unity Event**: 유니티의 Event System과 동일한 방식으로 객체 내 함수를 해당 입력 이벤트에 바인딩한다.
- **Invoke C Sharp Events**: Input Actions에서 설정한 키들을 바탕으로 유니티가 C# 코드를 작성해준다.
	- Save Asset시 코드도 자동으로 업데이트 된다.

## C# 이벤트 호출

![[Pasted image 20241130161056.png]]
- 유니티가 작성한 입력 클래스를 사용할 클래스에 선언하여 사용한다.
- 선언 후 원하는 Actions Maps를 활성화시킨다.
- Actions에 입력된 각각의 키 바인딩에 맞춰 실행할 함수들을 추가해준다.
	- **Waiting**: 키 입력 대기 시
	- **Started**: 키 입력이 시작되어 값은 받았지만 실행이 완료되지 않았을 시
	- **performed**: 키 입력받고 실행이 완료되었을 때
	- **canceled**: 키 입력 해제 시

### Change Action Maps
-  인게임 -> 일시 정지 상태 UI 등 게임 상황이 바뀌며 입력 값 처리도 바꾸리 때 사용한다.
- `PlayerInput.SwitchCurrentActionMap(string actionName)` 함수로 변경

### Button Remapping
![[Pasted image 20241130164830.png]]
- Invoke C Sharp Event를 이용할 때 변경을 원하는 키 값에 접근한 후 `PerformInteractiveRebinding()` 함수를 사용하여 변경 가능
### Touch Contorls
- InputSystem 설치 시 `On-Screen Stick` 컴포넌트도 설치된다.
- 캔버스의 가상스틱에 해당 컴포넌트를 import하여 터치 컨트롤 구현 가능
### 깔끔한 액션 할당 및 해제
액션의 중복 할당, 오남용 등을 방지하기 위해서 다음과 같은 작업을 해준다.
- Enable, Disable 될 때 바인딩 연결 및 해제와 액션의 활성화 여부를 변경한다
```c#
private void OnEnable()
{    
    inputAction_track1.started += CallbackTrack1;
    inputAction_track1.Enable();       
}

private void OnDisable()
{
    inputAction_track1.started -= CallbackTrack1;
    inputAction_track1.Disable();
}
```
- 원하는 함수가 맞는 키 상태에 발동되는지 확인한다.
```c#
private void CallbackTrack1(InputAction.CallbackContext context)
{
    if(context.action.phase == InputActionPhase.Started)
    {
        Attack();
    }
}
```

## InputSystem의 액션 타입
### 액션 타입 개요
InputSystem의 프로퍼티 설정에서 액션 타입은 Value, Button, Pass Through의 총 세 가지 종류가 있다.

인풋 시스템은 이벤트 기반이다.
즉 특정 입력이 들어오면 인풋 시스템이 이에 해당하는 이벤트들을 발생시키는 구조이다.

개발자는 해당 입력 이벤트에 원하는 함수들을 구독시켜놓음으로써 입력을 할당한다.
![[Pasted image 20241130165308.png]]
### 인터랙션(interaction)
InputSystem에서 할당하는 모든 액션은 interaction을 갖고 있다. interaction은 단순하게 게임 내에서 상호작용하는 방식을 뜻한다.
개발자는 여러 종류의 interaction을 액션 매핑 도중 추가할 수 있다.
![[Pasted image 20241130165410.png]]
모든 액션은 기본적인 interaction을 갖고 있다. 이 interaction은 구 입력 시스템의 Press와 유사하다.

![[Pasted image 20241130165430.png]]
- Started: 입력 값이 들어와 interaction이 시작되는 때
- Performed: interaction이 완료되었을 때 (키를 충분히 오래 눌러 Hold interaction이 완료되었을 때 등)
- Canceled: interaction이 취소되었을 때 (키를 누르던 중간에 때어 Hold interaction이 완료되지 않았을 때 등)
#### Interaction Timeline
![[Pasted image 20241130165542.png]]
### 값(Value)
개발자는 입력에서 다양한 값들을 받아올 수 있다.
![[Pasted image 20241130165644.png]]
#### 명확화
값을 받아올 때 InputSystem은 **명확화(disambiguation)** 를 실시한다. InputSystem의 명확화란 **어떤 것이 주 조작(main control)인지 결정하는 과정**을 뜻한다.
이는 한 번에 하나의 입력을 기대할 때 매우 유용한 과정이다.

![[Pasted image 20241130165759.png]]
명확화는 값 변화에 따른 interaction을 확인하면 이해하기 쉽다
- **Start** interaction은 액션에 대한 값이 기본값과 달라지면 실행된다.
- **Performed** interaction은 액션에 대한 값이 변화될 때마다 실행된다.
- **Canceled** interaction은 액션에 대한 값이 다시 기본값으로 돌아갈 때 실행된다.

명확화로 인해 Value와 Passthrough의 차이점이 갈린다.
Passthrough는 모든 입력 값을 가져오지만, Value는 명확화와 초기 상태 체크를 수행하여 걸러진 입력 값을 가져온다.
#### 초기 상태 체크
Value의 초기 상태 체크는 입력 액션 활성화 시에 실시하는 상태 체크를 말한다.
만약 액션이 `Enable()`로 활성화되었을 때 컨트롤러가 default값이 아닌 값을 갖고 있으면, InputSystem은 이에 해당하는 이벤트를 전송해준다.
반면, Passthrough는 활성화되었을 때의 값이 default 값이 아니더라도 이벤트가 전송되지 않는다.
### 버튼(Button)
버튼은 입력 시 값 전달 없이 액션을 트리거시키는 종류이다.
- 초기 상태를 체크하지 않는다.
- interaction을 이용해 이벤트의 전송 시점을 변화시킬 수 있다.
	- 예를 들어, Press interaction을 추가하여 한계점을 등록하면, 버튼이 눌리기 시작했을 때 Start가 호출되고, 버튼이 한계점 이상으로 눌렸을 때 Performed가 호출된다.
### 패스스루(Passthrough)
- 명확화를 하지 않아 주 입력이 없이 모든 입력을 직접적으로 받아온다.
- interaction에서 설정해주지 않는 한 Start와 Canceled 이벤트를 전송하지 않는다.
- 초기 상태를 체크하지 않는다.

### Value vs Passthrough
![[Pasted image 20241130172732.png]]