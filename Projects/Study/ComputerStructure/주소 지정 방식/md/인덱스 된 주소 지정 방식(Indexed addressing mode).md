- 유휴주소를 계산하기 위해 인덱스 레지스터의 내용에 명령어 주소필드 값을 더하는 방식
- 유휴주소 = 명령어 operand + 인덱스 레지스터
- 특징: 배열을 인덱싱할 때 많이 사용

[예시]
> LDA ADRS(R1); AC <- M[ADRS + R1]

기억장치의 ADRS + R1 번지에 있는 값을 AC에 적재한다.

> [!NOTE]
> 인덱스 된 주소 지정방식의 변형으로 베이스 레지스터 주소 지정방식(Base-register mode)가 있다. 이 방식에서는 베이스 레지스터의 내용에 명령어 주소 필드 값을 더하여 유휴주소를 계산한다. 이 방식은 인덱스 레지스터 대신 베이스 레지스터가 사용된다.
> 인덱스 레지스터가 명령어 주소필드에 대한 상대적인 위치 값을 갖는 반면 베이스 레지스터는 명령어 주소필드로부터 상대적인 변위값을 갖는다.