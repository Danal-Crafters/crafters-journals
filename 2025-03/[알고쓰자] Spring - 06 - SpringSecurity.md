# 🍃 Intro

다시 돌아왔습니다. 벌써 여섯번째 시리즈를 다루고 있군요.
다섯번의 시리즈 동안 Spring 의 기초 원리를 이야기 해 보았습니다. 간간히 객체지향에 대한 이야기도 나눴죠.

이번 편 부터는 Spring 생태계에서 사용할 수 있는 모듈들을 하나하나 소개 해드릴까 합니다. 이번 호에서는 개인적으론 Spring 을 배울 때 꽤나 진입장벽이 높았던 Spring Security 를 다뤄볼까 합니다.

> 글을 쓰면서도 조금 두렵습니다. Spring Security 를 처음 쓸 때와 지금의 생각은 많이 변했죠...뜯어보면 볼 수록 쉽지 않다는 생각이 듭니다.

이번 호도 힘차게 시작 해 보겠습니다. 커피 한잔과 함께 즐겨주세요 ☕.

# 🔒 일반적으로 백엔드 애플리케이션에 필요 한 보안 기능들

보통 웹사이트에 대한 예시를 들 때 쇼핑몰이 많이 나오죠. 그 만큼 많이 사용하시니 공감대가 높아서라고 생각합니다.

> 야밤에 핸드폰 볼 때 충동구매 단 한번도 안해보신 분이 계시다면, 무엇을 하더라도 성공하실 분이니 친하게 지내시기 바랍니다. 

우리는 쇼핑몰에 로그인을 해서 결제를 합니다. 일반적으론 그렇죠. 판매를 하시는 분이라면, 판매자의 정보들이 쇼핑몰에 작성되겠죠. 

이 모든 행위들은 타인의 귀속 자산이 아닌 개인 정보를 통해 이뤄집니다. 즉, 이 정보들이 노출된다면 우리가 원하지 않는 상황들이 발생할 수 있죠. 웹 서비스들은 고객을 위한 보안 기능들을 제공함으로써 이런 문제를 해결하고 있습니다. 

우선 암호화 기능이 필요합니다. 개인정보 데이터를 암호화해서 저장해야 하니까요. 또한 로그인 상태를 체크할 수 있는 기능들이 필요합니다. 어떤 서비스는 유저의 권한에 따라 특정 데이터에 대한 접근을 막아야하죠. 이를 흔히 말해서 인증, 인가 기능이라고 부릅니다. 

그 외에도 다양한 기능들이 있죠. Google 로그인 과 같은 기능도 필요합니다. 누군가는 계정을 여러 개를 생성하는 데 피로감을 느낄 수도 있으니까요. 

이 모든 기능들은 유저마다 역할을 분류하고 개인의 데이터를 지키는데 의미가 있습니다. 이런 기능들이 없다면 서비스는 중구난방이 되겠죠. 최소한 계정 정보를 저장하고 유저에게 로그인을 요구하는 서비스라면 말이죠. 단순 조회 서비스, 정적 페이지에서는 불필요할 수 있습니다. 

# 🧐 우리가 할 수 있는 선택지

당연하게도 모든 기능들을 직접 구현하는 방법이 있습니다. 학부 시절로 기억을 되돌려 보겠습니다. 전 학부에서 웹 프로그래밍에 대해 배울 때 JSP 와 Servlet 을 사용했었습니다. 특별히 프레임워크를 사용하지 않았죠. 학부 시절 열에 열은 꼭 해 보는 게시판 서비스 개발에서 JSP, Servlet 을 통해 로그인을 구현했던 기억이 납니다. 

잘 와닫지 않으실 것 같아서 예시를 하나 준비했습니다. 이 글을 읽으시는 분들 중에선 프레임워크 부터 시작하신 분도 분명 계실테니까요. 

> 학부 수업시간엔 원론적인 내용이 우선이다보니 날 Servlet 부터 공부했던 기억이 납니다. 😁
>  
> 당시엔 고통스러웠지만 지금 코드들을 다시 보니 추억이네요. 👍

