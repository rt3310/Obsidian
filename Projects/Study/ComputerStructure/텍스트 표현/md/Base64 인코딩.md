QP 인코딩은 1바이트를 표현하기 위해 3바이트를 사용하기 때문에 아주 비효율적이다.

Base64 인코딩은 3바이트 데이터를 4문자로 표현한다.
3바이트 데이터의 **24비트를 네 가지 6비트 덩어리로 나누고, 각 덩어리의 6비트 값에 출력 가능한 문자를 할당해 표현**한다. -> 7비트 안에 들어간다

이 인코딩은 모든 3바이트 조합을 4바이트 조합으로 변환할 수 있다. 하지만 원본 데이터 길이가 3바이트의 배수라는 보장은 없다.

패딩 문자를 도입해 이런 문제를 해결한다.
원본 데이터가 2바이트 남으면 끝에 =를 붙이고, 1바이트 남으면 끝에 ==를 붙인다.

Base64는 XML, JSON, REST API등 문자열 기반 데이터를 주고받는 환경에서 multi-form을 다룰경우 함께 사용된다.