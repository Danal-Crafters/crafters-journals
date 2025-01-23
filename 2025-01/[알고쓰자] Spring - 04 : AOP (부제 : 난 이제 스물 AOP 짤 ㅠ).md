Intro
=====

* * *

다시 돌아왔습니다. 알고쓰자 Spring 시리즈가 벌써 4회차 까지 가게 와버렸네요. 시간이 참 빠르네요.

> 이걸 처음 시작했을 때 얼마나 오래 하려나 싶었는데, 4개월이나 더 하게 될 줄은 몰랐습니다. 계속해서 읽어주시고 관심을 주시는 분들이 계셔서 한번 어디까지 할 수 있을 지 계속 해 보기로 했습니다.

이번 포스팅의 주제는 AOP (Aspect-Oriented Programming) 입니다. AOP 에 대해 알아보고 활용사례를 알아볼거에요.

Spring 은 Layered Architecture 를 가진다고 지난 호에 말씀 드렸죠. **사실 추상적인 개념이에요**. 위상적으로는 아파트 처럼 계층을 가지고 있겠지만, 소스코드 뭉탱이들을 모아 둔 프로젝트를 가만히 살펴보면 잘 와닫지는 않을 수도 있어요. `@Controller`, `@Service`, `@Repository` 와 같은 어노테이션들로 계층을 대강 구분할 수 있을 뿐이죠.


`@Controller`, `@Service`, `@Repository` 어노테이션 코드를 분석해보신 분들은 생각보다 별거 없다는 점을 알아보셨을 수도 있어요. 이 친구들이 특별한 기능을 가지고 있는 것 같지만 실제로는 `@Component` 어노테이션을 그저 확장시켰을 뿐이죠.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```

![image](https://github.com/user-attachments/assets/8903119a-c9a3-4d02-acaa-927c36fe406d)

그 중에서 @Controller (혹은 @RestController) 는 DispatcherServlet 의 호출을 받아서 특정 HTTP Request 에 연결 된 API 를 실행시킵니다.

어노테이션 그 자체만 보면 어떻게 이 기능이 동작하는 지 전혀 감이 오지 않을 수도 있습니다. 특히 Spring 에 입문하신 지 얼마 안되었거나, 프로그래밍에 이제 막 입문하신 분이라면 더더욱 이해가 가지 않습니다. 무슨 과정을 거치길래 이런 기능이 구현된 건지 코드단으로만 봤을 땐 잘 이해가 가지 않죠. **그저 이 짤막한 코드를 컴파일했을 뿐인데 어떤 과정을 거치길래 HTTP Request 를 받아올 수 있는걸까요?**

![image](https://github.com/user-attachments/assets/3510427e-5610-4a40-8630-b7526debfc98)


잠시 막간을 이용해서 Spring 과 같은 프레임워크를 사용하지 않고 HTTP I/O 를 처리하는 애플리케이션을 만들어보겠습니다.

```kotlin
import java.net.InetSocketAddress
import com.sun.net.httpserver.HttpServer
import com.sun.net.httpserver.HttpHandler
import com.sun.net.httpserver.HttpExchange

fun main() {
    // 서버 생성
    val server = HttpServer.create(InetSocketAddress(8080), 0)

    // 요청 핸들러 등록
    server.createContext("/hello", HelloHandler())

    // 서버 시작
    server.start()
    println("Server is running on http://localhost:8080")
}