```bash
jsp-servlet-login/
│── WebContent/
│   ├── index.jsp       (로그인 폼)
│   ├── welcome.jsp     (로그인 성공 페이지)
│   ├── error.jsp       (로그인 실패 페이지)
│── src/
│   ├── LoginServlet.java (로그인 처리 서블릿)
│── web.xml             (서블릿 매핑)
```
간단하게 이런 구조의 페이지가 존재한다고 가정 해 보겠습니다. 

web.xml 은 아래와 같습니다.
```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         version="3.0">
    <servlet>
        <servlet-name>LoginServlet</servlet-name>
        <servlet-class>LoginServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LoginServlet</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>LogoutServlet</servlet-name>
        <servlet-class>LogoutServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LogoutServlet</servlet-name>
        <url-pattern>/logout</url-pattern>
    </servlet-mapping>
</web-app>

```

index.jsp 에서 유저 정보를 입력한 후 로그인 요청을 보낸다면, 

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Login Page</title>
</head>
<body>
    <h2>Login</h2>
    <form action="login" method="post">
        <label>Username:</label>
        <input type="text" name="username" required>
        <br>
        <label>Password:</label>
        <input type="password" name="password" required>
        <br>
        <input type="submit" value="Login">
    </form>
</body>
</html>

```
request 파라미터에서 유저 정보를 받아온 후, 인증 로직이 성공한다면 세션 정보를 갱신합니다. 

```java
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        // 간단한 하드코딩된 유저 검증 (실제 환경에서는 DB를 사용해야 함)
        if ("admin".equals(username) && "1234".equals(password)) {
            HttpSession session = request.getSession();
            session.setAttribute("user", username);
            response.sendRedirect("welcome.jsp");
        } else {
            response.sendRedirect("error.jsp");
        }
    }
}
```
welcome.jsp 에서는 세션에 로그인 정보가 없다면 로그인 페이지로 이동시키고, 있다면 로그인 한 유저정보를 표시합니다. 
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="javax.servlet.http.HttpSession" %>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h2>Welcome Page</h2>
    <%
        HttpSession sessionObj = request.getSession(false);
        if (sessionObj != null && sessionObj.getAttribute("user") != null) {
    %>
        <p>Welcome, <%= sessionObj.getAttribute("user") %>!</p>
        <a href="logout">Logout</a>
    <%
        } else {
            response.sendRedirect("index.jsp");
        }
    %>
</body>
</html>
```
이처럼 java servlet 만 가지고도 심플하게 로그인을 구성할 수 있습니다. 하지만 생각보다 우리에게 요구되는 보안 요건들이 다양할 수 있습니다. 해당 예시에는 암복호화 로직이 없죠. 간단한 예시라 JDBC 커넥션 또한 따로 맺고 있지 않습니다. 

서비스를 구성하는 데 필요한 다양한 기능들을 이렇게 직접 구현할 수도 있겠지만, 사실 패턴들이 하나같이 정형화 되어있습니다. 아이디와 패스워드를 전달받아 유저를 검증하는 패턴이 서비스마다 사실 크게 다르지 않죠. 

만약 이런 정형화된 기능에 대해 큰 시간을 들이고 싶지 않다면 도구를 사용하는 선택지도 있습니다. Spring Security 가 그 예시죠. 

# 🍃 Spring Security 개요

Spring Security 는 Spring 기반 애플리케이션에 여러 보안 기능들을 구현하는 데 도움을 주는 프레임워크입니다.  
프레임워크이니까 막상 모든 기능들이 다 구현되어있지는 않겠지만, 앞서 보여드린 날 JSP, Servlet 코드에 비하면 구현할 게 줄죠.  

물론 JSP + Spring 조합에서도 많이 쓰입니다. Spring 은 웹 서비스를 개발하는 데 필요한 많은 도구들을 제공합니다. 그 중에선 Servlet 도 당연히 포함 되어있죠. 인증 및 인가 처리에 필요한 도구 뿐만 아니라 암호화 도구들도 제공 합니다. 

다시 과거로 기억을 되돌려 JSP + Spring 만 가지고 웹을 개발한다고 가정 해 보겠습니다. 여전히 인증, 인가 등의 기능은 직접 구현해야 합니다. 

앞서 말씀드렸다시피 이런 기능들은 사실 정형화 되어있는 기능들입니다. 10 명의 개발자에게 세션 기반 인증, 인가를 구현하라고 하면 대부분은 아래의 패턴을 따를겁니다. 

