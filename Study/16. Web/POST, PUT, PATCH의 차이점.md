토스페이먼츠는 고객에게 읽기 쉽고 간결한 API를 제공하는 목표로 제품을 설계하고 디자인합니다. 그래서 [토스페이먼츠 API](https://docs.tosspayments.com/reference) v1을 디자인할 때도 오래된 클라이언트 및 인프라 모델을 사용하는 고객에서 PATCH, PUT, DELETE 메서드를 사용하기 어렵다는 점을 고려해서 GET, POST 메서드만 제공하기도 했고요.

하지만 다음 세대의 토스페이먼츠 API 디자인을 논의할 시점에서 앞으로는 GET, POST 외 다른 메서드도 사용해서 HTTP 웹 표준을 지키는 디자인을 제공하자는 의견이 모였습니다. 클라이언트 모델을 업데이트한 고객도 많았고, 웹 표준을 지키는 API 설계가 관리하기도, 사용하기도 더 편하기 때문이에요.

HTTP 표준을 사용하면 여러 가지 장점이 있어요. 그중 하나는 API 디자인이 더 간결해진다는 점이에요. 예를 들어, `/{resource-name}/{id}/delete`와 같은 URI에 POST 메서드로 리소스를 삭제하는 대신, HTTP 표준에 따라 `/{resource-name}/{id}` URI에 DELETE 메서드를 사용하면 더 간결하고 이해하기 쉬운 API를 만들 수 있어요. 이렇게 하면 개발자들이 익숙한 표준 방식으로 API를 설계할 수 있어서, 활용하기도 편리하고 이해하기도 쉬워요.

새로운 API 디자인을 논의하면서 HTTP 메서드를 더 깊이 이해할 필요를 느꼈어요. 그래서 POST, PUT, PATCH 메서드의 차이점을 연구해봤는데요. 이번 아티클에서 각 메서드의 특징과 차이점을 [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) 문서를 바탕으로 자세히 알려드릴게요.

## POST

RFC 문서에서는 POST를 다음과 같이 정의하고 있어요.

> The POST method requests that the target resource process the representation enclosed in the request according to the resource's own specific semantics.

이해하기 조금 어려운데요. 한국어로는 "POST 요청의 요청 본문은 타깃 리소스의 정의대로 처리된다"로 번역할 수 있습니다. 쉽게 말하자면 클라이언트가 요청을 보내면 서버에서 모든 걸 알아서 처리해야 된다는 뜻이죠.

위 정의에 의하면 클라이언트는 리소스의 URI를 모르는 상태에서 리소스를 생성할 수 있어요. 예를 들어, 새로운 결제를 생성하기 위해 클라이언트는 `/payments`라는 결제 컬렉션의 URI로 POST 요청을 보냅니다. 그럼 서버는 `/payments` 아래에 `/payments/1`와 같이 새로운 결제 리소스를 만들어요. POST 메서드로 리소스를 생성하면 서버에서 정의한 로직대로 리소스의 URI(및 ID)를 발급합니다.

또 POST의 다른 특징은 [멱등성(idempotentency)](https://docs.tosspayments.com/blog/what-is-idempotency)이 없다는 점입니다. 같은 POST 요청을 여러 번 보내면 여러 개의 리소스가 생성돼요. 예를 들어, 같은 내용의 요청을 `/payments`로 3번 보내면 `/payments/1`, `/payments/2`, `/payments/3`와 같이 똑같은 데이터를 가진 리소스 3개가 생성돼요.

POST 메서드를 떠올리면 주로 리소스 생성을 생각하는데, 사실 POST 메서드는 **서버에서 정의한 다양한 작업**을 모두 처리할 수 있는 메서드에요. 실제로 RFC 문서에도 폼에 입력된 데이터를 불러올 때, 블로그에 메시지를 업로드할 때, 이미 존재하는 리소스에 데이터를 추가할 때 모두 POST 메서드를 사용할 수 있다고 하고요.

## PUT

PUT은 POST와 자주 같이 언급되는 메서드인데요. RFC 문서에 의하면 PUT과 POST의 차이점은 다음과 같습니다.

> The target resource in a POST request is intended to handle the enclosed representation according to the resource's own semantics, whereas the enclosed representation in a PUT request is defined as replacing the state of the target resource.

PUT 요청에서는 클라이언트가 리소스의 정확한 URI를 알아야 됩니다. `/payments`로 PUT 요청을 보내면 `/payments/1`와 같은 리소스가 생성되지 않아요. PUT 메서드를 사용할 때는 `/payments/1`와 같은 특정 리소스의 URI로 요청을 보내야 리소스가 생성돼요. 그래서 PUT 메서드로 리소스를 생성하려면 클라이언트가 URI를 만드는 방법을 알고 있어야 돼요. 또 POST 요청과 달리 PUT 요청은 멱등합니다.

그래서 PUT 메서드는 주로 리소스를 업데이트할 때 사용해요. 리소스 업데이트할 때는 이미 생성된 리소스의 정확한 URI를 알고 있고, 안전하게 멱등한 요청을 보내고 싶기 때문이죠. 그러나 PUT 메서드는 **리소스 전체**를 업데이트해야 된다는 특징이 있어요. 즉, 리소스 일부만 수정하고 싶어도 전체 필드 데이터를 요청 본문에 포함해야 돼요.

## PATCH

PATCH도 리소스 업데이트할 때 사용하는데요. PATCH 메서드는 [RFC 5789](https://www.rfc-editor.org/rfc/rfc5789)에 다음과 같이 정의되어 있어요.

> The PATCH method requests that a set of changes described in the request entity be applied to the resource identified by the Request-URI.

PATCH 메서드도 PUT 메서드와 같이 특정 리소스 URI를 정확히 알고 있어야 돼요. 그러나 PUT은 서버에 있는 리소스를 완전히 대체하지만, PATCH는 클라이언트의 요청에 따라 리소스를 수정하고 **부분 업데이트**를 합니다. 일반적으로 PATCH 요청은 변경이 필요한 필드만 요청 본문으로 보내고, 변경하고 싶지 않은 필드는 요청 본문에서 생략해요. 하지만 필요에 따라 다른 방법으로 PATCH 요청을 제공할 수도 있어요.

PATCH 메서드는 [RFC 2616](https://www.rfc-editor.org/rfc/rfc2616)에 정의된 기준으로 멱등하지 않고 안전하지 않아요. 예를 들어, 두 개의 PATCH 메서드가 동시에 호출되면 리소스의 데이터가 손실될 수 있고 클라이언트가 원하지 않는 결과가 나올 수도 있어요.

그럼 언제 PUT을 사용하고, 언제 PATCH를 사용하면 좋을까요? 클라이언트의 상황에 따라 유연하게 사용하면 됩니다. RFC 문서는 PATCH에 넣을 데이터가 PUT에 넣을 데이터보다 크다면, PUT이 더 적합한 메서드라고 기술하고 있어요. POST와 비교하기는 더 어려워요. POST 메서드는 PUT, PATCH와 비슷한 기능을 모두 수행할 수 있기 때문이에요.

## 들어가며

오늘은 POST, PUT, PATCH 메서드를 자세히 알아봤는데요. 각 메서드의 특징은 아래와 같이 정리할 수 있습니다. 토스페이먼츠는 앞으로도 이해하기 쉽고 편리한 API 디자인을 제공하기 위해서 노력하겠습니다.

| 메서드       | 설명                        | 요청 URI      | 멱등성 |
| --------- | ------------------------- | ----------- | --- |
| **POST**  | 리소스 생성 또는 서버에서 정의한 다양한 작업 | 리소스 컬렉션 URI | X   |
| **PUT**   | 리소스 생성 및 전체 업데이트          | 특정 리소스 URI  | O   |
| **PATCH** | 리소스 부분 업데이트               | 특정 리소스 URI  | X   |