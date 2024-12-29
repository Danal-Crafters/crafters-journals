[ 크래프터스 : \[알고쓰자\] Spring - 02 업데이트 내역과 함께하는 변천사 알아보기 ]
===============================================================================================

<div align="center">   
  <img src="https://github.com/user-attachments/assets/38ae7c1f-9724-4658-9ff2-2bd282c9bb47" width="800"/>  
</div>

Intro 
=====

현재 많은 개발자 분들이 Spring Boot 2.x 혹은 3.x을 사용하고 계실거에요.
저희 팀은 Spring Boot 3.x, JDK 17 을 채택하여 사용하고 있죠.\
전 Spring Boot 2.x 에서 3.x 로 넘어가는 시점에서 Spring Boot 에
입문했어요. 갑자기 Spring Framework 6 버전 업데이트 소식과 함께 Spring
Boot 3 릴리즈 소식을 들었던 게 엊그제같네요. Spring Security 쪽 코드가
상당히 바뀌었고, 막 입문해서 공부하던 저에겐 양날검 같은 소식이었어요.
공부할 때 보던 코드들이 최신 버전에선 전혀 쓸 수 없었으니까요.

> 이제와서 생각해보면\...그깟 설정 코드들이 뭐가 그렇게 중요했을까 싶어요.  
> 중요한 건 Spring Security 아키텍처의 이해도였는데...  
> 그 당시엔 코드 하나하나가 너무 생소했어서 뭔가 @Bean 으로 등록하는 필터체인이 새롭게 다가왔던 기억이 나네요.  
> 이상 흔한 주니어 개발자의 회상이었습니다.

과거를 회상하며 커피를 한잔하던 도중 과연 내가 Spring의 변천사를
**알고쓰고** 있는 지 의문이 들었습니다. Spring Framework 6 부터
입문했다고 해도 할 말이 없는 상황이었는데 그 전의 Spring 은 어땠는 지를
제대로 알고 쓰지 않았단 생각이 들었습니다.

이번 주제는 Spring 업데이트 내역과 함께하는 변천사입니다. 재밌게
읽어주세요.

주요 업데이트 내역
==================

변천사를 알아보기 전에 업데이트 내역을 한번 정리해보면 아래와 같아요.

-   1.0 : 2004년 3월 : XML

-   2.0 : 2006년 10월 : XML 편의기능 지원

-   3.0 : 2009년 12월 : Java 코드를 통한 Config

-   4.0 : 2013년 12월 : Java 8 지원

-   5.0 : 2017년 9월 : Spring Framework 5.0, Spring Boot 2.0 출시,
    Reactive Programming 지원

-   6.0 : 2022년 11월 : Spring Framework 6.0, Spring Boot 3.0 출시

아마 주로 쓰고 있을것이라 생각되는 Spring Boot 는 2010년에 나왔어요.

> **여기서 잠시! Spring Boot 가 뭔가요?**  
> Spring 기반 애플리케이션 개발을 빠르게 시작할 수 있도록 도와주는
도구에요.  
> Spring Boot를 쓰게 된다면, 여러 고민을 할 필요 없이 일단
프로젝트를 셋업하고, 필요할 때 커스터마이징을 할 수 있다는 장점이
있어요.  
> Spring Framework 그 자체를 그대로 사용하려면 개발자가 직접 판단을 통해 프로젝트의 구성을 결정해야 해요.  
> 예시로 JSON 처리와 관련 된 라이브러리를 선택할 때 Spring Framework 만 사용한다면 Jackson, GSON 등 여러가지 선택지 중에 하나를 고르고, 버전 호환성을 고민해야 해요.  
> 또한 web.xml 설정, Web Application Server 설정
등 서비스를 배포하기 위해 복잡한 설정을 건드려야 해요.  
> Spring Boot 는 이런 고민을 덜어주죠. 미리 필요하다고 판단되는 여러 라이브러리들을 미리 정해주고 상호 호환되는 버전을 제공해줘요. 게다가 WAS 또한 내장되어있어서 단일 jar 파일로 배포할 수 있죠.