class HelloHandler : HttpHandler {
    override fun handle(exchange: HttpExchange) {
        val response = "Hello, Kotlin!"
        exchange.sendResponseHeaders(200, response.toByteArray().size.toLong())
        exchange.responseBody.use { outputStream ->
            outputStream.write(response.toByteArray())
        }
    }
}
```

학부 시절에 아마 비슷한 코드를 작성해보신 분들도 계실겁니다.

Spring 프레임워크를 사용하지 않는다면, 직접 Java 가 제공해주는 HttpServer 클래스를 통해 요청에 대한 Handler 를 구현해야 하죠. `@Controller` 어노테이션은 여기서 11번 줄의 역할을 합니다. Dispatcher Servlet 에게 어떤 녀석이 Controller 고 어떤 녀석에게 URI 가 매핑 된 메소드가 있는 지 알려주는 역할을 하죠.

![image](https://github.com/user-attachments/assets/81a3ebf4-bc2b-4b17-8f2d-80e40815b522)


갑자기 뜬금없는 이야기가 나왔습니다만 사실 이번 포스팅은 지난 포스팅에서 이어지는 내용입니다. 지난번에 Layered Architecture 를 다뤘잖아요? 각 레이어 별로, 혹은 기능에 따라 공통적으로 구현해야 하는 로직들이 필요하게 되는 경우가 생깁니다. 만약 이 Controller 영역에서 전달되는 Request Parameter 들을 로깅하고 싶다면 어떻게 해야할까요? 매번 이 Controller 영역의 메소드들마다 Logger 를 불러와서 메소드를 실행시켜야할까요?

이번 포스팅에서는 이 공통적으로 구현해야 하는 로직을 처리하는 방법에 대해 알아보겠습니다.

**Aspect Oriented Programming (관점 지향 프로그래밍) 개요**
================================================

* * *

AOP (관점 지향 프로그래밍) 은 관점을 기준으로 다양한 기능들을 분리해서 기능을 개발하는 방법을 말합니다. 여기서 관점이란 부가기능들을 의미합니다. 앞서 말씀드린 Logging 과 관련 된 로직들을 관점이라 할 수 있어요.

> 내 Spring 애플리케이션에서는 모든 Controller 영역의 메소드가 호출될 때 파라미터를 로그로 찍고 싶어!

이런 것들을 관점이라 볼 수 있죠.

일반적인 객체지향 프로그래밍(OOP) 에서는 목적에 따라 클래스를 만들고, 클래스를 통해 생성 된 객체간의 협력을 통해 로직을 처리합니다. 문제는 객체 각각을 분리하는 건 좋지만, 이 객체들의 기능들을 어떻게 나눌 지에 대한 정의는 부족합니다. 클래스에 네이밍을 통해 해당 클래스의 목적을 부여하는 것은 좋지만, 결국 프로그래머가 직접 모든 로직을 구현해야하죠.

AOP 는 이런 문제를 해결하는 개념입니다. OOP 에서 클래스들을 목적에 따라 분리하고 로직을 분업한다면, AOP 는 파생되는 부가 로직들에 집중합니다.

![image](https://github.com/user-attachments/assets/5ed5c846-3e14-4f87-a141-a4112c95faac)


Layer 들을 분리하는 건 객체지향 프로그래밍의 관점에서 나온 개발 방법론입니다. 각 Layer 들을 구성하는 클래스들은 서로를 직접 참조하지않고, 오직 인터페이스만을 통해 소통합니다.

하지만 이런 와중에도 각각의 Layer들이 공통적으로 가지는 로직들이 분명히 존재합니다. 앞서 말씀드린 Logging 도 마찬가지죠. Presentation Layer (Controller) 에서 메소드 파라미터에 대해 로깅을 하는 것과, Business Layer (Service) 에서 메소드 파라미터에 로깅을 하는 것 이 두가지는 차이가 있을까요? 일반적으로 구현한다고 생각해보면 Logger 인스턴스를 불러와서 콘솔에 무언가 찍을겁니다.

val log = getLogger()
log.info("Viva Danal!!")

간단히 생각하면 **그냥 logger 불러와서 찍으면 되는 것 아닌가?** 싶겠지만 모든 메소드에 일일히 `log.info` 메소드를 호출하고 싶어하는 사람은 없을 것 같습니다.

이렇게 Layer들을 관통하는 부가기능 로직들을 횡단 관심사 (Cross Cutting concerns) 라고 합니다. 대표적으로 Logging, Transaction, Security가 있죠. 하나의 트랜잭션은 각 Layer를 관통하잖아요? (물론 분리할 수도 있겠지만 그건 나중에 생각하기로 합니다.)

**Spring AOP**
==============

* * *

**개요**
------

* * *

사실 횡단 관심사라고 했지만, 클래스 별로 가지는 관심사가 다를 수 있습니다. 여러 객체가 자신의 포지션을 가지고 협업하는 객체지향 프로그래밍에서 각 객체의 관심사는 각자가 천차만별일 수 있습니다.

객체 지향 프로그래밍에서 객체는 스타크래프트나 LOL의 유닛이라고 생각하면 편합니다. LOL에서의 미드 라이너의 관심사는 정글의 블루 몹, 그리고 미드라인이겠지만, 정글러의 관심사는 또 다르겠죠?

> 정글러 : 뭐요? 망한 라인은 내가 킬을 먹고 크겠습니다!
> 
> ….. ![(파란별)](images/icons/emoticons/72/1f603.png) ![(파란별)](images/icons/emoticons/72/1f44a.png)

Spring 에서는 이런 관심사를 `@Aspect` 어노테이션을 통해 모듈화 시킬 수 있습니다. 이번 세션에서는 Spring AOP 에 대해 한번 톺아보겠습니다.

![image](https://github.com/user-attachments/assets/0acc20d3-0fb7-4d50-a2d8-f5ced11f376d)

Spring 에서는 Proxy 패턴을 통해 이 AOP 를 구현할 수 있습니다. 특이한 건 Spring Bean 에만 AOP 를 적용시킬 수 있다는 점입니다. 어찌보면 당연한게, Spring IOC 가 관리하는 영역 밖의 클래스들은 Spring 의 책임 밖이니까요.

Spring Boot 에서는 `spring-boot-starter-aop` Dependency 를 추가하면 해당 기능을 사용할 수 있습니다.

```kotlin
@Aspect
@Component
class LoggingAspect {

