Actor가 어떻게 생성되든 소멸 경로는 모두 같다.

![[Pasted image 20251020191744.png]]

## 디스크에서 로드(Load from Disk)

디스크에서 로드(Load from Disk) 경로는 `UEngine::LoadMap`이 발생할 때나 레벨 스트리밍에서 `UWorld::AddToWorld`를 호출하는 경우처럼 이미 레벨에 있는 Actor에 대해서 발생한다.
1. 패키지/레벨에 있는 Actor가 Disk에서 로드된다.
2. 디스크에서의 로드 완료 후 Serialized된 Actor에서 PostLoad를 호출한다.
	- 커스텀 버전 관리와 fix-up 행동이 여기서 이루어져야 한다.
	- PostLoad는 PostActorCreated와 상호 배타적이다.
3. 월드에서 `UAISystemBase::InitializeActorsForPlay`를 호출하여 Actor를 준비하고 게임플레이를 시작한다.
4. 레벨에서 Serialized되지 않은 Actor와 Seamless Travel carry-over에 대해 `ULevel::RouteActorInitialize`를 호출한다.
	1. Actor의 컴포넌트에서 InitializeComponent를 호출하기 전에 `AActor::PreInitializeComponents`를 호출한다.
	2. `UActorComponent::InitializeComponent`는 Actor에 정의된 각 컴포넌트를 생성하는 헬퍼 함수이다.
	3. `AActor::PostInitializeComponents`는 Actor의 컴포넌트가 초기화된 후에 호출된다.
5. 레벨이 시작될 때 `AActor::BeginPlay`가 호출된다.

## 에디터에서 플레이(Play in Editor)

에디터에서 플레이(Play in Editor) 경로에서는 Actor를 디스크에서 로드하는 대신 에디터에서 복사한다. 그런 다음 복사된 Actor는 디스크에서 로드 경로에 설명된 흐름과 유사하게 초기화된다.
1. 에디터의 Actor가 새 월드에 복제된다.
2. `U::Object::PostDuplicate`를 호출한다.
3. `UAISystemBase::InitializeActorsForPlay`
4. 초기화되지 않은 Actor에 대해 `ULevel::RouteActorInitialize`를 호출하고 모든 seamless travel 전환 처리를 한다.
	1. Actor의 컴포넌트에서 InitializeComponent를 호출하기 전에 `AActor::PreInitializeComponents`를 호출한다.
		1. `UActorComponent::InitializeComponent`는 Actor에 정의된 각 컴포넌트를 생성하는 헬퍼 함수이다.
		2. `AActor::PostInitializeComponents`는 Actor의 컴포넌트가 초기화 된 후에 호출된다.
5. 레벨이 시작될 때 `AActor::BeginPlay`가 호출된다.

## 스폰(Spawning)

Actor 인스턴스를 스폰할 때 따라가는 경로는 다음과 같다.
1. `UWorld::SpawnActor`를 호출한다.
2. Actor가 월드에 스폰된 이후 `AActor::PostSpawnInitialize`가 호출된다.
3. `AActor::PostActorCreated`는 생성 이후 스폰된 Actor에 대해 호출되며, 모든 생성자 구현 동작은 여기로 이동해야 한다. PostActorCreated는 PostLoad와 상호 배타적이다.
4. `AActor::ExecuteConstruction`
5. `AActor::OnConstruction`: 액터 생성, 블루프린트 Actor의 컴포넌트 생성 및 블루프린트 변수가 초기화된다.
6. `AActor::PostActorConstruction`
	1. Actor의 컴포넌트에서 InitializeComponent를 호출하기 전에 `AActor::PreInitializeComponents`를 호출한다.
		1. `UActorComponent::InitializeComponent`는 Actor에 정의된 각 컴포넌트를 생성하는 헬퍼 함수이다.
		2. `AActor::PostInitializeComponents`는 Actor의 컴포넌트가 초기화 된 후에 호출된다.
7. `UWorld::OnActorSpawned`가 UWorld에서 Broadcast 된다.
8. `AActor::BeginPlay`가 호출된다.

## 디퍼드 스폰(Deferred Spawn)

'스폰 시 노출(Expose on Spawn)'로 설정된 Property가 있으면 Actor는 디퍼드 스폰(Deferred Spawn)될 수 있다.
1. `UWorld::SpawnActorDeferred`는 Procedural Actor 스폰을 위한 것으로, 블루프린트 Construction Script 전에 추가적인 구성을 할 수 있다.
2. SpawnActor 안에서 모든 일이 발생하지만, `AActor::PostActorCreated` 후에 다음과 같은 일이 일어난다.
	1. 유효하지만 불완전한 Actor 인스턴스로 다양한 '초기화 함수'를 구성하고 호출한다.
	2. `AActor::FinishSpawning`이 Actor를 마무리하기 위해 호출되며, SpawnActor 줄의 `AActor::ExecuteConstruction`에서 선택된다.

## 액터 라이프사이클 종료(End of Actor Lifecycle)

Actor를 소멸하는 방법은 여러가지가 있지만, 월드에서 제거하는 방법은 동일하다.
게임플레이 도중 다음 함수를 호출할 수 있지만, 플레이 도중 실제로 소멸되지 않는 Actor가 많으므로 완전히 선택사항이다.
- Actor를 제거해야 하지만 게임플레이가 계속 진행 중일 때마다 게임에서 `AActor::Destroy`를 수동으로 호출한다. Actor는 pending kill로 표시되고 Level의 Actor 배열에서 제거된다.
- Actor의 수명이 다해가는 것을 보장하기 위해 `AActor::EndPlay`가 여러 곳에서 호출된다. 플레이 도중 Actor가 포함된 스트리밍 레벨이 Unload되면 이 메서드와 Level Transition이 호출된다.
- EndPlay가 호출되는 경우는 다음과 같다.
	- Destroy에 대한 명시적 호출
	- 에디터에서의 플레이가 종료된 경우
	- Level Transition(Seamless Travel 또는 Load Map)
	- Actor가 포함된 Streaming Level이 Unload될 때
	- Actor의 수명이 만료됨
	- 애플리케이션 종료(모든 Actor가 소멸됨)

발생 과정과 상관없이, Actor는 `RF_PendingKill`로 표시되어 다음 Garbage Collection 주기 동안 UE가 메모리에서 할당 해제한다. 또한, Pending Kill 을 수동으로 확인하는 대신 `FWeakObjectPtr<AActor>`를 사용하는 것이 더 깔끔하다.

> [!warning]
> Actor는 EndPlay가 호출될 때 반드시 소멸되지 않을 수 있다. 예를 들어 `s.ForceGCAfterLevelStreamedOut`이 `false`이고 서브레벨이 빠르게 리로드되면 액터의 EndPlay가 호출되지만, Actor가  "부활"되어 기본값으로 초기화되지 않은 로컬 변수와 함께 이전에 존재했던 것과 똑같은 Actor가 될 수 있다.