주요 패치내역은 위와 같지만, Spring Framework 가 가지는 핵심 원리들은
크게 변화하지 않았어요.

그럼 지금부터 버전 별로 핵심적인 키워드들을 한번 짚어 가볼게요.

1.0 \~ 2.0 : XML 
================

초기의 Spring 은 XML 을 통해 프로젝트의 셋업을 진행했어요.

Spring 프로젝트를 처음 셋업하면 web.xml, root-context.xml,
servlet-context.xml 이 세가지의 파일이 생성됩니다. Spring Boot 로
프로젝트를 셋업한다면, 사실 직접적으로 볼 일은 없겠지만 알고 넘어가는 게
좋겠죠? 각각의 역할을 정리하면 아래와 같아요.

-   web.xml

    -   애플리케이션의 배포 및 구성 정보를 정의합니다.

    -   주로 DispatcherServlet 을 설정하는 데
        사용됩니다[.]{.inline-comment-marker
        ref="fd7c6ee8-3098-4da6-ab1e-aee7b75ee6e1"}

    -  ```xml
        <web-app xmlns="http://java.sun.com/xml/ns/javaee"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
                                     http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
                 version="3.0">

            <!-- 1. 서블릿 정의 -->
            <servlet>
                <servlet-name>dispatcher</servlet-name>
                <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
                <load-on-startup>1</load-on-startup>
                <init-param>
                    <param-name>contextConfigLocation</param-name>
                    <param-value>/WEB-INF/spring/dispatcher-servlet.xml</param-value>
                </init-param>
            </servlet>

            <!-- 2. 서블릿 매핑 -->
            <servlet-mapping>
                <servlet-name>dispatcher</servlet-name>
                <url-pattern>/</url-pattern>
            </servlet-mapping>

            <!-- 3. 필터 정의 -->
            <filter>
                <filter-name>encodingFilter</filter-name>
                <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
                <init-param>
                    <param-name>encoding</param-name>
                    <param-value>UTF-8</param-value>
                </init-param>
                <init-param>
                    <param-name>forceEncoding</param-name>
                    <param-value>true</param-value>
                </init-param>
            </filter>

            <!-- 4. 필터 매핑 -->
            <filter-mapping>
                <filter-name>encodingFilter</filter-name>
                <url-pattern>/*</url-pattern>
            </filter-mapping>

            <!-- 5. 리스너 정의 -->
            <listener>
                <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
            </listener>

            <!-- 6. 컨텍스트 초기화 파라미터 -->
            <context-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>/WEB-INF/spring/root-context.xml</param-value>
            </context-param>

            <!-- 7. 세션 타임아웃 설정 -->
            <session-config>
                <session-timeout>30</session-timeout>
            </session-config>
        </web-app>
        ```

-   root-context.xml

    -   애플리케이션 전반에 걸쳐 사용되는 공통 빈들을 정의하는
        파일입니다.

    -   DB Connection, Security, Service 계층 등을 정의합니다.

    -   ``` xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:context="http://www.springframework.org/schema/context"
               xmlns:tx="http://www.springframework.org/schema/tx"
               xsi:schemaLocation="
                   http://www.springframework.org/schema/beans 
                   http://www.springframework.org/schema/beans/spring-beans.xsd
                   http://www.springframework.org/schema/context 
                   http://www.springframework.org/schema/context/spring-context.xsd
                   http://www.springframework.org/schema/tx 
                   http://www.springframework.org/schema/tx/spring-tx.xsd">

            <!-- 1. Component Scan -->
            <context:component-scan base-package="com.example.service" />

            <!-- 2. DataSource 설정 -->
            <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
                <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mydb" />
                <property name="username" value="root" />
                <property name="password" value="password" />
            </bean>

            <!-- 3. Hibernate SessionFactory 설정 -->
            <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
                <property name="dataSource" ref="dataSource" />
                <property name="packagesToScan" value="com.example.entity" />
                <property name="hibernateProperties">
                    <props>
                        <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                        <prop key="hibernate.show_sql">true</prop>
                    </props>
                </property>
            </bean>

            <!-- 4. 트랜잭션 매니저 설정 -->
            <bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
                <property name="sessionFactory" ref="sessionFactory" />
            </bean>

            <!-- 5. 트랜잭션 관리 활성화 -->
            <tx:annotation-driven transaction-manager="transactionManager" />

            <!-- 6. 서비스 빈 등록 -->
            <bean id="myService" class="com.example.service.MyServiceImpl">
                <property name="someDependency" ref="someDependencyBean" />
            </bean>

        </beans>
        ```

