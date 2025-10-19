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

### 컴포넌트 등록
액터 컴포넌트가 각 프레임을 업데이트하고 Scene에 영향을 미치도록 하려면, 엔진이 액터 컴포넌트를 등록(register)해야 한다.
등록은 Actor spawn 과정에서 해당 Actor의 서브 오브젝트로 생성된 컴포넌트에 대해 자동으로 이루어진다. 그러나 플레이 과정에서 생성된 컴포넌트의 경우 수동 등록이 가능하다.
`RegisterComponent` 함수가 이러한 기능을 제공한다. 단 해당 컴포넌트가 Actor와 연관되어 있어야 한다.

> [!warning]
> 플레이 도중 컴포넌트를 등록하면 성능에 영향을 미칠 수 있으므로 꼭 필요한 경우에만 등록해야 한다.

#### 등록 이벤트
컴포넌트 등록 과정에서 엔진은 컴포넌트와 Scene을 연관시켜 프레임 당 업데이트를 가능하게 할 뿐만 아니라, 다음 `UActorComponent` 함수를 실행한다.

| 함수                     | 설명                                         |
| ---------------------- | ------------------------------------------ |
| `OnRegister`           | 컴포넌트를 등록할 때 이 함수를 override하여 코드를 추가할 수 있다. |
| `CreateRenderState`    | 컴포넌트의 [[#렌더 상태(Render State)]]를 초기화 한다.    |
| `OnCreatePhysicsState` | 컴포넌트의 [[#물리 상태(Physics State)]]를 초기화 한다.   |

### 컴포넌트 등록 해제
업데이트, 시뮬레이션 또는 렌더링 프로세스에서 액터 컴포넌트를 제거하려면 `UnregisterComponent` 함수로 등록을 해제하면 된다.

#### 등록 해제 이벤트
아래의 `UActorComponent` 함수는 컴포넌트 등록 해제 시 실행된다.

| 함수                      | 설명                                                |
| ----------------------- | ------------------------------------------------- |
| `OnUnregister`          | 컴포넌트를 등록 해제할 때 이 함수를 override하여 코드를 추가할 수 있다.     |
| `DestroyRenderState`    | 컴포넌트의 [[#렌더 상태(Render State)]]를 초기화하지 않은 채로 둔다.   |
| `OnDestroyPhysicsState` | 컴포넌트의 [[#물리 상태(Physics State)]]를 초기화하지 않은 상태로 둔다. |

### 업데이트하기
액터 컴포넌트에는 Actor와 유사한 방식으로 **각 프레임을 업데이트하는 기능**이 있다.
`TickComponent` 함수로 컴포넌트가 각 프레임에서 코드를 실행하도록 할 수 있다. 예를 들어, **USkeletalMeshComponent**는 자체 `TickComponent` 함수를 사용하여 애니메이션 및 스켈레탈 컨트롤러를 업데이트하는 반면, UParticleSystemComponent는 emitter를 업데이트하고 particle event를 처리한다.

기본적으로 **액터 컴포넌트는 업데이트하지 않도록 설정되어 있다**.
액터 컴포넌트가 각 프레임을 업데이트하도록 하려면, **생성자에서 `PrimaryComponentTick.bCanEverTick`을 `true`로 세팅하여 Tick을 활성화**해야 한다. 그런 다음 생성자 또는 다른 위치에서 **`PrimaryComponentTick.SetTickFunctionEnable(true)`을 호출하여 업데이트를 활성화**해야 한다. 추후 `PrimaryComponentTick.SetTickFunctionEnable(false)`을 호출하여 Tick을 비활성화할 수 있다.
향후 컴포넌트에 업데이트가 필요하지 않거나, (소유 액터 클래스 등으로부터) 수동으로 업데이트 함수를 호출할 계획인 경우 `PrimaryComponentTick.bCanEverTick`을 default 값인 `false`로 두고 퍼포먼스를 약간 향상시켜도 좋다.

### 렌더 상태(Render State)
렌더링을 하려면 **액터 컴포넌트가 Render State를 생성해야 한다**. 또한 Render State는 **렌더 데이터를 업데이트해야 하는 컴포넌트에 대해 무언가 변경되었다는 사실을 엔진에 알린다**. 이러한 변경이 발생하면 Render State는 '**dirty**'로 표시된다.
자신만의 컴포넌트를 빌드하는 경우 `MarkRenderStateDirty` 함수를 사용하여 렌더 데이터를 dirty로 표시할 수 있다.
**프레임의 끝에서 모든 더티 컴포넌트는 엔진에서 렌더 데이터를 업데이트**한다.
씬 컴포넌트(프리미티브 컴포넌트 포함)는 기본적으로 Render State를 생성하는 반면, 액터 컴포넌트는 그렇지 않다.


### 물리 상태(Physics State)
엔진의 Physics Simulation 시스템과 상호작용하려면 액터 컴포넌트에는 Physics State가 필요하다.
Physics State는 변경이 발생하는 즉시 업데이트하여 '프레임 뒤처짐(frame-behind)' 아티팩트와 같은 문제를 방지하고 'dirty'를 표시할 필요성을 제거해준다.
기본적으로 액터 컴포넌트 및 씬 컴포넌트에는 Physics State가 없지만, 프리미티브 컴포넌트에는 Physics State가 있다. 컴포넌트 클래스의 인스턴스에 Physics State가 필요한지 여부를 결정하려면 `ShouldCreatePhysicsState` 함수를 override하라.

> [!warning]
> 클래스가 Physics를 사용할 경우, 단순히 `true`를 반환하는 것은 추천하지 않는다.
> 컴포넌트 디스트럭션 도중과 같이 Physics State를 생성하면 안되는 상황에 대해 파악하려면 함수의 `UPrimitiveComponent` 버전을 확인해 보라. 또한 일반적으로 `true`를 반환하는 사례에서는 `Super::ShouldCreatePhysicsState`를 반환할 수도 있다.

### 시각화 컴포넌트(Visualization Components)
일부 Actor 및 Component에는 비주얼 표현이 없어 선택하기 어렵거나, 보이지 않는 중요한 Property가 있다. 개발자는 에디터에서 작업하는 동안 추가 컴포넌트를 더하여 정보를 표시할 수 있지만, 이러한 추가 컴포넌트는 에디터에서 플레이(Play In Editor, PIE) 도중이나 패키지 빌드를 실행할 때는 필요하지 않다.
에디터는 이러한 문제를 해결하기 위해 에디터에서 작업할 때만 존재하는 보통의 컴포넌트인 시각화 컴포넌트(Visualization Components)를 지원한다.

시각화 컴포넌트를 만들려면, **일반 컴포넌트를 생성한 후 그 컴포넌트에서 `SetIsVisualizationComponent`를 호출**한다. 컴포넌트가 에디터 바깥에 존재할 필요가 없기 때문에, 컴포넌트에 대한 모든 레퍼런스는 **`WITH_EDITORONLY_DATA` 또는 `WITH_EDITOR`로 확인되는 PreProcessor 체크에 포함되어야 한다**. 이렇게 하면 패키지 빌드가 이러한 컴포넌트에 영향을 받거나 코드 내에서 참조하지 않는다. 예를 들어, 카메라 컴포넌트(Camera Component) [^1]프러스텀(Frustum)을 보여주는 프러스텀 컴포넌트 그리기(Draw Frustum Component)와 같이 에디터에서 유용한 정보를 표시하는 몇 가지 다른 컴포넌트를 사용한다.

헤더 파일에서 프러스텀 컴포넌트 그리기는 클래스 내에서 다음과 같이 정의된다.
```cpp
#if WITH_EDITORONLY_DATA
	// 카메라 필드 오브 뷰의 위치를 시각적으로 보여주는 데 사용되는 Frustum Component
	class UDrawFrustumComponent* DrawFrustum;
	// ...
```

유사하게, 이 컴포넌트에 대한 모든 레퍼런스는 소스 파일 내 `WITH_EDITORONLY_DATA`로 확인되는 preprocessor 체크에 포함된다. `OnRegister` 내 `WITH_EDITORONLY_DATA` 체크에 포함되는 이 코드는 카메라 컴포넌트가 유효한 Actor에 Attach 되어 있는지 여부를 확인한 후, 다음과 같이 프러스텀 컴포넌트 그리기 코드를 추가한다.
```cpp
void UCameraComponent::OnRegister()
{
#if WITH_EDITORONLY_DATA
	if (AActor* MyOwner = GetOwner())
	{
		// ...
		if (DrawFrustum == nullptr)
		{
			DrawFrustum = NewObject<UDrawFrustumComponent>(MyOwner, NAME_None, RF_Transactional | RF_TextExportTransient);
			DrawFrustum->SetupAttachment(this);
			DrawFrustum->SetIsVisualizationComponent(true);
			// ...
		}
	}
	// ...
#endif
	Super::OnRegister();
	// ... 추가 코드(모든 빌드에서 실행)는 여기에 삽입
}
```
`DrawFrustum`은 이제 에디터에만 존재하며 시각화 컴포넌트로 간주되므로 에디터 내 플레이 테스트에서는 나타나지 않는다.

## 씬 컴포넌트(Scene Components)

썬 컴포넌트는 월드 내 특정한 물리적 위치에 존재하는 액터 컴포넌트이다. 이 위치는 트랜스폼(Transform, `FTransform`)으로 정의되며, 여기에는 컴포넌트의 Position, Rotation, Scale이 포함된다.
씬 컴포넌트에는 서로 Attach되어 트리를 형성하는 기능이 있으며, Actor가 단일 씬 컴포넌트를 'root'로 지정할 수 있다. 이는 Actor의 World Position, Rotation, Scale이 해당 루트 컴포넌트에서 그려짐을 의미한다.

### 어태치먼트(Attachment)
오직 씬 컴포넌트(`USceneComponent` 및 그 자손 클래스)만이 서로 attach될 수 있다. 이는 자손 및 부모 컴포넌트 간의 공간적 관계를 설명하는 Transform에 대한 필요성 때문이다.
씬 컴포넌트의 자손 수에는 제한이 없는 반면, 부모는 단 하나만 가질 수 있다. 그렇지 않을 경우 월드에서 직접 배치된다.
씬 컴포넌트 시스템은 Attachment Cycle을 지원하지 않는다.
주요 방법은 2가지로, 생성자에게, 그리고 아직 등록되지 않은 컴포넌트 처리 시 유용한 `SetupAttachment`와 하나의 씬 컴포넌트를 다른 씬 컴포넌트에 즉시 attach하며 플레이 도중에 유용한 `AttachToComponent`가 있다. 또한 이러한 Attachment 시스템은 한 Actor의 루트 컴포넌트를 다른 Actor에 속한 컴포넌트에 attach함으로써 Actor를 서로 attach할 수 있다.

## 프리미티브 컴포넌트(Primitive Components)

프리미티브 컴포넌트(`UPrimitiveComponent`)는 일종의 Geometry를 포함하거나 생성하는 씬 컴포넌트로, **주된 용도는 rendering이나 collision**이다.
다양한 Geometry 타입을 위한 여러가지 서브클래스가 있지만, 지금까지 가장 흔한 것은 박스 컴포넌트(Box Component), 캡슐 컴포넌트(Capsule Component), 스태틱 메시 컴포넌트(Static Mesh Component) 및 스켈레탈 메시 컴포넌트(Skeletal Mesh Component)이다.
박스 컴포넌트와 캡슐 컴포넌트는 Collision 감지용으로 보이지 않는 Geometry를 생성하는 반면, 스태틱 메시 컴포넌트와 스켈레탈 메시 컴포넌트에는 렌더링 된 사전 빌드 Geometry가 포함되어 있으며, 원한다면 Collision 감지에도 사용할 수 있다.

### 씬 프록시(Scene Proxy)
프리미티브 컴포넌트의 씬 프록시(Scene Proxy, `FPrimitiveSceneProxy`)는 엔진이 게임 스레드와 병행하여 컴포넌트를 렌더링할 때 사용하는 씬 데이터를 캡슐화한다.
각 타입의 프리미티브에는 필요한 특정 렌더 데이터를 담는 자체 씬 프록시 자손 클래스가 있다.

[^1]: 절두체
