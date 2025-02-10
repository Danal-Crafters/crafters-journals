# Intro

다시 돌아왔습니다. 알고쓰자 Spring 시리즈에서 상당히 중요한 파트를 차지 할 것 같아보이는 주제를 가져왔습니다.

바로 IoC, DI 입니다. 아마 많은 사람들이 Spring 을 사용하시면서 자주 들으셨을, 그 만큼 Spring 동작 원리에 큰 영향을 끼치는 주제죠.

아마 많은 분들이 이미 잘 알고 쓰고 계실 것이라 생각합니다만, 그 만큼 중요한 주제기에 포스팅을 통해 제대로 점검 해 보고 싶었습니다.

이번 알고쓰자 Spring 도 한번 힘차게 시작 해 보겠습니다.

# Spring 에서의 IoC, DI

Spring 공부를 하다보면 IoC 와 DI 라는 키워드를 정말 많이 들어보셨을 것입니다. AOP 만큼이나 두 키워드는 Spring 프레임워크의 핵심적인 동작 원리이기 때문입니다.

우선 IoC (Inversion of Control, 제어의 역전) 의 의미는 영어 뜻 그대로 제어가 역전되었다는 의미입니다. 소프트웨어의 제어권이 개발자가 아니라 프레임워크에 있다는 의미죠. DI (Dependency Injection, 의존성 주입) 은 객체가 필요로 하는 의존성 (함께 상호작용 하는 다른 객체 등) 을 직접 생성하는 게 아니라 외부에서 주입받겠다는 의미입니다.

말로만 들었을 땐 사실 의미가 잘 와닫지 않습니다. 한번 깊게 들어가볼까요?

## IoC

만약 Spring 을 지금 시점에서 처음 써보셨거나 써보신 지 얼마 안되신 분이라면, 혹은 프레임워크 자체에 익숙하지 않으신 분이라면 Spring 애플리케이션 코드를 작성하는 방식이 상당히 신기하게 느껴지셨지 않을까 하는 생각이 듭니다.

```kotlin
@Controller
class TestController {
    //Do Something
}

@Service
class TestService {
    //Do Something
}
```

비즈니스 로직을 처리하는 `Service` 컴포넌트, 요청과 응답을 처리하는 `Controller` 컴포넌트를 작성하기만 해도 알아서 이 컴포넌트들이 동작합니다. 우리가 막 Java 혹은 Kotlin 를 배웠던 시절의 기억을 떠올려봅시다.

```java
public static void main (String[] args) {
    //Do Something
}
```

아마 이 메인 메소드부터 코드를 작성했을 것이고, 이 메소드를 실행시켰겠죠. Spring 에서도 마찬가지로 존재하지만, 일반적으로 개발자가 크게 이 메소드에 개입하지는 않습니다.

```java
@SpringBootApplication
public class ExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```

어노테이션을 추가하거나 다른 메소드를 작성할 진 몰라도 직접적으로 main 메소드에 코드를 작성할 일은 흔하지 않습니다.
이 말은 즉 Spring 애플리케이션은 개발자의 통제 범위에서 어느정도 벗어나있다는 의미입니다. 실제로 앞서 선언했던 컴포넌트들의 라이프사이클은 개발자가 전혀 개입하지 않습니다. 실제로 아래와 같은 코드는 개발자가 직접 작성할 일은 없습니다.

```java
TestController testController = new TestController()
```

제어권이 프레임워크에 넘어 간 객체들은 DI 를 적용시킬 수 있습니다.

## DI

Spring 의 제어권에 들어 간 객체들은 DI 를 통해 주입받을 수 있습니다. 앞서 보여드렸듯이 IoC 의 영역에 있는 객체라면, 직접적으로 객체를 생성할 일은 없습니다.

```kotlin
interface SendMessage {
    fun sendMessage(param : String)
}

@Service
fun TestService : SendMessage {
    override fun sendMessage(param : String) {
        //Do Something
    }
}

@RestController
fun TestController (
    private val testService : SendMessage
) {
    @PostMapping("/api/message")
    fun sendMessage(@RequestBody request : SendMessageRequest) : ResponseEntity<String> {
        testService.sendMessage(request.message)
	return ResponseEntity.ok().body("SUCCESS")
    }
}
```

간단한 RestController 예제입니다. `/api/message` REST API (POST) 를 구현하는 예시입니다. 이 코드들에서는 직접적으로 객체를 생성하는 코드가 단 하나도 없습니다.

IoC, 즉 객체의 제어권이 개발자에게 없으므로 `TestService` 는 `TestController` 에서 생성되는 게 아니라 생성자 파라미터로서 주입받아서 사용됩니다.

