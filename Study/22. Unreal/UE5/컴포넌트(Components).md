컴포넌트(Components)는 액터(Actors)가 자기 자신에 서브 Object로 attach할 수 있는 특수한 타입의 Object이다.
Components는 비주얼 표현 디스플레이 또는 사운드 재생 기능과 같은 일반적인 동작을 공유할 때 유용하다. 또한 Components는 Vehicle이 입력을 인식하고 속도 및 방향을 변경하는 방식과 같은 프로젝트 별 콘셉트를 표현할 수도 있다.
예를 들어, 사용자가 컨트롤 가능한 자동차, 항공기, 배가 있는 프로젝트의 경우 Vehicle Actor가 사용하는 Components의 종류를 변경함으로써 Vehicle Control 및 Movement의 차이를 구현할 수 있다.

## 액터 컴포넌트(Actor Components)

`UActorComponent`는 모든 컴포넌트에 대한 **베이스 클래스**이다.
Components는 Mesh와 이미지를 렌더링하고, Collision을 구현하며, 오디오를 재생할 수 있는 유일한 방법이기 때문에, 플레이어가 게임을 플레이할 때 월드에서 보거나 상호작용하는 모든 것은 궁극적으로 여러 타입의 컴포넌트 작업으로 이루어진다.

개성 있는 Components를 생성할 때 이해해야 할 주요 클래스는 액터 컴포넌트(Actor Components), 씬 컴포넌트(Scene Components), 프리미티브 컴포넌트(Primitive Components)이다.
- **액터 컴포넌트**(`UActorComponent`)는 **움직임, 인벤토리, Attribute 관리 및 기타 비물리적 개념과 같은 추상적인 동작** 대부분에 유용하다. 액터 컴포넌트에는 Transform이 없으며, 이는 액터 컴포넌트의 경우 World 내 물리적 위치 또는 회전이 없다는 것을 의미한다.
- **씬 컴포넌트**(`USceneComponent`, `UActorComponent`의 자손)는 **Geometry 표현이 필요하지 않은 위치 기반 동작을 지원**한다. 여기에는 spring arms, cameras, physical forces, constraints(physical objects는 해당 없음), 심지어 오디오도 포함된다.
- **프리미티브 컴포넌트**(`UPrimitiveComponent`, `USceneComponent`의 자손)는 **Geometry 표현이 있는 씬 컴포넌트**로, 주로 비주얼 요소 렌더링이나 physical objects와의 collide 또는 overlap에 사용된다. 여기에는 box, capsule, sphere collision volumes뿐만 아니라 static mesh 나 skeletal mesh, sprites 나 billboards, particle systems도 포함된다.

## 컴포넌트 등록

액터 컴포넌트가 각 프레임을 업데이트하고 Scene에 영향을 미치도록 하려면, 엔진이 액터 컴포넌트를 등록(register)해야 한다.
등록은 Actor spawn 과정에서 해당 Actor의 서브 오브젝트로 생성된 컴포넌트에 대해 자동으로 이루어진다. 그러나 플레이 과정에서 생성된 컴포넌트의 경우 수동 등록이 가능하다.
`RegisterComponent` 함수가 이러한 기능을 제공한다. 단 해당 컴포넌트가 Actor와 연관되어 있어야 한다.

> [!warning]
> 플레이 도중 컴포넌트를 등록하면 성능에 영향을 미칠 수 있으므로 꼭 필요한 경우에만 등록해야 한다.

### 등록 이벤트
컴포넌트 등록 과정에서 엔진은 컴포넌트와 Scene을 연관시켜 프레임 당 업데이트를 가능하게 할 뿐만 아니라, 다음 `UActorComponent` 함수를 실행한다.

| 함수                     | 설명                                         |
| ---------------------- | ------------------------------------------ |
| `OnRegister`           | 컴포넌트를 등록할 때 이 함수를 override하여 코드를 추가할 수 있다. |
| `CreateRenderState`    | 컴포넌트의 [[#렌더 상태(Render State)]]를 초기화 한다.    |
| `OnCreatePhysicsState` | 컴포넌트의 [[#물리 상태(Physics State)]]를 초기화 한다.   |

## 컴포넌트 등록 해제

업데이트, 시뮬레이션 또는 렌더링 프로세스에서 액터 컴포넌트를 제거하려면 `UnregisterComponent` 함수로 등록을 해제하면 된다.

### 등록 해제 이벤트
아래의 `UActorComponent` 함수는 컴포넌트 등록 해제 시 실행된다.

| 함수                      | 설명                                                |
| ----------------------- | ------------------------------------------------- |
| `OnUnregister`          | 컴포넌트를 등록 해제할 때 이 함수를 override하여 코드를 추가할 수 있다.     |
| `DestroyRenderState`    | 컴포넌트의 [[#렌더 상태(Render State)]]를 초기화하지 않은 채로 둔다.   |
| `OnDestroyPhysicsState` | 컴포넌트의 [[#물리 상태(Physics State)]]를 초기화하지 않은 상태로 둔다. |

## 업데이트하기

액터 컴포넌트에는 Actor와 유사한 방식으로 **각 프레임을 업데이트하는 기능**이 있다.
`TickComponent` 함수로 컴포넌트가 각 프레임에서 코드를 실행하도록 할 수 있다. 예를 들어, **USkeletalMeshComponent**는 자체 `TickComponent` 함수를 사용하여 애니메이션 및 스켈레탈 컨트롤러를 업데이트하는 반면, UParticleSystemComponent는 emitter를 업데이트하고 particle event를 처리한다.

기본적으로 **액터 컴포넌트는 업데이트하지 않도록 설정되어 있다**.
액터 컴포넌트가 각 프레임을 업데이트하도록 하려면, **생성자에서 `PrimaryComponentTick.bCanEverTick`을 `true`로 세팅하여 Tick을 활성화**해야 한다. 그런 다음 생성자 또는 다른 위치에서 **`PrimaryComponentTick.SetTickFunctionEnable(true)`을 호출하여 업데이트를 활성화**해야 한다. 추후 `PrimaryComponentTick.SetTickFunctionEnable(false)`을 호출하여 Tick을 비활성화할 수 있다.
향후 컴포넌트에 업데이트가 필요하지 않거나, (소유 액터 클래스 등으로부터) 수동으로 업데이트 함수를 호출할 계획인 경우 `PrimaryComponentTick.bCanEverTick`을 default 값인 `false`로 두고 퍼포먼스를 약간 향상시켜도 좋다.

## 렌더 상태(Render State)

렌더링을 하려면 액터 컴포넌트가 렌더 상태를 생성해야 한다. 또한 렌더 상태는 렌더 데이터를 업데이트해야 하는 컴포넌트에 대해 무언가 변경되었다는 사실을 엔진에 알린다. 이러한 변경이 발생하면 렌더 상태는 'dirty'로 표시된다.
자신만의 컴포넌트를 빌드하는 경우 `MarkRenderStateDirty` 함수를 사용하여 렌더 데이터를 더티로 표시할 수 있다.
프레임의 끝에서 모든 더티 컴포넌트는 엔진에서 렌더 데이터를 업데이트한다.
씬 컴포넌트(프리미티브 컴포넌트 포함)는 기본적으로 렌더 상태를 생성하는 반면, 액터 컴포넌트는 그렇지 않다.

## 물리 상태(Physics State)