컴포넌트(Components)는 액터(Actors)가 자기 자신에 서브 Object로 attach할 수 있는 특수한 타입의 Object이다.
Components는 비주얼 표현 디스플레이 또는 사운드 재생 기능과 같은 일반적인 동작을 공유할 때 유용하다. 또한 Components는 Vehicle이 입력을 인식하고 속도 및 방향을 변경하는 방식과 같은 프로젝트 별 콘셉트를 표현할 수도 있다.
예를 들어, 사용자가 컨트롤 가능한 자동차, 항공기, 배가 있는 프로젝트의 경우 Vehicle Actor가 사용하는 Components의 종류를 변경함으로써 Vehicle Control 및 Movement의 차이를 구현할 수 있다.

## 액터 컴포넌트(Actor Components)

`UActorComponent`는 모든 컴포넌트에 대한 베이스 클래스이다.
Components는 메시와 이미지를 렌더링하고, Collision을 구현하며, 오디오를 재생할 수 있는 유일한 방법이기 때문에, 플레이어가 게임을 플레이할 때 월드에서 보거나 상호작용하는 모든 것은 궁극적으로 여러 타입의 컴포넌트 작업으로 이루어진다.

개성 있는 Components를 생성할 때 이해해야 할 주요 클래스는 액터 컴포넌트(Actor Components), 씬 컴포넌트(Scene Components), 프리미티브 컴포넌트(Primitive Components)이다.