-   servlet-context.xml

    -   특정 서블릿과 관련된 설정을 관리하는 파일입니다.

    -   주로 ViewResolver, Controller 등의 웹 계층 설정을 정의합니다.

    -   ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:context="http://www.springframework.org/schema/context"
               xmlns:mvc="http://www.springframework.org/schema/mvc"
               xsi:schemaLocation="
                   http://www.springframework.org/schema/beans 
                   http://www.springframework.org/schema/beans/spring-beans.xsd
                   http://www.springframework.org/schema/context 
                   http://www.springframework.org/schema/context/spring-context.xsd
                   http://www.springframework.org/schema/mvc 
                   http://www.springframework.org/schema/mvc/spring-mvc.xsd">

            <!-- 1. 컴포넌트 스캔 -->
            <context:component-scan base-package="com.example.controller" />

            <!-- 2. Spring MVC 활성화 -->
            <mvc:annotation-driven />

            <!-- 3. 뷰 리졸버 설정 -->
            <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                <property name="prefix" value="/WEB-INF/views/" />
                <property name="suffix" value=".jsp" />
            </bean>

            <!-- 4. 정적 자원 핸들링 -->
            <mvc:resources mapping="/resources/**" location="/resources/" />

            <!-- 5. 메시지 소스 설정 (다국어 지원) -->
            <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
                <property name="basename" value="messages" />
            </bean>

            <!-- 6. 기본 핸들러 매핑 -->
            <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />

            <!-- 7. 디폴트 서블릿 핸들러 활성화 (정적 파일 처리) -->
            <mvc:default-servlet-handler />

        </beans>
        ```

> **Servlet? 그게 뭐죠? DispatcherServlet 은 또 무엇이구요?**  
> Java 기반 웹 애플리케이션에서 서버 측에서 동작하는 프로그램입니다.  
> 주로 클라이언트의 HTTP 요청을 처리하고 HTTP 응답을 생성하는 역할을 해요.  
> Thymeleaf 와 같은 템플릿 엔진을 활용해서 웹페이지를 생성하거나, JSON
형태의 응답을 생성하는 데 사용됩니다.  
> DispatcherServlet 은 Spring 에서 클라이언트의 요청을 처리해서, 적절한 컨트롤러로 요청을 분배 (Dispatch) 하는 역할을 합니다.  
> 이번 포스팅의 주제는 Spring 아키텍처에 대한 내용이 아니니
DispatcherServlet 에 대한 내용은 여기까지 하겠습니다.


잠깐 예시들을 보여드렸는데, 저 설정들이 익숙하신 분들도 계실거에요.
Spring Framework 를 쓰기 전에 학부에서 JSP 공부할 때 저런 파일들을
하나하나 셋업했던 기억이 나네요.

3.0 : Java 기반 Configuration
=============================

Spring Framework 3.0 부터 Java 기반 Configuration 을 지원하기
시작했습니다.

Spring Framework 3.0 은 우리에게 익숙할 수 있는 버전이죠? 토비의
스프링이 3.1 버전 기준으로 작성 되어있죠.

```java