# 객체지향 관점에서의 IoC 와 DI

지금까지 Spring 에서의 IoC, DI에 대해 이야기 해 보았습니다. Spring 의 이런 특징들은 Spring 에 입문하는데 장벽이 되곤 하지만, Spring 을 사용하는 이유가 되곤 합니다. 바로 객체지향 관점에서 IoC, DI가 좀 더 기존의 Java 애플리케이션에 비해 유리하기 때문입니다.

알고쓰자 시리즈에서 몇번 언급을 했던 내용이지만, 사실 순수 Java 는 객체지향 원칙을 모두 지키기 어려운 언어입니다. 제가 알고쓰자 시리즈를 처음 작성했을 때 사실 IoC, DI 이야기를 잠시 언급한 적이 있습니다.

```java
Shape shape = new Circle(); // 인터페이스가 구체적인 클래스에 의존
```

만약에 우리가 다형성을 지키기 위해 인터페이스 타입을 가진 객체를 생성한다고 가정 해 보겠습니다. 문제는 이 인터페이스를 데이터타입으로 갖는 객체는 클래스가 변화할 때 영향을 받을 수 밖에 없습니다. 이는 **DIP** *(Dependency Inversion Principle, 의존관계 역전 원칙. 프로그래머는 추상화에 의존해야지, 구체화에 의존하면 안된다.)* 를 위배할 수 밖에 없습니다. 또한 Circle 클래스가 문제가 생겨서 새로운 CircleV2 로 변경한다고 하면 위의 코드가 수정되므로 **OCP** *(Open / Closed Principle, 개방 폐쇄 원칙, 소프트웨어 요소는 확장에는 열려있으나 변경에는 닫혀 있어야 한다.)* 를 위반합니다.

> **여기서 다형성이란?**
> 다형성(polymorphism)이란? 다형성(polymorphism)이란 **하나의 객체가 여러 가지 타입을 가질 수 있는 것을 의미**합니다. 자바에서는 이러한 다형성을 부모 클래스 타입의 참조 변수로 자식 클래스 타입의 인스턴스를 참조할 수 있도록 하여 구현하고 있습니다.

하지만 IoC, DI 를 사용하게 된다면 객체의 생성 자체가 코드 단의 책임이 아니므로 이런 문제에서 자유로워 집니다. IoC는 객체 생성과 라이플사이클 관리의 책임을 가져감으로써 개발자가 객체지향 5대 설계원칙을 위배하지 않도록 도와주죠. 특히 OCP 를 실현하는 데 매우 중요한 역할을 합니다. **다시 말씀 드리지만 객체의 생성 책임이 코드 단의 책임이 아니니까요.** `new Circle()`  과 같은 코드가 없으니까요.

또한 앞서 예시에서 DI 는 우리가 필요로 하는 의존성 (객체) 들을 외부에서 주입받게 함으로써 인터페이스 기반 프로그래밍을 촉진시킬 수 있습니다. 사실 꼭 인터페이스로 컴포넌트들을 구현 할 필요는 없긴 하지만 다형성을 챙기고 싶다면 인터페이스 분리를 하는 것을 권장 드립니다.

인터페이스 분리는 결합도를 낮추는 데 큰 도움을 줍니다. 소프트웨어 엔지니어링 전공 시간에 결합도와 응집도라는 말이 정말 많이 나오죠? 결합도는 객체 간의 의존 정도를 의미합니다. 객체를 넘어 모듈(패키지) 가 될 수 있죠. 결합도가 높으면 높을 수록 소프트웨어를 유지보수하기 힘들어집니다. 서로 엮인 게 많다는 건, 하나를 고쳤을 때 영향을 크게 받는다는 의미니까요. 응집도는 이런 결합이 적은 객체, 또는 모듈들이 함꼐 속하는 정도, 즉 서로 연관성이 얼마나 있느냐를 구분하는 척도입니다. 응집도가 높을 수록 모듈 간에 책임이 명확히 구분 되므로 유지보수를 하기 쉬워집니다. 각자의 역활이 명확한데 결합이 적다면, 문제가 생겼을 때 그 문제를 격리시킬 수 있으니까요.

# IoC 동작과정

Spring 의 Ioc 컨테이너는 ApplicationContext 또는 BeanFactory 인터페이스를 통해 구현됩니다.

* **`BeanFactory`** : 기본적인 IoC 컨테이너로, 지연 초기화(Lazy Initialization)를 지원합니다.
* **`ApplicationContext`** : `BeanFactory`의 확장판으로, 이벤트 리스너, 메시지 소스 등을 추가로 제공합니다.

