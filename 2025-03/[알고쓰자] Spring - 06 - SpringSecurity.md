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

[FilterChain 이미지]()

Spring 의 FilterChain 은 웹 애플리케이션에서 들어오는 요청과 나가는 응답을 여러 단계의 필터를 통해 처리하도록 도와줍니다. 우리가 FilterChain 에 커스텀 Filter 를 넣을 수도 있죠. 

Spring Security 를 설정하면 이 FilterChain 영역에 DelegatingFilterProxy 를 통해 Security 의 FilterChainProxy 를 연결 시킵니다. 해당 FilterChainProxy 를 통해 실제로는 외부에 배치 되어있는 SecurityFilterChain 이 마치 FilterChain 내에 존재하는 것 처럼 사용할 수 있습니다. 

DelegatingFilterProxy 를 통해 FilterChain 에서 SecurityFilterChain 으로 요청이 올 타이밍에 Spring 의 영역의 자원들 (Beans 들) 을 사용할 수 있습니다. 즉 IoC 컨테이너에서 DI 를 받을 수 있단 의미죠. 

이를 통해 우리가 원하는 Filter 를 FilterChain 내에 커스텀할 수 있습니다. 

[SecurityFilterChain 이미지]()

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

# Spring Security 가 지원하는 인증 방식들



# Outro

# Reference

- [https://spring.io/projects/spring-security](https://spring.io/projects/spring-security)