@Configuration
public class AppConfig {
    private @Value("#{jdbcProperties.url}") String jdbcUrl;
    private @Value("#{jdbcProperties.username}") String username;
    private @Value("#{jdbcProperties.password}") String password;

    @Bean
    public FooService fooService() {
        return new FooServiceImpl(fooRepository());
    }

    @Bean
    public FooRepository fooRepository() {
        return new HibernateFooRepository(sessionFactory());
    }

    @Bean
    public SessionFactory sessionFactory() {
        // wire up a session factory
        AnnotationSessionFactoryBean asFactoryBean =
            new AnnotationSessionFactoryBean();
        asFactoryBean.setDataSource(dataSource());
        // additional config
        return asFactoryBean.getObject();
    }

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(jdbcUrl, username, password);
    }
}
```

아마 Spring Boot 로 입문했던 저 같은 사람들은 이렇게 Java 기반 Config 이
익숙할거에요. 저도 이게 훨씬 익숙하네요.

```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver" />
```
```java
@Bean
public AcceptHeaderLocaleResolver localeResolver() {    
    return new AcceptHeaderLocaleResolver();
}
```

xml 을 통해 bean 을 등록하려면 Bean ID 와 class path 를 입력 해 주어야
했어요. Java 기반 Config 은 훨씬 직관적으로 Bean 컴포넌트를 명세하고
등록할 수 있죠.


> **`@Bean` 과 `@Component` 의 차이는 개발자가 얼마나 개입하느냐의 차이에요.**  
> `@Bean` 은 RestTemplate, DataSource 등 개발자가 개입하기 어려운 외부
라이브러리 Bean 을 생성할 때 사용해요.   
> `@Component` 는 개발자가 코딩 한
컴포넌트를 Bean 으로 등록할 때 사용하죠.


```java
@Component
public class ExampleComponent () {
  //TODO
}
```

4.0 : Java 8
============

Java 8 버전은 제가 학부때 처음 JDK 를 설치했을 때 사용했 던 버전이에요.
생각보다 Java 8 이 릴리즈 된 게 오래 전의 일은 아니에요. 2014년
릴리즈니까 이제 10년이 지났죠.

> 2030 년 까지 지원되는 현재 많은 유저들이 아직도 사용중인 Java
> 버전이에요. 사견으로는 Java 8에서도 상당히 많은 변화가 있었고, 그
> 당시에 Java 8 마이그레이션 했던 개발자들이 상당히 고생해서 그런 것
> 아닐까 하는 생각도 듭니다. (두번은 하기싫어요!!! 💦 )

Java 8 에서 볼 수 있는 가장 큰 변화점들에 대해 설명드리고 넘어갈게요.


```java
class Main {
    public static void main(String[] args) {
        List<Integer> numList = Arrays.asList(0,1,2);
        numList.forEach(x -> System.out.println(x));
    }
}
```

**먼저 Lambda 식이에요.** 이전에 익명함수라고 불리던 기능이 있었죠. Java
8 부터는 이 익명함수를 Lambda 표현식으로 작성할 수 있어요.


```java
public interface IExample {
    int getMax(int x, int y);
}

//Java 8  이전의 익명함수
public static void main(String[] args) {
    IExample iExample = new IExample() {
        @Override
        public int getMax(int x, int y) {
            return (x > y) ? x : y;
        }
    };
    System.out.println("결과는 " + iExample.getMax(10,20) + "입니다.");
}

