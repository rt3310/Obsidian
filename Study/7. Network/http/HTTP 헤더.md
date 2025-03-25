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
### 협상과 우선순위2
- 구체적인 것이 우선한다.
- Accept: text/\*, text/plain, text/plain;format=flowed, \*/*
	1. text/plain;format=flowed
	2. text/plain
	3. text/*
	4. \*/*
### 협상과 우선순위3
- 구체적인 것을 기준으로 미디어 타입을 맞춘다.
- Accept: text/\*;q=0.3, text/html;q=0.7, text/html;level=1, text/html;level=2;q=0.4, \*/\*;q=0.5

| Media Type        | Quality |
| ----------------- | ------- |
| text/html;level=1 | 1       |
| text/html         | 0.7     |
| text/plain        | 0.3     |
| image/jpeg        | 0.5     |
| text/html;level=2 | 0.4     |
| text/html;level=3 | 0.7     |

## 전송 방식
### 단순 전송
![[Pasted image 20250325163526.png]]
### 압축 전송
![[Pasted image 20250325163534.png]]
### 분할 전송
![[Pasted image 20250325163608.png]]
- 0이면 끝났다는 표시
- 분할 전송 시에는 Content-Length를 넣으면 안된다.
### 범위 전송
![[Pasted image 20250325163649.png]]

## 일반 정보
### From
유저 에이전트의 이메일 정보
- 일반적으로 잘 사용되지 않음
- 검색 엔진 같은 곳에서, 주로 사용
- 요청에서 사용
### Referer
이전 웹 페이지 주소
- 현재 요청된 페이지의 이전 웹 페이지 주소
- A -> B로 이동하는 경우 B를 요청할 때 Referer: A를 포함해서 요청
- Referer를 사용해서 유입 경로 분석 가능
- 요청에서 사용
### User-Agent
유저 에이전트 애플리케이션 정보
- uesr-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
- 클라이언트의 애플리케이션 정보(웹 브라우저 정보, 등등)
- 통계 정보
- 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
- 요청에서 사용
### Server
요청을 처리하는 ORIGIN 서버의 소프트웨어 정보
- Server: Apache/2.2.22 (Debian)
- server: nginx
- 응답에서 사용
### Date
메시지가 발생한 날짜와 시간
- Date: Tue, 15 Nove 1994 08:12:31 GMT
- 응답에서 사용

## 특별한 정보

### Host
요청한 호스트 정보(도메인)
- 요청에서 사용
- 필수
- 하나의 서버가 여러 도메인을 처리해야 할 때
- 하나이 IP 주소에 여러 도메인이 적용되어 있을 때
### Location
페이지 리다이렉션
- 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동(리다이렉트)
- 응답코드 3xx에서 설명
- 201 (Created): Location 값은 요청에 의해 생성된 리소스 URI
- 3xx (Redirection): Location 값은 요청을 자동으로 리디렉션하기 위한 리소스를 가리킴