1. 인증을 받는 시점(Servlet 시점), 또는 인증이 핸들러로 전달 된 시점 (Controller 시점) 에서 HttpServletRequest 내의 세션 정보를 가져온다. 
2. 서버 내에 저장 된 세션 정보와 비교한다. 
3. 통과 혹은 제한

1번에서 시점을 두가지로 나누었습니다. Spring MVC 의 구조에서는 `Controller` 핸들러보다 앞단에 `Dispatcher Servlet` 이 존재하니까요. 
사실 요청 자체가 이쯤에서 걸러지는 게 가장 이상적입니다. Controller 객체의 책임이 유저 인증 인가 까지 가지고 있게 된다면, AOP 를 통해 또 하나의 횡단 관심사를 전역 Controller 에 구현해야 할 수 있습니다. 

물론 `AOP` 로 구현하는 건 좋은 선택지 입니다. `Controller` 가 가져야 할 관심사가 인증, 인가라고 결정했다면 말이죠. `Controller` 단에서 작성해야 할 코드가 상당히 줄어들겠죠.  

하지만, 이 경우엔 모든 요청이 일단은 통과된다는 문제점이 생깁니다. 우선은 요청 자체가 통과 되었기 때문에 `Handler Adapter` 가 동작을 하죠. **이 상황 자체가 문제가 된다고 판단했다면, 인증 및 인가의 책임을 `Controller` 에게 줘서는 안됍니다.** 

`Servlet Filter` 라는 키워드가 떠올랐다면, Spring Security 를 도입 해 보는 것을 고려해볼 수 있습니다. Spring Security 는 `Controller` 영역에서 인증 및 인가의 검증 책임을 덜어주는 역할을 합니다. 

이후에 언급하겠지만, Spring Security 는 Servlet Filter 영역에서 동작합니다. Servlet Filter 영역에서 허가받지 않은 요청을 처리한다면, Controller 영역에서는 이런 요청에 대해 전혀 신경쓰지 않아도 좋습니다. 어차피 올 일이 없을테니까요. 

# 🕸️ Spring Security 구조

Spring Security 는 엄밀히 말하자면 Spring 에 포함 되어있는 모듈이라고 말 하기는 어렵습니다. Spring Security 가 Spring 의 영역 내부에 있다고 말 할수는 없기 때문입니다. 

하지만 Spring 내의 Bean 들을 Servlet 영역에서 사용할 수 있도록 도와주는 역할을 합니다. DelegatingFilterProxy 를 통해서 말이죠. 

<p align="center">
    <img src="https://github.com/user-attachments/assets/571271fb-7f86-4644-bc0a-328ce849dbc8" width="30%" />
    <img src="https://github.com/user-attachments/assets/ecd92b4c-8928-40e2-b357-ec1ed0fc7b30" width="50%" />
</p>

Spring 의 FilterChain 은 웹 애플리케이션에서 들어오는 요청과 나가는 응답을 여러 단계의 필터를 통해 처리하도록 도와줍니다. 우리가 FilterChain 에 커스텀 Filter 를 넣을 수도 있죠. 

