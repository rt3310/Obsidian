1. 웹 클라이언트가 WAS에 요청을 보낸다
2. WAS가 세션 키를 생성한다.
3. WAS가 세션 키를 이용해 저장소를 생성한다.
4. WAS가 세션 키를 담은 Cookie를 생성한다.
5. 웹 클라이언트에 세션 키를 담은 쿠키를 포함하여 응답한다
6. 웹 클라이언트가 세션 키를 가지고 있는 쿠키를 가지고 WAS에 요청을 보낸다.
7. WAS가 쿠키의 세션 키를 이용해 이전에 생성한 저장소를 활용한다.