//Java 8 부터의 익명함수
public static void main(String[] args) {
    IExample iExample = (x, y) ->x > y ? x : y;
    System.out.println("결과는 " + iExample.getMax(10,20) + "입니다.");
}
```

**다음으로 Optional 클래스 기능이에요.**   
Null 값이 될 수 있는 객체를
Optional 객체에 담을 수 있어요. Null 값을 직접적으로 건드리는 경우 Java
에서는 `NullPointerException` 이 발생할 수 있지만, 언어의 특성 상 이
NPE를 컴파일 단계에서 피할 수 있는 방법이 당시엔 없었어요. Optional
객체를 사용하면, Nullable 한 객체에 대한 접근을 보호함과 동시에 Stream
API 와 연계한 로직을 처리할 수 있어요.


```java
class Main {
    public static void main(String[] args) throws Exception {
       Car car = new Car();
       
       // 빈 생성자로 생성 된 객체의 값에 직접 접근하면 NPE가 발생할 수 있습니다.
       System.out.println(car.getPerson().getName())
       
       // Optional 을 연계하면 NPE 가 발생하지 않습니다. 
       System.out.println(Optional.of(car)
               .map(Car::getPerson)
               .map(Person::getName)
               .orElse("사용자 없음"));
    }
}

class Person {
    private String name;
    //getter, setter
}

class Car {
    private String name;
    private Person person;
   //getter, setter
}
```

**다음으로 Stream API 이에요.**   
Stream API 는 컬렉션 데이터를 처리하기
위해 사용되는 Java 8 에서 추가 된 신규 기능이에요. 데이터의 연속적인
흐름을 표현한다는 의미에서 Stream 이라는 이름이 생겼다고 하네요. for
Loop 반복문을 통한 데이터 처리보다 간결하고 가독성이 높은 코드를 작성할
수 있어요. 또한 데이터를 가공하는 데 쓰이는 다양한 기능들을 제공해요.

Lambda 식과는 일맥상통 한다고 볼 수 있는 기능인데, Stream API 는 컬렉션
데이터에 대핸 Stream 을 생성시켜서 연산을 처리하는데, 이 연산 처리
과정에서 Lambda 식을 연동시킬 수 있어요.

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> result = names.stream()
                          .filter(name -> name.startsWith("A"))
                          .collect(Collectors.toList());
```


`filter`, `map`, `sorted`, `distinct` 등의 중간 연산 (Intermediate
Operators) 사이에 들어가는 로직은 Lambda 식으로 작성할 수 있어요. 또한
Stream의 마지막 최종 연산 `collect`, `forEach`, `reduce`, `count` 등
(Terminal Operations) 을 통해 우리가 원하는 결과로 값을 변환시킬 수
있죠.

또한 Stream API 는 병렬처리를 지원해요. `pararellStream` 을 생성해서
병렬적으로 데이터를 처리할 수 있죠. 내부적으로 Fork/Join 프레임워크를
사용하여 여러 쓰레드에서 데이터를 분할하고 처리한 후 결과를
결합시켜줘요.


```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> result = names.parallelStream()
                           .filter(name -> name.startsWith("A"))
                           .collect(Collectors.toList());
```


**다음으로 LocalDateTime 이에요.**  
이 기능에 대해선 설명을 짧게
정리하겠습니다. 불변 시간정보를 가지고 있는 객체고 기존의 DateTime, Date
를 대체하는 기능이죠.

**마지막으로 interface default, static 메소드에요.**   
인터페이스 자체는
구현부가 없죠. 하지만 Java 8 부터는 default, static 지시어로 된 메소드를
구현할 수 있어요.

차이점이 있다면 default 는 재정의가 가능하고, static 은 재정의가
불가능해요.


```java
public interface Calculator {

    default int sumDefault(int x, int y) {
        return x + y;
    }

    static int sumStatic(int x, int y) {
        return x + y;
    }
}
```


> 여기까지 정리 해 보니 왜 자꾸 Java8 에서 안 넘어 가시려는 지 알것 같네요. 이거 어느세월에 마이그레이션 하셨데요...

5.0 : Reactive Programming
==========================

Spring Framework 5.0 이 출시 되었습니다. Spring Boot 는 2.0 이 출시 된
시점이죠.