Spring IoC 컨테이너는 다음과 같은 단계를 통해 동작합니다.

* 구성 파일 또는 어노테이션을 분석하여 Bean 정의 로드
* **Bean** 객체를 생성하고, 필요하면 의존성을 주입
* 생성된 Bean을 `ApplicationContext` 또는 `BeanFactory`에서 관리
* 필요에 따라 라이프사이클 콜백(`@PostConstruct`, `@PreDestroy`)을 실행

>  라이프사이클 콜백은 이름에서 유추할 수 있다시피 생성자 실행 후, 객체 소멸 전 등 각각의 시점에서 콜백을 지정할 수 있는 기능입니다. 

```kotlin
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration
import org.springframework.context.annotation.AnnotationConfigApplicationContext
import org.springframework.stereotype.Component

// 의존성 인터페이스 정의
interface MessageService {
    fun sendMessage(message: String)
}

// EmailService 구현체
@Component
class EmailService : MessageService {
    override fun sendMessage(message: String) {
        println("Email sent: $message")
    }
}

// UserService에서 MessageService 사용
@Component
class UserService(private val messageService: MessageService) {
    fun notifyUser() {
        messageService.sendMessage("Hello, Spring IoC!")
    }
}

// Spring IoC 컨테이너 설정
@Configuration
@ComponentScan(basePackages = ["com.example"])
class AppConfig

fun main() {
    // IoC 컨테이너(ApplicationContext) 초기화
    val context = AnnotationConfigApplicationContext(AppConfig::class.java)

    // Bean 가져오기
    val userService = context.getBean(UserService::class.java)
    userService.notifyUser()

    // 컨테이너 종료
    context.close()
}

```

코드를 보며 동작 과정을 살펴보겠습니다.

* **인터페이스 정의**
  * `MessageService` 인터페이스를 만들고 `EmailService`가 이를 구현하도록 구성합니다.
* **의존성 주입(DI) 처리**
  * `UserService` 클래스에서 `MessageService`를 생성자 주입(`constructor injection`)으로 주입받습니다.
  * Spring이 자동으로 `EmailService`를 찾아 `UserService`의 생성자에 주입합니다.
* **Spring IoC 컨테이너 생성**
  * `AnnotationConfigApplicationContext(AppConfig::class.java)`를 통해  **IoC 컨테이너 생성** 합니다.
  * `@ComponentScan`이 `com.example` 패키지의 모든 `@Component`를 자동 스캔하여 Bean 등록 합니다.
* **Bean 조회 및 실행**
  * `context.getBean(UserService::class.java)`를 통해 `UserService` Bean을 가져와 `notifyUser()` 실행 합니다.

`@ComponentScan` 어노테이션을 Spring 애플리케이션의 가장 상위 클래스에 선언하면, baskPackage 하위의 `Bean` 컴포넌트들을 스캔하여 `ApplicationContext `에 지정하게 됩니다. 35번줄을 자세히 살펴보면,`AppConfig` 이 가장 상위 클래스로 간주되고 있죠. 따로 UserService 를 등록하는 과정이 코드 레벨 단에서는 존재하지 않습니다. ComponentScan 어노테이션을 선언하는 것 외에는 말이죠.

# IoC 컨테이너 심화 : BeanFactory VS ApplicationContext

` BeanFactory` 는 Lazy Initialization (지연 초기화) 를 사용합니다. 메모리를 적게 먹지만, `ApplicationContext` 보다 기능이 적죠. Spring Boot 에서는 거의 사용되지 않습니다.

```kotlin
import org.springframework.beans.factory.BeanFactory
import org.springframework.beans.factory.xml.XmlBeanFactory
import org.springframework.core.io.ClassPathResource

fun main() {
    val beanFactory: BeanFactory = XmlBeanFactory(ClassPathResource("beans.xml"))

    val userService = beanFactory.getBean(UserService::class.java)
    userService.notifyUser()
}

```

> 여기서 Lazy Initialization (지연 초기화) 란?
>
> 지연 초기화( **Lazy Initialization** )는 **객체(Bean)의 인스턴스를 필요할 때까지 생성하지 않고, 실제로 사용할 때 초기화하는 기법**입니다. Spring에서는 `@Lazy` 어노테이션을 사용하여 Bean을 지연 초기화할 수 있습니다.
>
> 지연 초기화의 특징은 아래와 같습니다.
>
> * **초기 로딩 속도 향상**
>   * 애플리케이션 시작 시 불필요한 객체 생성을 방지하여 **기동 속도(startup time)** 를 줄일 수 있습니다.
> * **메모리 절약**
>   * 불필요한 객체를 즉시 생성하지 않으므로  **메모리 사용량이 감소합니다** .
> * **의존성이 필요한 시점에 주입됨**
>   * `@Lazy`로 설정된 Bean은  **해당 Bean을 처음 사용할 때 객체가 생성됩니다** .
> * **단점: 첫 번째 호출이 느려짐**
>   * Bean이 사용될 때 최초 한 번 생성되므로 **첫 사용 시 약간의 성능 오버헤드**가 발생할 수 있습니다.

