## double의 부동소수점 문제
오늘날 컴퓨터는 대부분 IEEE 754 부동 소수점 방식을 사용하기 때문에 java에서 double을 사용해 소수를 표현한다면 소수점 약 15자리부터 오차가 발생한다.

## 오차 해결
자바에는 BigDecimal이라는 자료형이 있다. 이를 이용해 정확한 소수를 표현할 수 있다.

## BigDecimal은 어떻게 정확하게 표시할 수 있을까?

BigDecimal은 정수를 사용하여 소수를 다루기 때문에 정확한 표현을 할 수 있게 된다.

BigDecimal이 소수를 표현하기 위한 상태 값들은 다음과 같다.
- `BigInteger intVal`: 정수부 값
- `int precision`: 정수부, 소수부의 길이를 각각 합한 값
- `int scale`: 소수부의 길이
- `String stringCache`: 숫자를 String으로 변환한 값
- `long intCompact`: 소수점을 제외한 전체 수

### BigDecimal은 valueOf를 통해 생성하자.
BigDecimal을 사용할 때 생성자를 통한 객체 생성은 권장하지 않는다.

다음 코드를 살펴보자.
```java
double number = 1.0;
BigDecimal bigDecimal = new BigDecimal(number); // 0.100000000000000005551115123...
```
이 경우 double에서 이미 오차가 발생한 상황이다. 그렇기 때문에 BigDecimal에 저장된 값도 오차가 있는 값이 들어가게 된다.

하지만 다음과 같이 valueOf를 통해 생성하면 해결할 수 있다.
valueOf는 내부에서 문자열로 변환시키는 로직을 수행하기 때문에 문제가 되지 않는다.
```java
double number = 1.0;
BigDecimal bigDecimal = BigDecimal.valueOf(number);
```

```java
public static BigDecimal valueOf(double val) {  
    // Reminder: a zero double returns '0.0', so we cannot fastpath
    // to use the constant ZERO.  This might be important enough to
    // justify a factory approach, a cache, or a few private
    // constants, later.
    return new BigDecimal(Double.toString(val));  
}
```
위 코드에서 알 수 있듯, 직접 문자열로 변환하면 생성자를 사용해서 생성해도 위 문제를 해결할 수 있다.
```java
double number = 1.0;
BigDecimal bigDecimal = new BigDecimal(Double.toString(number));
```

> [!TIP]
> 이 생성자의 결과는 다소 예측하기 어려울 수 있습니다.  
> Java에서 BigDecimal(0.1)을 새로 작성하면 0.1(스케일이 1인 스케일 없는 값)과 정확히 같은 BigDecimal이 생성될 것이라고 생각할 수 있지만 실제로는 0.1000000000000000055511151231257827021181583404541015625와 동일합니다. 이는 0.1을 정확히 2진수(또는 유한 길이의 2진수 분수)로 표현할 수 없기 때문입니다.  
> 따라서 생성자에 전달되는 값은 겉보기와는 달리 0.1과 정확히 같지 않습니다.

> [!TIP]
> 반면에 문자열 생성자는 완벽하게 예측할 수 있습니다.  
> 새로운 BigDecimal("0.1")을 작성하면 예상대로 0.1과 정확히 같은 BigDecimal이 생성됩니다.  
> 따라서 일반적으로 이 생성자보다는 문자열 생성자를 사용하는 것이 좋습니다.

> [!TIP]
> 이 생성자는 정확한 변환을 제공하므로 Double.toString(double) 메서드를 사용하여 더블을 문자열로 변환한 다음 BigDecimal(String) 생성자를 사용하는 것과 동일한 결과를 제공하지 않는다는 점에 유의하세요.  
> 이러한 결과를 얻으려면 정적 valueOf(double) 메서드를 사용하십시오.

## BigDecimal의 단점
BigDecimal은 사용하기 번거롭다는 점이 있다.
만약 ORM을 쓰고 있다면 double보다 사용하기 꽤 불편할 것이다. 또한 double보다 오버헤드가 다소 있다는 문제가 있다.

만약 돈과 관련된 것이라면 정확성을 위해 BigDecimal의 사용은 필수적일 수 있다.

## 위경도에 어떤 자료형을 쓸까

만약 개발 시 지도 api를 활용한 서비스를 준비한다면 위경도 값을 api를 통해 받을 것이다.
대부분의 지도 API(Google Map, 네이버 지도, 카카오맵 등)은 WGS84 위도/경도 좌표계를 사용한다.

그럼 api를 받을 때 다음과 같은 형식으로 받을 것이다.
```json
{ latitude: 37.51243121154322, longitude: 127.03763421212431}
```

여기서 소수점 오차를 얼마나 허용할 수 있을까?

5자리까지는 1m, 6자리는 10cm, 7자리는 1cm의 차이가 있다.
만약 엄청 정밀한 위치 표시를 요구하는게 아니라면 double을 사용해서 좌표를 표시해도 무리가 없어보인다.
![[Pasted image 20240430010822.png]]

만약 위치, 지도를 사용하는 서비스를 개발 중이라면 위/경도의 자료형을 한 번 고민을 해보는 것이 좋을 것 같다.