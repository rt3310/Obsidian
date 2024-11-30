## 시네머신(Cinemachine)

영화 촬영을 하는 것처럼 Scene을 촬영하여 게임 화면 상에 비추어주는 Unity Package

Cinemachine은 기본적으로 Unity 상의 카메라 오브젝트를 생성하지 않고 가상 카메라를 이용한다.
가상 카메라는 오브젝트를 통해 Unity 상에 생성된 카메라를 다양한 각도에서 비추어 게임 상의 Scene을 보여주도록 도와준다.
또한 가상 카메라는 각각 서로에게 영향을 끼치지 않으며, Unity 카메라에도 영향을 받지 않기 때문에 자유로운 카메라 구성이 가능하다.

### 가상 카메라로 할 수 있는 기능
- 카메라의 포지션 설정
- 카메라의 시선 설정
- 카메라에 노이즈 설정 또는 이펙트나 쉐이크 설정
가상 카메라는 Unity 카메라보다 소모하는 프로세스 등이 적으므로, 더욱 유용하게 활용할 수 있다.

Cinemachine에는 Virtual Camera(가상 카메라), FreeLook Camera, Blend List Camera, State-Driven Camera, ClearShot Camera, Dolly Camera, Target Group Camera, Mixing Camera, 2D Camera가 있는데, 각각 활용도에 따라 다르게 사용할 수 있다.

## Cinemachine Brain

Scene 안에 있는 Cinemachine의 모든 가상 카메라를 모니터링하는 오브젝트로 Unity 카메라에 붙여서 사용한다.
Cinemachine Brain은 실질적으로 게임 화면 상에 비추어 줄 가상 카메라가 무엇인가를 설정하는데, 이때 가장 먼저 엑티브가 되거나 우선순위가 있는 가상 카메라를 비추어준다.

### Cinemachine 종류
- Virtual Camera(가상 카메라)
	- 가장 기본이 되는 카메라로, Unity 카메라를 조정하듯 자유롭게 활용이 가능하다.
- FreeLook Camera
	- 오브젝트를 중심으로 원형의 링을 생성하여 그 구간 안에서 타깃을 관찰하는 카메라
- Blend List Camera
	- 할당된 Virtual Camera들을 정해진 블랜드 방식에 따라 순차적으로 전환하는 카메라
- State-Driven Camera
	- 타깃 에니메이션의 상태별로 활성화/비활성화 시킬 수 있는 카메라
- ClearShot Camera
	- 플레이어의 충돌/트리거 상태에 따라 활성화/비활성화 시킬 수 있는 카메라
- Dolly Camera
	- 트랙을 깔고 그 트랙을 따라 움직이는 카메라
	- Dolly Track과 함께 쓰인다
- Target Group Camera
	- 그룹으로 묶인 카메라들을 자동으로 계산해 한 화면에 보여주는 카메라
- Mixing Camera
	- Child Camera Weight 값에 따라 활성화/비활성화 해주는 카메라
- 2D Camera
	- 직교 뷰로 사용되는 카메라

## Cinemachine Inspector

![[Pasted image 20241130193531.png]]
- Property: 카메라의 우선도
- Save During Play: Unity 상에서 플레이 중에 변경한 시네머신 설정을 저장한다.
- Follow: 어떤 오브젝트를 따라다닐지 설정한다.
- Look At: 어떤 오브젝트를 바라볼지 설정한다.
- Standby Update: Live 상태가 아닌 카메라의 업데이트 빈도 설정
	- Never: 항상
	- Always: Live 일 때만
	- Round Robine: 정기적으로
- Lens: 카메라 렌즈를 설정
- Transitions: 카메라 사이를 이동할 때 Scene 전환 효과 설정
- Body: Scene 내부의 Virtual Camera가 움직일 때 따라가는 알고리즘 설정을 변경한다.
- Aim: Scene 내부의 Virtual Camera가 Look At 타깃을 바라볼 때 따라가는 알고리즘 설정을 변경한다.