    private val logger: Logger = LoggerFactory.getLogger(this::class.java)

    // Service 계층의 모든 메소드에 대해 Pointcut 정의
    @Pointcut("@target(org.springframework.stereotype.Service)")
    fun serviceMethods() {}

    // 메소드 호출 전 로깅
    @Before("serviceMethods()")
    fun logBefore() {
        logger.info("Service 메소드 호출 시작")
    }

    // 메소드 호출 후 정상 반환 시 로깅
    @AfterReturning("serviceMethods()")
    fun logAfterReturning() {
        logger.info("Service 메소드 호출 성공")
    }

    // 메소드 호출 중 예외 발생 시 로깅
    @AfterThrowing(pointcut = "serviceMethods()", throwing = "exception")
    fun logAfterThrowing(exception: Throwable) {
        logger.error("Service 메소드 호출 중 예외 발생: ${exception.message}", exception)
    }

    // 메소드 실행 전후 시간 측정 및 로깅
    @Around("serviceMethods()")
    fun logAround(joinPoint: ProceedingJoinPoint): Any? {
        val startTime = System.currentTimeMillis()
        return try {
            val result = joinPoint.proceed()
            val duration = System.currentTimeMillis() - startTime
            logger.info("Service 메소드 ${joinPoint.signature.name} 실행 시간: ${duration}ms")
            result
        } catch (e: Throwable) {
            logger.error("Service 메소드 ${joinPoint.signature.name} 실행 중 에러 발생", e)
            throw e
        }
    }
}
```

```shell
# 로직 성공
INFO  LoggingAspect - Service 메소드 호출 시작
서비스 로직 실행
INFO  LoggingAspect - Service 메소드 호출 성공
INFO  LoggingAspect - Service 메소드 doSomething 실행 시간: 2ms

