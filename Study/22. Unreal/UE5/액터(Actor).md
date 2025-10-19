## 액터 생성하기

`AActor` 클래스의 새 인스턴스를 생성하는 것을 스폰(Spawn)이라고 한다. 일반 `SpawnActor()` 함수나 특화된 템플릿 버전 중 하나를 사용하여 스폰할 수 있다.

## [[컴포넌트(Components)]]

액터(Actors)는 어떤 의미에서 [[컴포넌트(Components)]]라고 하는 특별한 타임의 오브젝트(Objects)를 담는 컨테이너로 생각할 수 있다. 다양한 타임의 컴포넌트를 사용하여 Actor의 이동 및 랜더링 방식을 제어할 수 있다. 또 다른 액터의 주요 기능은 플레이 도중 네트워크를 통해 프로퍼티 및 함수 호출을 [[리플리케이션(Replication)]] 하는 것이다.

컴포넌트는 생성될 때 자신이 포함된 액터에 연결된다.

몇 가지 주요 컴포넌트 타입은 다음과 같다.
- **UActorComponent**
	- 베이스 컴포넌트이다. Actor의 일부로 포함될 수 있다. 원하면 Tick하게 할 수 있다.
	- 액터 컴포넌트(ActorComponents)는 특정 Actor와 연결되어 있지만, 월드의 특정 위치에 존재하지는 않는다.
	- 보통 AI 또는 플레이어 입력 해석과 같은 개념적 기능에 사용된다.
- **USceneComponent**
	- 씬 컴포넌트(SceneComponents)는 Transform이 있는 ActorComponents이다.
	-  Transform은 월드에서의 위치로, Position과 Rotation, Scale로 정의된다.
	-  SceneComponents 계층형 방식으로 서로 attach 될 수 있다.
	-  Actor의 Position과 Rotation, Scale은 계층구조의 루트에 있는 SceneCompoents에서 가져온다.
- **UPrimitiveComponent**
	- 프리미티브 컴포넌트(PrimitiveComponents)는 메시나 파티클 시스템 같은 일종의 그래픽 표현이 있는 SceneComponents이다. 여기에는 흥미로운 Physics 및 Collision 세팅이 많이 있다.

Actor는 SceneComponents의 계층구조를 포함할 수 있다. 각 Actor에는 Actor의 루트 역할을 할 컴포넌트를 지정하는 `RootComponent` 프로퍼티도 있다.
Actor 자체에는 Transform이 없으므로 Position, Rotation, Scale도 없다. 대신, 자신의 컴포넌트, 좀 더 구체적으로 루트 **컴포넌트의 Transform**에 의존한다. 이 컴포넌트가 **SceneComponents**인 경우, Actor에 대한 Transform 정보를 제공한다. 그렇지 않다면, 해당 Actor에는 Transform이 없다.
Attach된 다른 컴포넌트에는, Attach 된 대상 컴포넌트를 기준으로 한 Transform 정보가 있다.

### 예시
액터와 계층구조의 예시는 다음과 같다.
![[Pasted image 20251019172542.png]]

## [[Ticking(티킹)]]

Ticking은 Actor가 언리얼 엔진에서 업데이트되는 방식을 나타낸다.
모든 Actor는 프레임마다, 또는 최소한 사용자 정의 간격마다 Tick할 수 있는 기능이 있으므로, 필요한 동작이나 계산을 업데이트할 수 있다.

ActorComponents도 기본적으로 업데이트될 수 있지만, `TickComponent()` 함수를 사용하여 업데이트 된다.


## [[라이프사이클(Life Cycle)]]


## [[리플리케이션(Replication)]]


## Destroying Actors(액터 소멸하기)

World Object가 Actor References 리스트를 보유하므로 Actor는 일반적으로 Garbage Collected되지 않는다.
Actor는 `Destroy()`를 호출하여 명시적으로 소멸시킬 수 있다. 그러면 Actor가 레벨에서 제거되고 'pending kill'로 표시된다. 즉, 남아 있다가 다음 Garbage Collection 시 Clean up 된다.