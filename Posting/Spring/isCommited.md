`@RestController`를 사용하거나  `@Controller` + `@ResponseBody`를 사용할 때 dto, entity, String, Long 등을 반환하게 되면 Interceptor에서 response를 꺼내 수정할 수 없다.

아래와 같이 `@RestController`가 선언된 UserController가 제공하는 API는 모두 Interceptor에서 값을 수정할 수 없다. 여기서 더 중요한건 controller의 return type이다.
```java
@RestController  
@RequestMapping("/users")  
public class UserController {  
	// UserService 생성자 주입받는 코드  
	  
	@GetMapping("/string")  
	public String returnString() {  
		return "AA";  
	}  
	  
	@GetMapping("/boolean")  
	public boolean returnBoolean() {  
		return true;  
	}  
	  
	@GetMapping("/user")  
	public User returnUser() {  
		return new User("email", "password", "01012331233", true);  
	}  
	  
	@GetMapping()  
	public List<UserResponse> findAll() {  
		return userService.findAll();  
	}  
}
```
반대로 Interceptor에서 response 수정이 가능한 return type을 가진 형태는 아래와 같다.
즉, `void`나 `Null`, `ResponseEntity<Void>` 와 같은 return type을 가지고 있다.

```java
@RestController  
@RequestMapping("/users")  
public class UserController {  
	// UserService 생성자 주입받는 코드  
	  
	@GetMapping("/null")  
	public Null returnNull() {  
		return null;  
	}  
	  
	@GetMapping("/response-entity")  
	public ResponseEntity<Void> returnResponseEntityVoid() {  
		return ResponseEntity.ok().build();  
	}  
	  
	@PostMapping()  
	public void signUp(@RequestBody UserRequest request) {  
		userService.signUp(request);  
	}  
}
```

Interceptor를 통해 response를 조작하고 싶다면 Controller가 return 한 뒤에 실행되는 `postHandle`을 생각하게 되는데 이 때 인자로 받는 `HttpServletResponse` 에는 isCommitted라는 메서드가 있다.

```java
@Override  
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
	System.out.println("isCommitted: " + response.isCommitted());  
	HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);  
}
```

`HttpServletResponse`의 `isCommitted` 메서드가 true라면 이미 response data가 쓰여진 상태로 출력 stream을 통해 전송되었기 때문에 이 상태에서는 response의 데이터를 꺼내 수정해서 다시 넣을 수 없다.

> `ServletResponse.isCommited()` checks if the response has been already committed to the client or not (Means the servlet output stream has been opened to writing the response content).
> 
> The committed response holds the HTTP Status and Headers and you can’t modify it.

Interceptor를 적용하고 위에서 값을 수정할 수 없는 API는 모두 `isCommitted`가 true를 반환하고 값을 수정할 수 있는 API는 false를 반환한다. 이를 확인하기 위해 Interceptor를 등록해보자.

아래와 같이 WebMvcConfigurer를 구현하는 class에 Custom 인터셉터를 등록했다.
```java
@Configuration  
public class InterceptorConfig implements WebMvcConfigurer {  
	@Override  
	public void addInterceptors(InterceptorRegistry registry) {  
		registry.addInterceptor(new CustomInterceptor());  
	}  
}
```
response를 조작한다는 관점에서 관심사는 `postHandle` 메서드이기 때문에 이 메서드만 오버라이딩 해서 application을 실행해보자.

```java
@Component  
public class CustomInterceptor implements HandlerInterceptor {  
	@Override  
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
		System.out.println("isCommitted: " + response.isCommitted());  
		HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);  
	}  
}
```

컨트롤러에 적당한 로깅을 남기고 테스트해보면 맨 위에서 결론을 낸 것처럼 반환 값이 없을 때 Interceptor에서 수정이 가능하다.

```text
======return User Entity======  
postHandle isCommitted: true  
  
======return User Response======  
postHandle isCommitted: true  
  
======return Null======  
postHandle isCommitted: false  
  
======return ResponseEntity<Void>======  
postHandle isCommitted: false  
  
======signUp======  
postHandle isCommitted: false
```

