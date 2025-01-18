`RotateTowards()` 메서드는 현재 쿼터니언과 목표 쿼터니언을 향해 회전하는 쿼터니언을 생성한다.

```c#
public static Quaternion RotateTowards(Quaternion from, Quaternion to, float maxDegreesDelta);
```
- from: 현재 쿼터니언
- to: 목표 쿼터니언
- maxDegreesDelta: 회전 속도

## 예시

```c#
private void Update() 
{
    Vector3 direction = (target.transform.position - transform.position).normalized;
    Quaternion rotation = Quaternion.LookRotation(direction);
    Quaternion rotateValue = Quaternion.RotateTowards(transform.rotation, rotation, 60.0f * Time.deltaTime);
    
    trasform.rotation = rotateValue;
}
```
LookAt과 LookRotation 메서드와는 다르게 즉시 회전하는 것이 아닌 일정한 속도로 회전한다.
Time.deltaTime 옆의 상수값 (60.0f)를 조절하여 회전속도를 조절할 수 있다.