# 에러 발생
INFO  LoggingAspect - Service 메소드 호출 시작
ERROR LoggingAspect - Service 메소드 호출 중 예외 발생: 예외 발생
ERROR LoggingAspect - Service 메소드 throwError 실행 중 에러 발생
java.lang.RuntimeException: 예외 발생
```

간단한 예시를 한번 보겠습니다. `@PointCut` 어노테이션을 통해 어떤 컴포넌트가 이 관심사를 가질 지 지정합니다. 그리고 `@Before`, `@AfterReturning`, `@AfterThrowing`, `@Around` 등을 통해 해당 시점의 로깅 로직을 작성하고 있습니다.

사용 예시를 보았으니 자세한 사용방법을 한번 알아볼까요?

**주요 개념**
---------

* * *

`@Aspect` 어노테이션을 붙히기만 해서 끝나는 건 아닙니다. `@Controller` 도 마찬가지로 엔드포인트를 선언 해 주어야 하는 것 처럼 이 Aspect 가 어떤 행동을 할 지 지정해주어야 합니다.

또한 `@Aspect` 는 Spring Bean 으로 간주됩니다. Spring IOC 컨테이너의 영향권 내에 있다는 이야기죠. 자세한 원리를 알아보기 전에 사용방법을 먼저 알아보겠습니다.

우선 주요 개념들을 한번 정리 해 보겠습니다.

![image](https://github.com/user-attachments/assets/ef2dbbac-ec6a-4eb7-bbc8-6d5524efc8f9)


*   Aspect
    
    *   흩어진 관심사를 모듈화 한 것을 의미합니다.
        
    *   JoinPoint, Advice를 결합 한 객체를 의미합니다.
        
    *   관심사의 로직, 그리고 해당 로직이 수행 될 시점을 결합 하여 모듈화 한 객체를 의미합니다.
        
*   Target
    
    *   Aspect 를 적용하는 곳을 의미합니다.
        
*   Advice
    
    *   어떤 일을 해야 할 지에 대한 기능들을 담은 `구현체(class)` 입니다.
        
*   JointPoint
    
    *   Advice 가 적용 될 위치를 의미합니다.
        
    *   메소드 진입, 생성자 호출, 필드에서 값을 꺼내올 때 등 다양한 시점에서 적용이 가능합니다.
        
    *   `before`, `after-throwing`, `after`, `around` 등의 시점을 직접 지정해줄 수 있습니다.
        
*   PointCut
    
    *   JointPoint 의 상세한 스펙을 정의한 것을 의미합니다.
        
    *   표현식들로 특정한 대상(클래스, 메소드 )을 지정할 수 있습니다.
        
*   Advisor
    
    *   Advice 와 Pointcut 을 합친 객체 입니다.
        
    *   추가 할 부가기능과 부가기능을 적용 할 대상을 모두 알고 있는 객체 입니다.
        

Spring AOP 의 특징
---------------

* * *

Spring은 proxy 패턴 기반의 AOP 구현체를 사용합니다. proxy를 사용하는 이유는 실질적인 Target과 Aspect에 접근하기 위한 중간다리가 필요하기 때문입니다.

AOP 코드의 예시를 하나 살펴보겠습니다. AOP 클래스 또한 Spring IoC의 영향을 받으므로 `@Component` 어노테이션을 붙혀서 Spring Bean으로 등록 해 주어야 합니다.

```kotlin
@Component
@Aspect
class TestAspect {
	@PointCut("execution(\* com.test.\*.TestService.\*(..))")
	fun cut() {}

	@Before("cut()")
	fun before(joinPoint : JoinPoint) {
		//Do Something
	}
}
```

`@PointCut`을 통해 JoinPoint를 지정하였고, `@Before`를 통해 해당 메소드들의 진입 이전 시점의 로직을 작성하였습니다.

![image](https://github.com/user-attachments/assets/b1d4b384-ac90-4a59-92ad-8d4261a78adc)

문제는 이것 만으로는 TestAspect 가 할 수 있는 게 아무것도 없다는 것입니다. 이 클래스에는 `@Aspect` 와 `joinPoint` 만 지정 되어있고 실제 Target과의 연결이 전혀 없습니다.

이 역할을 Proxy 클래스가 대신 해줍니다. Proxy 패턴은 요청 처리의 중간 단계에서 원본 객체를 대신해서 작업을 처리하도록 하는 패턴입니다. Proxy 객체는 원본 객체와 같은 인터페이스를 가지고 있기 때문에 사용하는 입장에서는 원본을 사용하는 것과 같은 효과를 가질 수 있습니다. 이러한 Proxy를 쓰는 이유를 생각 해 보면 원본 객체 그 자체에 대한 접근을 제어하기 위해서, 그리고 부가적인 기능을 제어하기 위해서라고 할 수 있습니다.

> _원본 객체의 메소드에 접근하기 전에 권한에 대한 검사를 한다거나 로그를 남기는 등의 역할을 Proxy가 대신 해 줄 수 있습니다. 원본 객체는 원래 원본이 가지고 있던 책임만 다 하면 되죠._

Spring AOP 에서도 마찬가지입니다. 원본 객체에 대한 코드 수정은 배제하고, Proxy를 통해 부가적인 처리를 추가하고자 하기 위해 Proxy 객체를 사용합니다.

앞서 말씀드렸듯이 AOP를 도입하는 이유는 비즈니스 로직과는 다른 부가적인 기능들에 대한 모듈화 입니다. 우리는 원본 객체 (Service 컴포넌트 등) 에 이러한 부가적인 기능들을 추가하기 위해 코드를 수정하는 것을 바라지 않습니다.

그래서 Spring 에서는 AOP 구현체를 생성할 때 Proxy 형태로 생성하게 됩니다. Target을 감싸고 있는 형태고, 우리가 지정한 부가적인 처리들이 Target Method 를 실행하기 전에 우리가 설정한 옵션 값에 따라 호출됩니다.

즉, Spring AOP 는 Proxy 를 통해 원본의 코드를 전혀 수정하지 않고 부가적인 기능들을 구현하도록 도와줍니다. AOP 의 영향권에 있는 컴포넌트 마다 똑같이 코드를 수정하는 게 아니라, Proxy를 통해 실제 기능이 호출되기 전 AOP 의 로직이 작동하는 것입니다.

![image](https://github.com/user-attachments/assets/9fff023e-5e88-41d0-a6c0-06830bdaae5f)

**스프링에서 Proxy 인스턴스를 생성하는 방법**
-----------------------------

* * *

Spring AOP 는 앞서 말씀드렸듯이 Proxy 패턴을 기초로 합니다. 많은 블로그 글에서 찾아볼 수 있겠지만 Spring은 두 가지 방법의 Proxy 인스턴스 생성 방식을 지원합니다.

![image](https://github.com/user-attachments/assets/eb6d424f-9e4f-4660-a33d-dd4a4fd43272)

*   JDK Dynamic Proxy
    
    *   Target Object 가 인터페이스 기반으로 생성 되었을 때 주로 사용합니다.
        
*   CGLib Proxy
    
    *   만약 Target Object 가 인터페이스 기반으로 개발 되어있지 않은 경우 해당 오브젝트의 바이트 코드를 조작하여 Proxy를 생성합니다.
        

실제로 Proxy 를 통해 AOP 가 작동하는 과정을 코드로 이해 해 보겠습니다.

```kotlin
class TestServiceImpl : TestService {
	private val log = getLogger()

