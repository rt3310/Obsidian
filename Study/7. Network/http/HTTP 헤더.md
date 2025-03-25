## HTTP BODY

### RFC7230
- 메시지 본문(message body)을 통해 표현(representation) 데이터 전달
- 메시지 본문 = 페이로드(payload)
- 표현은 요청이나 응답에서 전달할 실제 데이터
- 표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공
	- 데이터 유형(html, json), 데이터 길이, 압축 정보 등등

## 표현 헤더

- Content-Type: 표현 데이터의 형식
- Content-Encoding: 표현 데이터의 압축 방식
- Content-Language: 표현 데이터의 자연 언어
- Content-Length: 표현 데이터의 길이

표현 헤더는 전송, 응답 둘 다 사용

### Content-Type
표현 데이터의 형식 설명
- 미디어 타입, 문자 인코딩
#### 예시
- text/html; charset=utf-8
- application/json
- image/png
### Content-Encoding
표현 데이터 인코딩
- 표현 데이터를 압축하기 위해 사용
- 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
- 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
#### 예시
- gzip
- deflate
- identity
### Content-Language
표현 데이터의 자연 언어
- 표현 데이터의 자연 언어를 표현
#### 예시
- ko
- en
- en-US
### Content-Length
표현 데이터의 길이
- 바이트 단위
- Transfer-Encoding(전송 코딩)을 사용하면 Content-Length를 사용하면 안됨

## 협상(Content Negotiation)

클라이언트가 선호하는 표현 요청
- Accept: 클라이언트가 선호하는 미디어 타입 전달
- Accept-Charset: 클라이언트가 선호하는 문자 인코딩
- Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
- Accept-Language: 클라이언트가 선호하는 자연 언어
협상 헤더는 요청 시에만 사용
### 협상과 우선순위1
- Quality Values(q) 값 사용
- 0~1, **클수록 높은 우선순위**
- 생략하면 1
- Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
	1. ko-KR;q=1 (q생략)
	2. ko;q=0.9
	3. en-US;q=0.8
	4. en;q=0.7
