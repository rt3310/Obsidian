- VR, 모바일, 콘솔 등 여러 입력에 유연하게 대응하기 위해 기존의 `Input Manager`를 대체하는 시스템
- Package Manager에서 `Input System`을 install하여 사용할 수 있다.

## Input Actions

- 게임 내 모든 입력을 일괄적으로 매핑하게 해주는 시스템
- **Action Maps**에 입력 값을 받을 대상 지정
- **Actions**에 지정한 대상이 실시할 액션을 추가
- **Properties**에 액션을 실행시킬 입력 값을 할당

## Actions Mapping

### 입력받는 방법
- **Value**: 기본값, 컨트롤 상태에 대한 지속적인 변경을 추적해야 하는 모든 입력에 사용
- **Button**: 누를 때마다 한 번씩 동작을 트리거하는 입력에 사용
- **Pass Through**: 바인딩 된 컨트롤을 변경하면 해당 컨트롤 값으로 콜백이 트리거 된다.
	- 액션 집합의 모든 입력을 처리하려는 경우 유용하다.

## Bind With Code

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