이 시점부터 Spring 은 Reactive Programming 을 지원하기 시작했죠. 찬호
대리님이 지난 호에 Webflux 에 대한 글을 작성하셨죠.

Reactive Programming 을 지원하기 전 Spring 은 전형적인 MVC 패턴으로
요청을 처리했습니다. 추후 Spring 의 구조에 대해 포스팅을 할 때 더 자세히
다루겠지만, Spring 은 Model 의 데이터를 View 에 출력하고 Controller 를
통해 View 와 Model 사이의 요청을 관리해요.

여기까지만 봤을 땐 ***당연한 것 아닌가...***라는 생각이 드시겠지만, 이
패턴은 대규모의 요청이 들어왔을 때 시스템이 부하를 주는 구조라는 단점이
존재해요.

Spring MVC 에서는 매 요청마다 스레드를 할당 시켜요. 요청에 대한 응답이
나오기 전까지 스레드는 유지되죠. 이를 블로킹 방식이라고 해요. 요청이
늘어날 수록 할당 되는 스레드도 많아지겠죠. 언젠가 이 스레드가 고갈되게
된다면 더이상 요청을 정상적으로 처리할 수 없겠죠.

Reactive Programming 을 사용하려면 Spring Webflux 를 사용하실 수 있어요.
Webflux 는 비동기 논블로킹 방식을 사용하는데, 요청이 들어오면 즉시
처리할 수 없다면 이벤트루프에 등록하고, 이후 작업이 완료되면 이벤트를
발생시켜 처리하는 방식이에요. 해당 방식은 스레드를 적게 사용하고, 많은
요청을 효과적으로 처리할 수 있죠.

2017년 시점에서 기억을 되돌아보면 저는 군대 ***(해군병 639기 입니다.
해군이시라면 함께 소주와 함께 군대 얘기를....크흡)*** 에 있긴 했지만,
이미 대부분의 사람들의 손에는 스마트폰이 쥐어 진 상태였죠. 꼭 최신
플래그쉽이 아니더라도 그 당시엔 저가형 스마트폰도 상당히 많았던 기억이
나요. 이미 그 시점에서도 유튜브나 SNS는 핫 했죠. 인터넷 서비스의
접근성이 낮아지는 만큼 트래픽은 늘어났고, Spring 생태계에도 이러한
대용량 트래픽을 적은 비용으로 해결할 수 있는 방법들에 대한 요구가
늘어났었다고 해요.

Spring Webflux 와 MVC의 코드를 한번 비교해볼게요. /hello/{name} 경로에
대한 요청을 받으면, Service 레이어에서 "Hello, {name}!" 이라는 데이터를
반환하는 예시에요. **편의상 이번엔 코틀린으로 작성 해 보았습니다.**


```kotlin
// Spring MVC Controller
@RestController
@RequestMapping("/mvc")
class HelloController(private val helloService: HelloService) {

    @GetMapping("/hello/{name}")
    fun sayHello(@PathVariable name: String): ResponseEntity<String> {
        val message = helloService.getGreeting(name)
        return ResponseEntity.ok(message)
    }
}

// Spring MVC Service
@Service
class HelloService {

    fun getGreeting(name: String): String {
        // 비즈니스 로직 처리
        return "Hello, $name!"
    }
}
```

우리가 알고있는 전형적인 코드죠. 요청이 들어오는 순간 스레드가 할당되어
코드들을 실행해요.


```kotlin
// Spring WebFlux Controller
@RestController
@RequestMapping("/webflux")
class HelloReactiveController(private val helloService: HelloReactiveService) {

    @GetMapping("/hello/{name}")
    fun sayHello(@PathVariable name: String): Mono<String> {
        return helloService.getGreeting(name)
    }
}

// Spring WebFlux Service
@Service
class HelloReactiveService {

    fun getGreeting(name: String): Mono<String> {
        // 비즈니스 로직 처리
        val message = "Hello, $name!"
        return Mono.just(message)
    }
}
```

