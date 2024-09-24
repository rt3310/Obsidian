## Object & Reference

- Objects: 데이터와 기능의 집합
- Actors: 레벨에 투입할 수 있는 Object
- Component: Actor에 투입할 수 있는 Objects
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

## Glossary
- Spawning: Creating an object while playing
- Transform: Location, rotation and scale