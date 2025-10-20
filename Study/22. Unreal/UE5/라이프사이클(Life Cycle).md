Actor가 어떻게 생성되든 소멸 경로는 모두 같다.

![[Pasted image 20251020191744.png]]

## 디스크에서 로드

디스크에서 로드(Load from Disk) 경로는 `UEngine::LoadMap`이 발생할 때나 레벨 스트리밍에서 `UWorld::AddToWorld`를 호출하는 경우처럼 이미 레벨에 있는 Actor에 대해서 발생한다.
1. 패키지/레벨에 있는 Actor가 Disk에서 로드된다.
2. 디스크에서의 로드 완료 후 Serialized된 Actor에서 PostLoad를 호출한다.
	- 커스텀 버전 관리와 fix-up 행동이 여기서 이루어져야 한다.
	- PostLoad는 PostActorCreated와 상호 배타적이다.
3. 월드에서 `UAISystemBase::InitializeActorsForPlay`를 호출하여 Actor를 준비하고 게임플레이를 시작한다.
4. 레벨에서 Serialized되지 않은 Actor와 Seamless Travel carry-over에 대해 `ULevel::RouteActorInitialize`를 호출한다.
	1. Actor의 컴포넌트에서 InitializeComponent를 호출하기 정네 `AActor::PreInitializeComponents`를 호출한다.
	2. `UActorComponent::InitializeComponent`는 Actor에 정의된 각 컴포넌트를 생성하는 헬퍼 함수이다.
	3. `AActor::PostInitializeComponents`는 Actor의 컴포넌트가 초기화된 후에 호출된다.
5. 레벨이 시작될 때 `AActor::BeginPlay`가 호출된다.

## 에디터에서 플레이

에디터에서 플레이(Play in Editor) 경로에서는 Actor를 디스크에서 로드하는 대신 에디터에서 복사한다. 그런 다음 복사된 Actor는 디스크에서 로드 경로에 설명된 흐름과 유사하게 초기화된다.