요청이 들어오면 비동기적으로 `sayHello` 메소드가 실행되고, Mono 가
완료될 때 까지 스레드를 점유하지 않아요. MVC에서는 커피를 반드시
그자리에서 받아야 했다면, Webflux 에서는 커피를 주문하고 기다릴 수
있는거죠.

Webflux에서는 Mono, Flux 이 두가지를 요청의 최종 리턴타입으로 가질 수
있어요. Mono 는 최대 하나, Flux는 0개 이상의 요소를 리턴하는 경우에
사용하죠.

```kotlin
fun findUserById(id: String): Mono<User> {
    return Mono.just(User(id, "Alice"))
}

fun findAllUsers(): Flux<User> {
    return Flux.just(
        User("1", "Alice"),
        User("2", "Bob"),
        User("3", "Charlie")
    )
}
```

6.0 : 그리고 지금
=================

2022년 Spring Framework 6.0 이 출시되었죠. Spring Boot 또한 3.0 이
출시되었습니다. 지금 저희 팀에선 3.1.X를 사용하고 있죠.

SpringFramework 6.0 이상부터 최소 JDK 17 이상이 필요합니다. 또한 Java EE
를 사용하던 많은 라이브러리들이 Jakarta EE 로 변환되었습니다. 특히 JPA
와 관련 된 코드들은 Java EE 가 아닌 Jakarta EE 로 마이그레이션 해야
했어요.


> **기존의 Java EE 는 왜 Jakarta EE 로 대체되었을까요?**  
> Oracle 이 2017 년 Jave EE 8 릴리즈를 마지막으로 이클립스 재단에 Java EE 프로젝트를 이관했기 때문이에요.  
> Java EE 는 1999년 썬 마이크로시스템즈가 발표한 소프트웨어 개발 표준
플랫폼이에요.   
> Oracle 이 썬 마이크로시스템즈를 인수하면서 Java EE
프로젝트의 주도권을 가지고 있었으나, Spring Framework 등의 오픈소스 소프트웨어가 발전하면서 상업적 목적으로의 Java EE 프로젝트 개발에 메리트가 줄었기 때문이에요.  
> 이클립스 재단에 Java EE 가 넘어간 이후로부턴 Jakarta EE 라는 이름으로 불리고 있어요.  
> Java EE 라는 이름의 상표권은 아직 Oracle 에 있기 때문이에요.


또한 XML 은 SpringFramework 6.0 이후부턴 가능하면 사라지는 추세로 변경 된다고 해요. 또한 사소한 변경으로는 HttpMethod가 Enum 이 아니라 Class 로 변경된다는 등이 있죠.

눈에 띄는 변화로는 AOT (Ahead Of Time) 엔진을 지원한다는 점인데, 기존의 JVM 방식에서는 프로그램이 올라갈 때 바이트 코드를 컴파일 해요. 

하지만 Spring AOT는 스프링 애플리케이션의 코드와 의존성을 분석하여 필요한 정보를 미리 생성된 메타데이터로 저장하고, 이후 GraalVM을 사용하여 네이티브 이미지로 컴파일할 수 있게 하죠. 이를 통해 최종적으로 생성된 애플리케이션은 JVM 없이도 실행 가능한 바이너리 파일로 만들 수 있어요.


주로 `spring-aot-maven-plugin`이나 `spring-aot-gradle-plugin`을 사용하여
AOT 빌드 프로세스를 지원해요.

JVM 을 통해 실행하는 Spring 에 비해 Spring AOT 를 사용하면 아래와 같은
이점을 가질 수 있어요요.

-   애플리케이션 실행시간 단축

    -   AOT로 생성 된 네이티브 이미지는 JIT 컴파일이 불필요해요. 이 점은
        서버리스 환경 (AWS Lambda) 에서 유리할 수 있어요.