`ApplicationContext` 는 `BeanFactory` 를 확장한 고급 IoC 컨테이너입니다. 아래와 같은 추가 기능을 제공하죠.

* **Eager Initialization (즉시 초기화)** : 애플리케이션이 시작될 때 Bean을 즉시 생성 합니다.
* **이벤트 리스너 지원** : Application 이벤트 (`ContextStartedEvent`, `ContextRefreshedEvent` 등) 처리 가능합니다.
* **메시지 소스 지원** : 다국어 메시지를 처리할 수 있습니다.
* **환경 설정 지원** : `@PropertySource`, `Environment` 객체를 활용한 설정 가능 합니다.

일반적으로 우리가 자주 사용하는 Spring Boot 는 `ApplicationContext` 을 사용합니다.

```kotlin
import org.springframework.context.ApplicationContext
import org.springframework.context.annotation.AnnotationConfigApplicationContext

fun main() {
    val context: ApplicationContext = AnnotationConfigApplicationContext(AppConfig::class.java)

    val userService = context.getBean(UserService::class.java)
    userService.notifyUser()
}

```

> Eager Initialization (즉시 초기화) 의 특징을 좀더 자세히 살펴볼까요?
>
> * **애플리케이션 시작 시 모든 Singleton Bean을 미리 생성합니다**
>   * Spring Boot 애플리케이션이 실행되면서 `ApplicationContext`가 로드될 때, 모든 Bean을 한 번에 생성하고 초기화합니다.
>   * `@Component`, `@Service`, `@Repository`, `@Bean`으로 등록된 Bean이 즉시 초기화됩니다.
> * **첫 번째 요청 속도가 향상 됩니다**
>   * 애플리케이션 실행 후, 첫 요청 시 이미 초기화된 객체를 사용하므로 성능이 빠릅니다.
> * **초기 로딩 시간이 길어질 수 있습니다**
>   * 애플리케이션 실행 시 모든 Bean을 초기화하므로,  **Bean이 많을수록 실행 속도가 느려질 수 있습니다** .
> * **모든 Bean이 즉시 주입되므로, 미사용 Bean도 로딩됩니다**
>   * **실제로 사용되지 않는 Bean도 미리 생성됩니다** → 메모리 낭비 가능성.

그러면 현재의 Spring Boot 애플리케이션은 Eager Initialization 만 사용할까요? 상황에 따라서는 Lazy Initialization 을 사용할 수 있습니다.

```kotlin
import org.springframework.context.annotation.Lazy
import org.springframework.stereotype.Component

@Component
@Lazy
class LazyBean {
    init {
        println("LazyBean 생성됨!")
    }
}

```

이렇게 개발 Bean 을 `@Lazy` 어노테이션을 통해 변경할 수 있죠.

```yaml
spring:
  main:
    lazy-initialization: true
```

Spring Boot 2.2 이상에서는 lazy-initialization 옵션을 통해 모든 Bean 들의 초기화 방식을 수정할 수 도 있죠.

Spring Boot 에선 기본 값으로 Eagar Initialization 을 사용한다고 말씀 드렸는데, 만약 애플리케이션 기동 시간이 중요하거나 무거운 Bean 들이 많을 때는 부분적으로 Lazy Initialization 을 도입하는 것을 고려해볼 수 있습니다.

하지만 일반적으로는 Eager Initializaion 이 유리합니다. 크게 세 가지 부분에서 이점을 가지죠.

* **자주 사용되는 Bean이 많을 때**
  * 서비스 로직의 핵심이 되는 Bean이라면, Eager Initialization이 성능에 유리합니다.
* **초기 실행 속도보다, 응답 속도가 중요한 경우**
  * 예: 웹 서버(Spring Boot 애플리케이션)에서는 미리 Bean을 초기화하면 첫 요청이 빠릅니다.
* **캐시(Cache) 등 미리 로딩해야 하는 데이터가 있을 때**
  * 예: 데이터베이스에서 값을 미리 가져와야 하는 경우.

# IoC 컨테이너 심화 : 설정 방식