![image](https://github.com/user-attachments/assets/7b982a78-684f-4273-b048-f24271ac9c72)

FilterChain 은 Spring 의 IOC 가 직접적으로 관여하는 영역은 아닙니다. 그러므로 Spring 의 Bean 들이 직접 IoC 될 수는 없습니다. 하지만 DelegatingFilterProxy 가 있다면, 얘기는 다릅니다. 해당 FilterProxy 는 요청을 가로채서 Spring Container 영역으로 요청을 가져오는 역할을 합니다. 

![image](https://github.com/user-attachments/assets/d3eff6e9-b70c-4d8f-87ae-a93ad1e2e837)

Spring Security 를 설정하면 이 FilterChain 영역에 DelegatingFilterProxy 를 통해 Security 의 FilterChainProxy 를 연결 시킵니다. 해당 FilterChainProxy 를 통해 실제로는 외부에 배치 되어있는 SecurityFilterChain 이 마치 FilterChain 내에 존재하는 것 처럼 사용할 수 있습니다. 

DelegatingFilterProxy 를 통해 FilterChain 에서 SecurityFilterChain 으로 요청이 올 타이밍에 Spring 의 영역의 자원들 (Beans 들) 을 사용할 수 있습니다. 즉 IoC 컨테이너에서 DI 를 받을 수 있단 의미죠. 

이를 통해 우리가 원하는 Filter 를 FilterChain 내에 커스텀할 수 있습니다. 

![image](https://github.com/user-attachments/assets/a93957ab-a953-469d-8fc2-f9d00ffcc388)


Security FilterChain 은 11개 의 기본 필터로 이뤄 져 있습니다. 이 필터들 각각이 본인의 역할들을 수행합니다. 그림에서는 필터 각각이 각자의 역할을 수행해야 할 때 호출하는 객체들을 화살표로 이어 보았습니다. 

각각의 필터들은 자신들의 역할이 주어졌을 때 조건에 따라 화살표로 연결 된 컴포넌트들을 호출합니다. 

FilterChain 내의 기본 필터들은 개발자가 커스텀 할 수 있습니다. 아예 해당 필터의 자리를 대체할 수도 있고, 앞 뒷단에 필터를 배치할 수도 있죠. 

이 설정은 filterChain 설정에서 수정할 수 있습니다. Spring Security 5.7 전에는 WebSecurityConfigurerAdapter 를 상속 인가 받아서 SecurityFilterChain 을 구성 해 주었습니다. `configure` 메소드를 사용하면 되었죠. 

하지만 최신 Spring Security 부터는 SecurityFilterChain 그 자체를 `@Bean` 으로 선언 해야 합니다. 

```kotlin
@Bean
fun filterChain(http : HttpSecurity) : SecurityFilterChain {
    return http.authorizeHttpRequests((authz) -> authz.anyRequest().authenticated())
        .httpBasic(withDefaults())
        .build()
}
```

처음에 Spring Security 를 입문할 때 봤던 교재에서 WebSecurityConfigurerAdapter 를 사용 했었는데, 그 당시에 막 Spring Boot 3.0 이 릴리즈 되었던 시기였을 겁니다. 2022년 중순 쯤이었죠. 최신버전에 익숙해지겠다고 열심히 컨픽들을 Bean 기반으로 수정했던 기억이 납니다. 생각보다는 할만 했어요. 

사실 SecurityFilterChain 보다 그 당시엔 Bean 으로 등록해야했던 다양한 컴포넌트들이 잘 이해가 안되었었죠. AuthenticationProvider 등의 역할이 정확히 어떤 지 잘 몰랐고, 왜 선언해야 하는 지도 많이 헷갈렸던 기억이 납니다. 

저는 이 문제를 동작과정을 살펴봄으로써 해결 했습니다. 


# Spring Security 동작 과정

우선 모든 요청은 FilterChain 을 통해서 전달 됩니다. FilterChain 에서 DelegatingFilterProxy 의 영역으로 요청이 전달되면, SecurityFilterChain 이 요청을 가로채서 각 Filter 들에게 요청을 보냅니다. 

요청의 성격에 따라 Filter 들이 동작하게 됩니다. Logout 처리를 담당하는 LogoutFilter 를 한번 살펴보겠습니다. 

```java

private SecurityContextHolderStrategy securityContextHolderStrategy = SecurityContextHolder
	.getContextHolderStrategy();

private RequestMatcher logoutRequestMatcher;

private final LogoutHandler handler;

private final LogoutSuccessHandler logoutSuccessHandler;

// 생략

private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
		throws IOException, ServletException {
	if (requiresLogout(request, response)) {
		Authentication auth = this.securityContextHolderStrategy.getContext().getAuthentication();
		if (this.logger.isDebugEnabled()) {
			this.logger.debug(LogMessage.format("Logging out [%s]", auth));
		}
		this.handler.logout(request, response, auth);
		this.logoutSuccessHandler.onLogoutSuccess(request, response, auth);
		return;
	}
	chain.doFilter(request, response);
}
```

만약 로그아웃 요청이라면, `LogoutHander` 를 호출하여 로그아웃 요청을 처리하고, 성공한다면 `logoutSuccessHandler` 를 호출해 후처리를 합니다. 
로그아웃 요청이 아니라면 FilerChain 의 doFilter 메소드를 호출해서 다음 Filter 에게 요청을 넘기죠. 

요청의 성격에 따라 각 필터들이 필요에 따라 동작하고, 각자의 역할을 하는 구조입니다. 만약 인증 요청이라면 `LogoutHandler` 가 동작하지 않을 것입니다.  

만약 Spring Session 이 아니라 JWT 인증을 구현한다면 어떻게 해야 할까요? JWT 토큰은 REST API 서버에서 흔히 쓰이는 인증 패턴이죠. 

> Spring Security 에서 JWT 인증을 제공하고 있지만, 이번 세션에서는 설명의 목적 상 직접 구현하는 케이스를 서술합니다.  
> Spring Security 에서 제공 하는 JWT 인증에 대해선 *다음 세션에서 서술합니다.* 😊

이 경우에는 FilterChain 을 약간 수정해야 하죠. 로그인 요청을 처리하는 필터는 `UsernamePasswordAuthenticationFilter` 입니다. 해당 필터에서 `Username` 과 `Password` 를 추출해서 `AuthenticaitonManager` 로 인증 메소드를 호출하죠. HttpSecurity 에서 제공해주는 FormLogin 기능을 사용한다면, 해당 필터를 통해 로그인 요청을 받습니다.  

JWT 토큰을 이미 요청 헤더에 전달하고 있는 상황에서는 해당 요청들을 FilterChain 에서 통과 시켜야 합니다. JWT 토큰을 검증해서 미리 분류를 해 주어야 하죠. `UsernamePasswordAuthenticationFilter` 에서는 일반적으로 로그인 요청값들에서 `Username` 과 `Password` 를 뽑아서 `AuthenticaitonManager` 에게 요청을 보냅니다. 그 후 `AuthenticationProvider` 에 따라 로그인 요청이 처리되죠.  
이미 토큰을 전달하는 경우엔 이 필터보다 먼저 `SecurityContextHolder` 에 인증 정보를 저장 해 줘야겠죠. 

```kotlin 
@Component
class JwtAuthenticationFilter(
    private val jwtProvider: JwtProvider,  // JWT 검증 및 파싱을 담당하는 Provider
    private val userDetailsService: UserDetailsService
) : OncePerRequestFilter() {

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        try {
            // 1. 요청 헤더에서 JWT 토큰 추출
            val token = resolveToken(request)

            // 2. 토큰이 유효하면 SecurityContext에 사용자 인증 정보 설정
            if (token != null && jwtProvider.validateToken(token)) {
                val username = jwtProvider.getUsername(token)
                val userDetails = userDetailsService.loadUserByUsername(username)
                val authentication = UsernamePasswordAuthenticationToken(
                    userDetails, null, userDetails.authorities
                )
                SecurityContextHolder.getContext().authentication = authentication
            }
        } catch (ex: Exception) {
            logger.error("JWT 인증 중 오류 발생: ${ex.message}")
        }

        filterChain.doFilter(request, response)
    }

    // Authorization 헤더에서 Bearer Token 추출
    private fun resolveToken(request: HttpServletRequest): String? {
        val bearerToken = request.getHeader("Authorization")
        return if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            bearerToken.substring(7)
        } else null
    }
}
```

이처럼 특정한 경우의 처리를 위해 우리는 FilterChain 에 커스텀 필터를 배치할 수 있습니다. 이 경우엔 OncePerRequestFilter 를 상속 받는 것이 좋습니다. 해당 클래스의 경우 `HttpRequest` 당 단 한번만 호출된다는 것이 보장되는 필터입니다. 
Forwarding 과 같은 케이스에서는 FilterChain 이 다시 동작하기 때문입니다. 

```kotlin
@RestController
class ForwardingController {

    @GetMapping("/test")
    fun testRequest(request: HttpServletRequest): String {
        println("Controller: Handling /test, Forwarding to /forwarded")
        request.getRequestDispatcher("/forwarded").forward(request, request)
        return "This won't be returned"
    }

    @GetMapping("/forwarded")
    fun forwardedRequest(): String {
        println("Controller: Handling /forwarded")
        return "Forwarded Response"
    }
}
```

커스텀 한 필터는 어떻게 배치할까요? filterChain 을 구성하는 Bean 메소드 안에서 해결할 수 있습니다. HttpSecurity 객체에서 `addFilter` 메소드들을 제공하죠. 
해당 메소드를 통해 특정한 Filter 앞, 뒤, 또는 해당 Filter 를 대체할 수 있죠. 아래의 예시는 `UsernamePasswordAuthenticationFilter` 앞단에 `JwtAuthenticationFilter` 를 배치하는 예시입니다. 

```kotlin 
http.addFilterBefore(JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter::class.java)
```

# FilterChain 내에서의 예외 처리

만약 필터 단에서 `Exception` 이 발생할 경우는 어떻게 될까요? 이 경우에는 따로 핸들링 해 주어야 합니다. 필터 레이어 상에서 나올 수 있는 예외를 처리해주는 컴포넌트가 존재합니다. `AuthenticationEntryPoint` 죠. 

Spring Security 5 이후로부터는 따로 이런 필터를 구성해줄 필요는 없고 Kotlin DSL 로 구성할 수 있습니다. 

```kotlin
@Bean
fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
    http {
        authorizeHttpRequests {
            authorize("/public/**", permitAll)
            authorize(anyRequest, authenticated)
        }
        exceptionHandling {
            authenticationEntryPoint { request, response, authException ->
                response.status = HttpServletResponse.SC_UNAUTHORIZED
                response.writer.write("Unauthorized access")
            }
            accessDeniedHandler { request, response, accessDeniedException ->
                response.status = HttpServletResponse.SC_FORBIDDEN
                response.writer.write("Access Denied")
            }
        }
    }
    return http.build()
}
```

위의 예시에서는 `AuthenticationEntryPoint` 와 `AccessDeniedHandler` 를 Kotlin DSL 로 구성 한 예시입니다. `AuthenticationEntryPoint` 는 FilterChain 에서 AuthException 이 발생한 경우 동작합니다.  
`AccessDeniedHandler` 도 마찬가지로 AccessDeniedException 이 발생한 경우 동작합니다. 

혹시 두 예외 외에 다른 에외 (`IllegalStateException` 등이 발생했다면 어떻게 해야 할까요?) 이 경우에는 커스텀을 해야 하죠. 구조 그림에서 볼 수 있다시피 `ExceptionTranslationFilter` 에서 부터 `AuthenticationEntryPoint` 와 엮이기 시작하죠.  
그럼 해당 필터 이전에 이런 Exception 들을 처리해주는 필터를 하나 구현하면 다른 Exception 들을 처리할 수 있습니다. 

```kotlin 
@Component
class GlobalSecurityExceptionFilter : OncePerRequestFilter() {
    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        try {
            filterChain.doFilter(request, response)
        } catch (ex: AuthenticationException) {
            // 인증 관련 예외 처리
            response.status = HttpServletResponse.SC_UNAUTHORIZED
            response.writer.write("Authentication failed: ${ex.message}")
        } catch (ex: AccessDeniedException) {
            // 권한 관련 예외 처리
            response.status = HttpServletResponse.SC_FORBIDDEN
            response.writer.write("Access denied: ${ex.message}")
        } catch (ex: Exception) {
            // 그 외 모든 예외 처리
            response.status = HttpServletResponse.SC_INTERNAL_SERVER_ERROR
            response.writer.write("Internal Server Error: ${ex.message}")
        }                    
    }
}
```
마찬가지로 FilterChain 에 등록하면 동작하죠. `ExceptionTranslationFilter` 의 앞단에 배치해야 합니다. 해당 Filter 의 책임을 대신할테니까요. 
```kotlin
@Bean
fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
    http {
        authorizeHttpRequests {
            authorize("/public/**", permitAll)
            authorize(anyRequest, authenticated)
        }
        exceptionHandling {
            authenticationEntryPoint { request, response, authException ->
                response.status = HttpServletResponse.SC_UNAUTHORIZED
                response.writer.write("Unauthorized access")
            }
            accessDeniedHandler { request, response, accessDeniedException ->
                response.status = HttpServletResponse.SC_FORBIDDEN
                response.writer.write("Access Denied")
            }
        }
    }

    // Global Exception Handling 필터 추가
    http.addFilterBefore(GlobalSecurityExceptionFilter(), ExceptionTranslationFilter::class.java)

    return http.build()
}
```

# Spring Security 가 지원하는 인증 방식들

가장 대표적으로는 Form-Based Authentication(Form 기반 인증) 입니다. 전통적인 HTML 로그인 Form 을 사용하여 사용자를 인증하죠. 

```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http {
            formLogin {
                loginPage = "/login" // 커스텀 로그인 페이지
                defaultSuccessUrl("/home", true) // 로그인 성공 후 이동 페이지
                permitAll()
            }
            authorizeHttpRequests {
                anyRequest().authenticated()
            }
        }
        return http.build()
    }
}
```

다음으로는 HTTP Basic Authentication, 즉 HTTP Authorization 헤더를 통해 Base64 인코딩 된 인증 정보를 전달하는 방식입니다.  
이 방법은 REST API 에 적합하지만, 매 요청마다 인증 정보가 전달되므로 보안 측면에서 주의해야 합니다. 

```kotlin
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .httpBasic(Customizer.withDefaults()) // HTTP Basic 인증 활성화
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```
```http
GET /api/resource HTTP/1.1
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=  // Base64(username:password)
```

또한 Spring 에서는 JWT 인증 또한 제공하고 있습니다. 사실 우리가 직접 JWT 인증 필터를 구현해야 할 필요는 없습니다. 만약 OAuth2 인증 서버가 외부에 배치 되어있는 경우라면, OAuth2 Resource Server Dependency 를 통해 JWT 토큰 인코딩 및 디코딩 등을 쉽게 구현할 수 있습니다. 

`sprring-security-oauth2-resource-server` 내에는 JWT 토큰을 Spring Security Authentication 객체로 변환하는 `JwtAuthenticationConverter`, JWT 의 Claim 중 scope (또는 scp) 를 권한으로 부여 해주는 `JwtGrantedAuthoritiesConverter`, JWT 인증 케이스에 맞게 수정 된 `accessDeniedHandler`, `authenticationEntryPoint` 가 구성 되어있습니다. 


```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http {
            authorizeHttpRequests {
                requestMatchers("/api/**").authenticated()
                anyRequest().permitAll()
            }
            oauth2ResourceServer {
                jwt {} // JWT 인증 활성화
            }
        }
        return http.build()
    }
}
```
```http
GET /api/protected HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

또한 Google, Facebook, Github 등 외부 인증 제공자를 사용한 OAuth2 / OpenID Connect (OIDC) 인증을 지원합니다. 
이 경우 `OAuth2LoginConfigurer` 또는 `OAuth2ResourceServerConfigurer` 를 사용합니다. Spring Security 에서는 기본적으로 Google, Facebook, Github 와 같은 OAuth2 제공자를 자동 설정 가능합니다. 

```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http {
            oauth2Login {} // OAuth2 로그인 활성화
            authorizeHttpRequests {
                anyRequest().authenticated()
            }
        }
        return http.build()
    }
}
```

만약 조직에서 LDAP 디렉토리를 사용하고 있다면, `LdapAuthenticationProvider` 를 사용하면 됩니다. 

```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http {
            authorizeHttpRequests {
                anyRequest().authenticated()
            }
            ldapAuthentication {} // LDAP 인증 활성화
        }
        return http.build()
    }
}