	fun method1() {
		method2()
	}

	fun method2() {
		log.info("Self call 예시")
	}
}

fun main() {
	val testService = TestServiceImpl()
	testService.method1()
}
```

Proxy 를 따로 사용하지 않고 직접 인스턴스를 생성해서 메소드를 호출 한 상황입니다. 어렵 게 이해할 것도 없이, 직접 인스턴스가 가지고 있는 원본 메소드를 호출 합니다.

```kotlin
class TestServiceImpl : TestService {
	private val log = getLogger()

	fun method1() {
		method2()
	}

	fun method2() {
		log.info("Proxy 예시")
	}
}

fun main() {
	val factory = ProxyFactory(TestServiceImpl())
	factory.addInterface(TestService::class)
	factory.addAdvice(TestAdvice())
	val testService = factory.getProxy() as TestService
	testService.method1()
}
```

아까완 다르게 직접 인스턴스를 생성하는 게 아니라 ProxyFactory를 통해 생성 하였습니다. 이 과정에서 사용 할 Proxy 인스턴스의 사양을 지정해줄 수 있습니다.

```kotlin
val proxyFactory = ProxyFactory()
proxyFactory.addInterface(TestService::class)
proxyFactory.addAdvice(advice)
proxyFactory.addAdvisor(adviso
```

어떤 오브젝트를 Target으로 둘 것인지, 어떤 부가기능들을 추가 할 것인지를 proxyFactory에 등록한 후, getProxy 메소드를 호출하면 원본을 참조하는 Proxy 인스턴스가 생성됩니다.

물론 우리가 이 모든 코드들을 직접 다 작성 할 필요는 없고, 우리가 @Transactional 과 같은 어노테이션을 컴포넌트에 사용하거나 @Aspect 컴포넌트에 부가 로직들을 작성하면, Spring 에서 컴포넌트 인스턴스들을 생성할 때 해당 어노테이션을 참조해서 AOP Proxy를 생성하게 됩니다.

**Outro**
=========

* * *

지금까지 AOP, 그리고 Spring 에서의 AOP 에 대해 알아보았습니다.

우리가 사용하고 있던 Spring 의 기능이 어떻게 돌아가는 지 직접 뜯어본 후 다시 Spring 프로젝트 코드를 살펴보면 얼마나 현재의 Spring Boot 가 편리하게 구성되어있는 지를 알 수 있죠. 복잡한 설정들을 모두 배제하고 비즈니스 로직에만 집중하면 되니까요.

하지만 한편으로는 이게 어떻게 가능한 지 아는 것도 중요하다고 생각합니다. 우리가 항상 프레임워크에만 의존할 수는 없는 노릇이니까요.

다음 주제는 DI, IoC 컨테이너입니다. 다음달도 고생 좀 하겠네요. 곧 다시 봽겠습니다.

