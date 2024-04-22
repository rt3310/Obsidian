# HttpSession은 어떻게 만들어지고 어떻게 유지될까?

## 서론

클라이언트와 서버는 Stateless인 HTTP 통신을 하게 되지만 로그인과 같이 접속을 했던 정보가 저장이 되어야 할 때가 있다. 이때 인증과 인가가 필요하게 된다.
인증은 클라이언트에서 보낸 정보에 담긴 아이디와 패스워드가 서버에서 저장하고 있는 정보와 일치할 때 인증이 발생하게 된다. 그 이후 로그인을 한 사용자는 자신의 계정으로 글을 쓰거나 물건을 살 때 무상태로 유지되게 되면 본인이 그 사용자임을 계속해서 인증을 해야한다. 이를 방지하기 위해 서버는 인증이 된 클라이언트에게 쿠키 또는 세션을 발급해주고 클라이언트는 이를 요청 정보에 담아 보냄으로써 지속적인 인증 없이 사용자 인증이 필요한 요청들을 할 수 있게 된다.

## 쿠키

쿠키는 서버에서 필요한 정보를 지정하면 클라이언트 측에서 저장을 하고 HTTP 요청마다 메세지에 담아 보내게 된다. 이로써 서버에 메모리 부담을 줄일 수 있지만 요청 시 쿠키 내부에 있는 보안정보들이 그대로 노출될 수 있어서 보안상의 문제가 있고, 쿠키의 크기가 클 경우 네트워크 부하가 커질 수 있다. 그리고 사용자가 보안상 쿠키 수집을 거절할 경우 사용이 불가능하며 웹 브라우저마다 지원형태가 달라 호환이 안된다.

## 세션
세션은 인증된 사용자의 정보를 SESSION ID와 매핑하여 서버에 저장하고 클라이언트에게 식별자와 문자열로 이루어진 SESSION ID를 응답헤더에 넣어 전송한다.
매 응답마다 SESSION ID만 보내기 때문에 네트워크 부하가 커지지 않고 서버에 저장되므로 클라이언트의 웹브라우저의 호환성 문제가 해결된다. 그리고 많은 정보를 유지할 수 있게 된다. 하지만 서버에 데이터 저장량이 많아지므로 서버 메모리에 부담이 갈 수 있다.

## 쿠키 / 세션 정리
세션도 결국 쿠키를 사용하기 때문에 역할과 동작원리는 비슷하다.
- 쿠키는 서버의 자원을 전혀 사용하지 않지만 네트워크에 부담이 생길 수 있고, 세션은 서버의 자원을 사용하지만 네트워크에 부담이 가지 않는다.
- 보안 면에서는 세션이 더 우수하지만, 세션은 서버에서 처리하기 때문에 요청 속도는 쿠키가 더 빠르다.
- 쿠키도 만료 시간이 있지만 파일로 저장되기 때문에 브라우저를 종료해도 계속해서 정보가 남아있을 수 있다. 또한 만료기간을 넉넉하게 잡아두면 쿠키 삭제를 할 때 까지 유지될 수도 있다. 반면, 세션도 만료시간을 정할 수 있지만 브라우저가 종료되면 만료시간에 상관없이 삭제된다. 예를 들어, 크롬에서 다른 탭을 사용해도 세션은 공유되며 다른 브라우저를 사용하게되면 다른 세션을 사용하게 된다.

# HttpSession

## Session 생성

### 1. @Autowired 사용
```java
@Controller 
public class LoginController {
	@Autowired
	private HttpSession session;

	@PostMapping("/login")
	public String login(MemberLoginDto memberLoginDto, Model model) {
		Member loginMember = loginService.login(memberLoginDto.getUserId(), memberLoginDto.getPassword());

		if (loginMember == null) {
			result.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
			return "member/loginForm";
		}

		// 로그인 성공 처리
		// 세션이 있으면 있는 세션 반환, 없으면 신규 세션을 생성
		HttpSession session = request.getSession();
		// 세션에 로그인 회원 정보 보관
		session.setAttribute("loginMember", loginMember);
		model.addAttribute("member", loginMember);
		return "member/memberInfo";
	}
}
```
HttpSession을 빈 주입 받는 것만으로는 Session이 생성되지 않는다. 해당 메서드의 `setAttribute()`나 `getAttribute()`이 호출되는 시점에 서블릿 컨테이너에서 session을 받을 수 있다.

### 2. 메서드 주입
```java
@Controller
public class LoginController {

	@PostMapping("/login")
	public String login(MemberLoginDto memberLoginDto, HttpSession session, Model model) {
				Member loginMember = loginService.login(memberLoginDto.getUserId(), memberLoginDto.getPassword());

		if (loginMember == null) {
			result.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
			return "member/loginForm";
		}
		
		HttpSession session = request.getSession();
		session.setAttribute("loginMember", loginMember);
		model.addAttribute("member", loginMember);
		return "member/memberInfo";
	}
}
```
1번과 방식이 비슷하지만 파라미터로 받게된다면, login 메서드가 실행되는 시점에 Session을 서블릿 컨테이너로부터 전달 받게 된다.

### 3. @SessionAttribute, @ModelAttribute로 주입
```java
@Controller
@SessionAttributes("loginMember")
public class LoginController {

	@PostMapping("/login")
	public String login(@ModelAttribute("loginMember") Member member, Model model) {
				Member loginMember = loginService.login(memberLoginDto.getUserId(), memberLoginDto.getPassword());

		if (member != null) {
			model.addAttribute("member", member);
			return "member/memberInfo";
		}
		model.addAttribute("memberLoginDto", new MemberLoginDto());
		return "member/loginForm";
	}
}
```
이미 생성되어있는 세션이 있을 때 Get 요청을 하면서 동일한 키(loginMember)를 조회하여 Member 파라미터로 전달 받게 된다.

## 세션 유지

Request시 Header에 SessionID가 포함되어 전달된다면 서블릿 컨테이너는 세션을 발급하지 않고 해당 SessionID에 해당하는 세션을 전달하게 되고, SessionID가 포함되지 않는다면 HttpSession을 요구하는 모든 요청에 대해 새로운 Session을 발급한다.
Spring에서 기본 서블릿 컨테이너를 Tomcat으로 사용하고 있기 때문에 Tomcat 기준의 내용이다.
Tomcat의 경우 SessionID를  JSESSIONID라는 쿠키를 생성하여 클라이언트에게 전달하고 클라이언트는 JSESSION이 담긴 쿠키를 헤더에 포함하여 인증을 유지한다.
![[Pasted image 20240422154851.png]]
![[Pasted image 20240422154905.png]]

## JSESSION은 어떻게 생성될까?
![[Pasted image 20240422154959.png]]
JSESSION은 HttpServletRequest의 getSession의 옵션에 따라 자동 생성이 된다. 그렇다면 먼저 HttpServletRequest 인터페이스에 있는 getSession을 추적해보자.
![[Pasted image 20240422155212.png]]
HttpServletRequest 인터페이스의 getSession 구현체는 아래와 같이 확인된다. 우리는 이것들 중 ApplicationHttpRequest를 추적하여 본다.