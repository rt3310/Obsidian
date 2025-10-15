# 특징
- 기본적으로 클라이언트의 소유이다.
- 서버에 요청 시 header에 자동으로 포함돼서 전송된다.
- 주로 서버 단의 필요에 의해 클라이언트에 생성하도록 지시한다.
    - ex) JSESSIONID
- HttpOnly, Secure 속성은 서버 → 클라이언트로 던져주며, 클라이언트 → 서버는 값을 주지 않는다.
    - 디버깅 창에서 Response 속성으로만 확인 가능하다.

## Tomcat 8.5 이상

- httpOnly default = true
    - true이면 Javascript로 탈취할 수 없다.
- secure=true이면 https로만 전송된다.

# 세션 쿠키 vs 영구적 쿠키

- 만료시각을 지정하면 영구적 쿠키
    - 파일 저장
- 만료시각이 없으면 세션 쿠키
    - 메모리 저장
    - 브라우저를 닫으면 삭제된다

```java
class Java {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

```java
class Java {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

```java
@Service
class MemberService {
    private final MemberRepository;

    public findById(Long id) {
        member
}
```