```
```shell
dc=example,dc=com
│
├── ou=users
│   ├── uid=user1 (cn=User One)
│   ├── uid=user2 (cn=User Two)
│
└── ou=groups
    ├── cn=admin
    ├── cn=user
```

최근에 많이 사용하는 WebAuthn(FIDO2) 또한 지원합니다. Spring Security 6.4 부터 기본적으로 지원하죠. FIDO2 를 사용한다면, 브라우저에서 생체인증(지문, 얼굴) 또는 보안 키 로그인을 구현할 수 있습니다.  

```kotlin
@Configuration
@EnableWebSecurity
class SecurityConfig {

    @Bean
    fun securityFilterChain(http: HttpSecurity): SecurityFilterChain {
        http {
            authorizeHttpRequests {
                anyRequest().authenticated()
            }
            webauthn {} // WebAuthn 인증 활성화
        }
        return http.build()
    }
}
```

앞서 말씀드린 Custom Filter 를 통해 다중 인증 (Multi Factor Authentication, MFA) 또한 구현할 수 있습니다.  
일반적으로는 Username + Password + OTP (One Time Password) 를 조합하는 형태로 만들 수 있죠. 

```kotlin
@Component
class OtpAuthenticationFilter : OncePerRequestFilter() {

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        val otp = request.getHeader("X-OTP")
        if (otp.isNullOrBlank() || !validateOtp(otp)) {
            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Invalid OTP")
            return
        }
        filterChain.doFilter(request, response)
    }

    private fun validateOtp(otp: String): Boolean {
        return otp == "123456" // 실제 환경에서는 OTP 서비스와 연동 필요
    }
}
```

마지막으로 API Key 인증입니다. `X-API-KEY` 와 같은 헤더를 사용해서 특정 서비스만 접근을 허용시킬 수 있죠. 이 경우에는 `OncePerRequestFilter` 를 통해 API 키 검증 필터를 검증할 수 있습니다. 

```kotlin 
@Component
class ApiKeyFilter : OncePerRequestFilter() {