Spring IoC 컨테이너는 과거엔 Xml 기반으로 Bean 을 설정했습니다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="messageService" class="com.example.EmailService"/>
    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="messageService"/>
    </bean>

</beans>

```

현재 Spring Boot 에서 이런 Xml 기반 설정은 거의 찾아볼 수 없죠. 물론 Xml 은 훌륭합니다. 하지만, Java / Kotlin 기반 설정에 비해 가독성이 크게 나쁘다는 단점이 있습니다. 뭣보다 코드 단에서 컴파일러의 관할에 있는 데이터가 아니므로 휴먼에러가 발생했을 때 트래킹 하기 매우 어렵습니다.

> `com.example.EmailService` 에서 오타가 났다고 생각해볼게요. 과연 코드단에서 찾는게 빠를까요 Config 파일 중 하나를 뒤지는게 빠를까요?

```kotlin
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration

@Configuration
class AppConfig {
    @Bean
    fun messageService(): MessageService {
        return EmailService()
    }

    @Bean
    fun userService(messageService: MessageService): UserService {
        return UserService(messageService)
    }
}

```

Spring Boot 부터는 일반적으로 `@Configuration` 클래스를 선언하는 방식으로 설정을 구성합니다. **타입 안정성**이 높고, 유지보수가 쉬워 Spring Boot에서 가장 많이 사용됩니다.

# 프레임워크 없이 DI, IoC 구현 해 보기

'바퀴를 만들지 말라!' 라는 말을 들어보셨을 것입니다. 이미 잘 동작하고 있는 도구들을 굳이 다시 재개발 하지 말라는 의미입니다. 하지만 때로는 공부 목적으로 우리는 바퀴를 발명 해 보는 게 큰 도움이 됩니다.
아래의 예시는 간단한 IoC 컨테이너를 Kotlin 만 가지고 구현 해본 예시입니다.

```kotlin
class SimpleIoCContainer {
    private val beans = mutableMapOf<Class<*>, Any>()

    fun <T : Any> registerBean(clazz: Class<T>, instance: T) {
        beans[clazz] = instance
    }

    fun <T : Any> getBean(clazz: Class<T>): T {
        return clazz.cast(beans[clazz]) ?: throw RuntimeException("Bean not found: $clazz")
    }
}

// 사용 예제
fun main() {
    val container = SimpleIoCContainer()

    // 수동으로 객체 등록
    container.registerBean(MessageService::class.java, EmailService())
    container.registerBean(UserService::class.java, UserService(container.getBean(MessageService::class.java)))

    // Bean 조회 및 실행
    val userService = container.getBean(UserService::class.java)
    userService.notifyUser()
}

```

동작 방식은 아래와 같습니다.

* `registerBean()`을 이용해 객체를 IoC 컨테이너에 등록.
* `getBean()`을 이용해 등록된 객체를 가져와 사용.

수동으로 IoC 컨테이너에 객체를 등록하는 과정을 진행 할 것이냐, 아니면 프레임워크에 맡길 것이냐의 차이죠. 결국 아래의 세 가지 이점으로 인해 직접 구현하는 것 보다 Spring IoC 컨테이너를 사용하는 것이 이점을 가집니다.

* **결합도 낮추기** : 객체가 직접 의존성을 생성하지 않고 외부에서 주입받음.
* **유지보수성 증가** : Bean 간 결합이 느슨해지면서 확장 및 변경이 용이해짐.
* **라이프사이클 자동 관리** : Bean 초기화 및 소멸 관리 기능을 제공.

# Outro

지금까지 Spring IoC, DI 의 개념, 동작과정, 설정 방법 등에 대해 알아보았습니다. 총 다섯개의 시리즈를 다루며 Spring 의 기본적인 동작원리, 설계원칙들을 다루었습니다. 좀 어떠셨나요? 저도 다섯번이나 글을 쓰게 될 줄은 몰랐습니다 키키...

다음에는 Spring Security 에 대해 한번 다뤄볼까 합니다. 사실 다섯개의 포스팅으로 Spring 을 제대로 이해하고 있다고 말씀 드릴 순 없다고 생각합니다만, 가장 기초적인 원리들은 다뤘으니 이제 파생되는 기술들을 하나씩 파고들어볼까 하는 생각이 드네요.

재밌게 읽으셨다면 정말 다행이고, 혹시 피드백 하실 부분이 있으시다면 자유롭게 의견 남겨주시면 감사하겠습니다.
그럼 추후 또 뵙겠습니다. 😊

# Reference

- https://docs.spring.io/spring-framework/reference/languages/kotlin/bean-definition-dsl.html
- https://docs.spring.io/spring-framework/reference/core/beans.html
