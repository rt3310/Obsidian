## Static Mesh Component

Mesh는 정점, 모서리, 면의 모음이다.
언리얼 엔진에는 정적 메시(static mesh)와 스켈레탈 메시(skeletal mesh)가 있다.

정적 메시는 간단하며, 의자나 다른 무생물에 정적 메시를 사용할 수 있다.
스켈레탈 메시는 캐릭터와 같은 복잡한 객체를 위해 고안되었으며, 모델을 애니메이션화하는데 사용할 수 있는 상호 연결된 뼈(스켈레톤) 세트가 있다.
## Object & Reference

- Objects: 데이터와 기능의 집합
- Actors: Level에 투입할 수 있는 Object
- Component: Actor에 투입할 수 있는 Objects
	- StaticMeshComponent
- Reference: Object를 찾는 위치
- Data Pin(파란색): 노드의 데이터 입출력
- Execution Pins(흰색): 노드를 언제 실행할지 정함

## Force and Impulse

Force = Mass $\times$ Acceleration
Impulse = Mass $\times$ Velocity Change

### Add Impulse
- Vel Change 체크 시 질량을 무시하고 원하는 속력을 적용
	- Z에 400을 입력하면 위쪽 방향으로 400m/s의 속력을 낸다.
	- 미체크 시에는 질량을 고려해야 하므로 400 * Mass 값을 입력해야 한다.


- Spawning: Creating an object while playing
- Transform: Location, rotation and scale
- Return pin: Output of a node
- Struct: An object that is usually small