반환 값이 없는 경우(`isCommitted`가 false인 경우)에 데이터를 수정하고 싶다면 아래와 같은 방식으로 수정할 수 있다.
```java
@Component  
public class CustomInterceptor implements HandlerInterceptor {  
	@Override  
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {  
		User user = new User("email", "password", "01012331233", true);  
		  
		String newContent = new ObjectMapper().writeValueAsString(user);  
		response.setContentType("application/json");  
		response.setContentLength(newContent.length());  
		response.getOutputStream().write(newContent.getBytes());  
		HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);  
	}  
}
```

## 반환 값이 있을 때 Response 데이터를 수정하고 싶다면?

먼저 `FilterRegistrationBean`에 작성한 Filter를 등록해보자.
```java
@Configuration  
public class FilterRegistry {  
	@Bean  
	public FilterRegistrationBean<Filter> registrationBean() {  
		FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<>();  
		registrationBean.setFilter(new CustomFilter());  
		return registrationBean;  
	}  
}
```

아래는 직접 작성한 CustomFilter로 `chain.doFilter` 메서드를 기준으로 다음 Filter나 dispatcher servlet으로 넘기기 전/후 `ServletResponse.isCommitted` 메서드를 호출하는 것을 알 수 있다.

```java
@Component  
public class CustomFilter implements Filter {  
	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
		System.out.println("=====request do Filter=====");  
		System.out.println("response.isCommitted():" + response.isCommitted());  
		  
		chain.doFilter(request, response);  
		  
		System.out.println("=====response do Filter=====");  
		System.out.println("response.isCommitted():" + response.isCommitted());  
	}  
}
```

`isCommitted` 실행결과가 true라면 그 시점에 response는 이미 output stream에 데이터가 쓰여지고 전송되었기 때문에 이때는 수정할 수 없다.

간단한 UserController를 만들고 값을 리턴하는 API를 만들자
```java
@RestController  
@RequestMapping("/users")  
public class UserController {  
	private final UserService userService;  
	  
	public UserController(UserService userService) {  
		this.userService = userService;  
	}  
	  
	@GetMapping()  
	public List<UserResponse> findAll() {  
		return userService.findAll();  
	}  
}
```
Filter가 적용된 상태로 /users로 GET 요청을 보내보면 아래와 같이 출력이 남는다.
request 영역에서는 아직 응답이 commit되지 않았기 때문에 false지만 response 영역에서는 true인 상태로 응답이 온다.

```text
=====request do Filter=====  
response.isCommitted():false  
  
=====response do Filter=====  
response.isCommitted():true
```

Interceptor와 마찬가지로 return되는 데이터가 있으면 response 데이터를 변경할 수 없다. 그렇다면 어떻게 변경할 수 있을까?

스프링에는 `ContentCachingResponseWrapper`라는 클래스가 있는데 이를 통해 Output Stream에 쓰여진 데이터를 캐시하고 Byte Array를 통해 이를 읽을 수 있는 방법을 제공한다.

따라서 내가 작성한 Filter를 아래와 같이 수정하자. 핵심은 Request 영역의 response를 `ContentCachingResponseWrapper` 클래스로 변경 후 이를 다음 chain.doFilter로 전달하는 것이다.

```java
@Component  
public class CustomFilter implements Filter {  
	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
		System.out.println("=====request do Filter=====");  
		System.out.println("response.isCommitted():" + response.isCommitted());  
		  
		ContentCachingResponseWrapper responseWrapper =  
		new ContentCachingResponseWrapper((HttpServletResponse) response);  
		  
		chain.doFilter(request, responseWrapper); // responseWrapper를 넣는다.  
		  
		System.out.println("=====response do Filter====="); 
		System.out.println("response.isCommitted():" + response.isCommitted());  
		System.out.println("responseWrapper.isCommitted():" + responseWrapper.isCommitted());  
	}
}
```

`ContentCachingResponseWrapper` 클래스는 내부적으로 상속 관계를 따라가다 보면 결국 ServletResponse 인터페이스를 구현하기 때문에 FilterChain에 넘겨줘도 정상적으로 동작한다.