-   낮은 메모리 사용량

    -   AOT로 생성 된 네이티브 이미지는 메모리를 덜 사용해요. 필요하지
        않은 메타데이터를 줄이고, 가비지 컬렉션 부담을 줄여
        애플리케이션이 보다 적은 메모리로 실행될 수 있어요.

-   경량화 및 배포 용이성

    -   JVM이 필요 없는 네이티브 이미지이기 때문에 배포 파일 크기가
        줄어들며, 경량 컨테이너 이미지 작성이 가능해요. 이는 클라우드
        네이티브 애플리케이션이나 컨테이너화된 애플리케이션에서 이점을
        제공해요.

-   성능 최적화

    -   GraalVM의 AOT 방식은 JIT보다 성능이 더 나은 경우가 많으며, 특히
        애플리케이션이 일정한 성능을 지속적으로 요구할 때 유용하죠.

그 외에 변동사항으로는 jakarte.servlet 은 Tomcat 11, Jetty11 을
사용하고, RPC 지원이 만료됩니다. jakarta.persistence 는 Hibernate ORM
6버전 기준으로 변경되었구요. 일부 Java EE API (EJB, JCA, JAX-WS) 는
지원이 불가능 할 수 있다 정도네요.

요약하면 SpringFramework 6.0 부터는 클라우드 네이티브 환경에 대응 한
버전이라고 할 수 있어요.

Outro
=====

지금까지 간략하게 SpringFramework 의 변천사에 대해 알아보았어요. 한번의
포스팅으로는 Spring 의 모든 변천사를 다룰 순 없겠지만, 큰 핵심 맥락들은
알아볼 수 있었어요.

프레임워크가 시간이 지날 수록 현재의 웹 서비스 환경에 좀 더 최적화
되어가는 과정을 돌아볼 수 있었어요. 실제로 가면 갈 수록 클라우드
환경에서 실행되는 서비스가 늘어나고 있잖아요? 상황에 따라 Scale UP & Out
되거나 빠른 배포를 지원하는 배포 파이프라인을 구축할 수 있는 클라우드
환경은 서비스의 개발 속도를 빠르게 하고, 서비스의 유지보수 비용을
줄여주죠.

글을 쓰기 전 까지는 사실 SpringFramework 6.0 이상을 워낙 당연하다시피
쓰고 있어서 얼마나 해당 버전이 클라우드 환경에 최적화 되어 있는 지 잘
와닫 지 않았어요. Docker 와 같은 컨테이너 환경을 사용할 수 있을 땐 별
생각 없이 이미지 빌드 기능을 사용했었죠. 프레임워크의 개발 방향성이 기능
그 자체도 있지만, 환경에 맞춰서 변해간다는 생각을 많이 했어요. Java
버전이 올라가니까, XML 파일이 잘 안쓰이니까 프레임워크도 마찬가지로
사람들의 니즈에 맞게 변해가는 모습을 다시 되돌아보며 사람들의 니즈를
계속 관찰하는 개발자가 되어야 겠단 생각이 들었어요.

다음 포스팅의 주제는 Spring 의 구조 편입니다. 이렇게 업데이트 내역이
길고 곧 7.0 이 나오는 Spring 이라고 해도 핵심 구조가 크게 변하지는
않았어요. 과연 우리는 Spring 의 구조를 잘 알고 쓰고 있을까요?

다음 포스팅도 꽤나 고생할 것 같네요....다음달에 다시 봽겠습니다.

Reference
=========

<https://www.samsungsds.com/kr/insights/java_jakarta.html>

<https://spring.io/>

------------
<div align="center">
    
|Profile|Link|Bio|
|--|--|--|
|<img src="https://github.com/user-attachments/assets/24d3f306-8014-41ae-98a3-a2f94bf6bdb9" width="100" height="100"/> | [github](https://github.com/swj9707) <br/> [LinkedIn](https://www.linkedin.com/in/woojin-son-18989b202/) |**"천천히, 하지만 꾸준히!"**|

</div>
