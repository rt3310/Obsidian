## Time.time

`Time.time`은 애플리케이션이 실행 된 시간이며 읽기 전용(read only)이다.

응용 프로그램은 각 프레임의 시작 부분에 현재 Time.time을 수신하며 값은 프레임 당 증가한다.
중요한 점은 Time.time은 프레임 당 시간이 아니라 애플리케이션이 실행된 시간을 제공하기 위한 것이다.

## Time.deltaTime

`Time.deltaTime`은 현재 프레임과 이전 프레임 사이의 시간을 제공한다.
이 값은 float 형태로 반환하며, 단위는 동일하게 초를 사용한다.

예시로, A의 컴퓨터는 빨라서 Update가 더 빨리 실행이 되고, B의 컴퓨터는 느려서 Update 함수 호출이 더 적다고 가정해보자.
Update에 캐릭터 이동거리를 정의해 놓았을 때, 이렇게 된다면 당연히 Update가 더 많이 호출되는 곳이 이동거리가 더 많을 수 밖에 없다.
```c#
public class ExampleClasss : MonoBehaviour {
	void Update() {
		float translation = Time.deltaTime * 10;
		transform.Translate(0, 0, translation);
	}
}
```
`Time.deltaTime`을 사용해서 1frame 당 이동거리를 계산하여 실행한다. 위와 같이 작성한다면 Update가 몇번이 수행이 되어도 같은 결과를 낼 수 있다.

Time.deltaTime * 10을 Update에 정의했고, 30 frame과 60 frame 컴퓨터가 있다고 가정해보자.
30 frame에는 Time.deltaTime에 1/30의 값이 전해지고 60 frame에는 1/60이 전달된다.
그렇기에 1초가 지났을 때 결국 Time.deltaTime에 전해진 값들을 모두 다 더하면 1이 된다.

## 결론

`Time.time` 은 애플리케이션이 실행된 후 지난 시간
`Time.deltaTime`은 이전 프레임과 현재 프레임 간의 시간