이 때 /users API를 호출하면 출력되는 로그를 보면 아래와 같다.
```text
=====request do Filter=====  
response.isCommitted():false  
  
=====response do Filter=====  
response.isCommitted():false  
responseWrapper.isCommitted():false
```

`isCommited`가 false라는 의미는 아직 데이터가 Output Stream에 쓰여지지 않았기 때문에 이제 response 데이터를 수정할 수 있다.

### Response 데이터를 조작하는 법
- Response 데이터 수정 없이 로깅만 하고 원본 데이터를 응답하는 경우
- 기존 데이터를 무시하고 새로운 데이터를 응답하는 경우
- 기존 데이터를 읽어 수정한 뒤 응답하는 경우

#### Response 데이터 수정 없이 로깅만 하고 원본 데이터를 응답하는 경우
`ContentCachingResponseWrapper`가 제공하는 `getContentAsByteArray` 메소드로 byte 배열을 읽고 이를 String으로 변환 후에 로깅한다.

이 때 맨 마지막 line인 `copyBodyToResponse` 메서드는 캐시해 둔 원본 데이터를 다시 response에 저장하는 방법이다. `ContentCachingResponseWrapper`가 response output stream에서 데이터를 읽는 시점에 stream은 빈 상태가 된다.
따라서 기존 응답을 그대로 전달하고 싶다면 `copyBodyToResponse` 메서드를 호출해야 한다.
```java
@Component  
public class CustomFilter implements Filter {  
	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
		ContentCachingResponseWrapper responseWrapper =  
		new ContentCachingResponseWrapper((HttpServletResponse) response);  
		  
		chain.doFilter(request, responseWrapper);  
		  
		byte[] responseArray = responseWrapper.getContentAsByteArray();  
		String responseStr = new String(responseArray, responseWrapper.getCharacterEncoding());  
		System.out.println(responseStr); // 여기서 로깅한다.  
		responseWrapper.copyBodyToResponse();  
}
```

#### 기존 데이터를 무시하고 새로운 데이터를 응답하는 경우
아래 Filter 에서는 ObjectMapper로 Json 객체를 생성하고 값을 넣어 응답하고 있다.

```java
@Component  
public class CustomFilter implements Filter {  
	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
		ContentCachingResponseWrapper responseWrapper =  
		new ContentCachingResponseWrapper((HttpServletResponse) response);  
		  
		chain.doFilter(request, responseWrapper);  
		  
		ObjectNode json = new ObjectMapper().createObjectNode();  
		json.put("message", "this response is modified");  
		  
		String newResponse = new ObjectMapper().writeValueAsString(json);  
		response.setContentType("application/json");  
		response.setContentLength(newResponse.length());  
		response.getOutputStream().write(newResponse.getBytes());  
	}  
}
```

#### 기존 데이터를 읽어 수정한 뒤 응답하는 경우
아래의 Filter는 응답 타입이 `List<UserResponse>` 일 때 어거지로 값을 수정하는 예제다.

```java
@Component  
public class CustomFilter implements Filter {  
	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
		HttpServletRequest httpRequest = (HttpServletRequest) request;
		ContentCachingResponseWrapper responseWrapper = 
		new ContentCachingResponseWrapper((HttpServletResponse) response);
		
		chain.doFilter(request, responseWrapper);
		
		if (httpRequest.getMethod().equals(HttpMethod.GET.name())) {  
			byte[] responseArray = responseWrapper.getContentAsByteArray();
			String responseStr = new String(responseArray, responseWrapper.getCharacterEncoding());  
			List<UserResponse> userResponses = Arrays.asList(new ObjectMapper().readValue(responseStr, UserResponse[].class));  
			  
			// UserResponse에 setter가 있다고 가정한다면  
			userResponses.get(0).setId(9999L);
			
			String newResponse = new ObjectMapper().writeValueAsString(userResponses);  
			response.setContentType("application/json");
			response.setContentLength(newResponse.length());
			response.getOutputStream().write(newResponse.getBytes());
		}
	}
}
```