    private val API_KEY = "my-secure-api-key"

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        val apiKey = request.getHeader("X-API-KEY")
        if (apiKey != API_KEY) {
            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Invalid API Key")
            return
        }
        filterChain.doFilter(request, response)
    }
}
```

정리하면 현재 Spring Security 6.4 기준으로는 아래의 인증 방식들을 지원합니다. 

|인증 방식|설명|주 사용 사례|
|---------|----|------------|
|FormLogin|웹 애플리케이션 로그인|전통적인 웹 애플리케이션|
|Basic Auth|HTTP 헤더 기반 인증|내부 시스템, API 인증|
|JWT|Stateless한 토큰 기반 인증|REST API, 모바일 앱|
|OAuth2|외부 인증 제공자 사용|소셜 로그인, SSO|
|LDAP|조직의 LDAP 디렉터리 활용|기업 내부 시스템|
|Passkey|비밀번호 없는 인증 (WebAuthn)|생체인증 기반 로그인|
|MFA|다중 인증 (OTP 추가)|보안이 중요한 서비스|
|API Key|특정 클라이언트만 인증|마이크로서비스, API|


# Outro

Spring Security 는 다양한 인증 방식을 제공하는 라이브러리입니다. 하지만, 만약 우리가 인증서버를 따로 분리한 경우가 아니라면, Spring Security 내에 선언 해 둔 FilterChain 들이 복잡하게 엮이는 경우가 반드시 생깁니다.  

예를 들어 하나의 REST API 서버에서 다양한 인증 방식들을 한번에 적용시키려고 하다보면 필연적으로 수 많은 FilterChain 들이 생성됩니다. 이 경우 URI 규칙이 복잡해질 수 있겠죠. 또한 Configuration 코드들이 복잡해질 수 있습니다. 

개인적인 경험을 토대로 말씀 드리자면, 인증 토큰을 발급하는 서버와 Resource 서버를 분리하는 패턴이 가장 관리하기는 편했습니다. 꼭 Micro Service 가 아니어도 인증과 Resource 서버정도만 분리 해 두어도 다양한 인증들을 추가할 때 코드가 복잡하게 꼬이는 것을 방지할 수 있습니다. 책임이 명확해지니까요. 

또한 상황에 따라선 FilterChain 을 상세하게 커스텀하고 싶을 수도 있죠. 이 경우에는 반드시 Spring Security 가 좋은 선택지는 아닐 수 있습니다. 라이브러리를 쓴다는 것은 해당 라이브러리의 규칙에 따라야 한다는 의미기 때문입니다. 정말 심플하게 Filter 몇개만 배치하고 싶은 경우엔 오히려 Spring Security 를 사용하는 데 들어가는 러닝커브, 코드 작성 공수들이 비용이 될 수 있습니다. 

지금까지 Spring Security 에 대해 다뤄보았습니다. 이 한편의 포스팅으로 사실 모든 것을 다루지는 못했습니다. 대략적인 기능의 흐름과 제공하는 기능들에 대해, 사용법에 대해 온보딩 하는 데 도움이 되셨을까요? 혹시나 잘못 된 내용이 있다면 피드백 꼭 부탁드립니다 😊.

다음 알고쓰자 Spring 의 주제는 Auto Configuration 입니다. 기대해주세요 🤟.

# Reference

- [https://spring.io/projects/spring-security](https://spring.io/projects/spring-security)
