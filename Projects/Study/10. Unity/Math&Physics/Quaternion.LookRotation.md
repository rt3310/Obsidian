```c#
public static Quaternion Quaternion.LookRotation(Vector3 forward);
```
```c#
public static Quaternion Quaternion.LookRotation(Vector3 forward, [DefaultValue("Vector3.up")] Vector3 upwards);
```

해당 벡터 방향을 바라보는 회전 상태를 반환한다.
반환받은 Quaternion을 Transform의 rotation에 넣어주면 된다.

`LookRotation()` 메서드 안에는 상대좌표를 넣어줘야 한다.
예를 들어, 인자로 `new Vector3(1, 1, 1)`을 적어준 경우 큐브가 원점에 있을 경우에는 월드좌표 (1, 1, 1)을 바라보지만, 원점이 아닌 곳에서는 월드좌표 (1, 1, 1)을 바라보지 않는다.
(1, 1, 1)은 상대좌표로 적용되고, 큐브가 (1, 2, 3)에 있을 경우 (2, 3, 4)를 바라보게 된다.

![[Pasted image 20250105123410.png]]
Transform의 rotation에는 Vector 좌표를 직접 넣을 수 없고, 이렇게 Quaternion으로 변환하여 넣어주어야 한다.

`LookRotation()` 메서드의 두 번째 인자에는 머리가 향하는 방향을 지정할 수 있다.
오브젝트의 머리 쪽이 두 번째 인자로 적어준 방향을 향한다. 두 번째 인자를 생략할 경우 자동으로 `Vector3.up`으로 적용된다.

따라서 어느 곳에서 어느 방향을 향해 `LookRotation()` 메서드를 적용하더라도 두 번째 인자가 적혀있지 않으면 오브젝트의 머리쪽은 월드좌표의 위쪽 방향을 향하게 된다.

## Rotate() vs LookRotation()

```c#
tf.Rotate(new Vector(45, 45, 45));
```
와
```c#
tf.rotation = Quaternion.LookRotation(new Vector(45, 45, 45));
```
는 다른 뜻이다.

`Transform.Rotate()` 메서드 안에 들어가는 벡터의 성분은 각도를 가리킨다. (형식은 벡터이지만 각도를 뜻한다)
하지만 `Quaternion.LookRotation()` 메서드 안에 들어가는 벡터는 실제 벡터이고, x, y, z 값을 뜻한다.

따라서 `tf.Rotate(new Vector3(45, 45, 45));` 에서의 45는 각도를 가리키고, `tf.rotation = Quaternion.LookRotation(new Vector3(45, 45, 45));` 에서의 45는 x, y, z 값을 가리킨다.

`tf.rotation = Quaternion.LookRotation(new Vector(45, 45, 45));`와
`tf.rotation = Quaternion.LookRotation(new Vector(1, 1, 1));` 은 같은 결과를 나타낸다.
`Quaternion.LookRotation` 메서드는 벡터에서 힘을 제외하고 방향만들 필요로 하는 메서드이기 때문이다.

## transform.LookAt() vs Quaternion.LookRotation()

`transform.LookAt()` 메서드는 오브젝트를 지정된 위치와 방향으로 회전한다.

```c#
public void LookAt(Transform target);
public void LookAt(Vector3 worldPosition, [DefaultValue("Vector3.up")] Vector3 worldUp);
public void LookAt(Vector3 worldPosition);
```
- target: 바라볼 오브젝트
- worldPosition: 오브젝트가 바라보게 될 위치
- worldUp: 오브젝트가 회전할 기준점

`Quaternion.LookRotation()` 메서드는 지정된 방향을 가리키는 쿼터니언을 생성한다.