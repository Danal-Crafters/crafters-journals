# 🍃 Intro

다시 돌아왔습니다. 이번 호는 예전에 IoC, DI 에 대해 다룰 때 잠시 다뤘던 주제를 좀 더 깊게 다뤄볼까 합니다. 바로 Bean, ApplicationContext, Environment 에 대해서 입니다.

사실 IoC, DI의 원리는 워낙 중요하고 Spring 동작 원리의 핵심인지라, 처음 Spring 을 공부하시는 분들이라고 해도 많이 들어보셨을 것이라 생각합니다.

큰 틀에서의 핵심 원리는 그렇다고 하는데 IoC 컨테이너 안에서 관리되는 Bean 컴포넌트들을 좀 더 자세히 뜯어봐야 겠다는 생각이 들어서 이번 주제를 선택했습니다. 조금 더 코어 이야기를 **깊게** 해보고 싶었어요.

이번 포스팅도 편하게 읽어 주세요!

# 🫘 Bean 개요

## 🫘 Bean 개요

Spring 에서 Bean 은 Spring IoC Container 가 관리하는 인스턴스를 의미합니다. 일반적인 Java 객체지만 Spring 컨테이너가 생성, 초기화, 소멸을 모두 관리하죠.

```kotlin
@Component
class MyService {
    fun execute() {
        println("Service logic")
    }
}
```

@Component, @Service, @Repository, @Controller 등 우리가 흔히 비즈니스 로직을 구현할 때 각 레이어를 구성하는 컴포넌트를 선언할 때 이런 어노테이션을 많이 사용하죠.

이렇게 컴포넌트로 선언 할 수도 있지만 메소드로 선언 할 수도 있죠.

```kotlin
@Bean
fun myService : MyService {
    return MyService()
}
```

## 🫘 Bean 이해하기 : 싱글톤

Bean 은 싱글톤입니다. 우리의 Spring 애플리케이션에서 특정한 Bean 을 선언하면 해당 Bean 은 싱글톤으로 동작합니다. 물론 우리가 일반적으로 생각하는 아래와 같은 싱글톤과는 다르게 IoC Container 에게 관리 권한이 들어간다는 차이가 있죠.

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {
        throw new IllegalStateException("SingleTon Class");
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

> 💡여기서 잠깐! 싱글톤이 무엇인가요?
>
> 전체 소프트웨어에서 오직 하나만 존재할 수 있는 객체를 싱글톤 객체라고 합니다.  
> 위의 예시를 보시면 알겠지만, private static final 로 필드값에 인스턴스 하나를 초기화 시켰죠.  
> 해당 클래스 내의 필드 인스턴스에 접근하기 위해서는 getInstance() 메소드를 사용해야 하죠. 직접적으로 외부에서 인스턴스 생성 등에 관여할 수 없습니다.
>
> 특정 클래스의 인스턴스를 하나만 만들어 어디서든 공유하고 싶을 때 사용합니다

오랜만에 Java 가 나왔네요. 막간을 이용해서 Kotlin 에서 순수 싱글톤을 선언하는 방법을 알아볼까요? Kotlin 은 Java 와 다르게 싱글톤 역할을 하는 타입이 존재합니다. 바로 `object` 죠.

```kotlin
object Singleton {
    val name = "My Singleton"

    fun sayHello() {
        println("Hello from $name")
    }
}
```

Kotlin 과 Java 의 차이는 여러가지가 있지만, static 영역을 어떻게 구현하는지에 따라 차이점이 있습니다.  
Java 와는 다르게 Kotlin 은 static 키워드가 존재하지 않고 object, companion object 를 사용합니다. JVM 메모리 상에서는 비슷하게 동작하죠.

주제에서 조금 벗어나는 이야기지만 Kotlin 은 top-level function 이라는 기능을 제공합니다. 우리가 Utility class 를 더이상 만들 필요가 없다는 의미죠. static 메소드만 잔뜩 가지고있는 클래스를 선언 할 필요없이 전역 함수를 선언해서 사용할 수 있습니다. 마찬가지로 이 전역 함수들은 static 영역으로 관리됩니다.

```java
public class StringUtils {

    // ❌ 더이상 private 생성자 만들 필요 없습니다. 
    private StringUtils() {
        throw new IllegalStateException("Utility Class");
    }

    public static void printString(String param) {
        System.out.println(param);
    }
}
```

```kotlin
// ✅ 그냥 파일 하나 생성해서 함수를 선언하면 됩니다. 
fun printString(param : String) {
    println(param)
}
```

만약 Java 와의 상호 운용이 필요하다면 `@JvmStatic` 키워드를 사용하면 됩니다.

```kotlin
object Singleton {
    val name = "My Singleton"

    @JvmStatic
    fun sayHello() {
        println("Hello from $name")
    }
}
```

top-level function 에서는 굳이 그럴 필요 없습니다. 파일 명이 만약 Utils.kt 라면 UtilsKt 라는 클래스에서 static 메소드를 불러오는 것 처럼 동작하죠.

```kotlin
package example.util

fun add(a : Int, b : Int) : Int = a + b
```

```java
int result = UtilsKt.add(1, 2);
```

> 💡만약 Java 에서 Kotlin 마이그레이션을 계획 중이시라면, 여기서 팁을 하나 얻어갈 수 있죠. 공통 유틸리티, 상수 같은 클래스들 먼저 Kotlin 으로 옮기는 것을 추천드립니다. Kotlin 클래스를 Java 에서 쓰는 것은 그렇게까지 까다롭지 않지만, 역은 @JvmOverloads, @JvmStatic, @JvmField 등 여러 어노테이션들을 써야 할 수 있죠.
>
> 개인적인 경험입니다만, 코드의 변경이 가장 적은 순서대로 진행하려면 공용 유틸, DTO 들 먼저 Kotlin 으로 옮긴 후 비즈니스 로직들은 천천히 옮기는 게 좋다고 봅니다. 실제로 무작정 옮기다가 많이 힘들었던 기억이 나네요 하하...😕

## 🌊 Bean 의 생명주기 전체 흐름

본론으로 다시 돌아올까요? 사실 Spring Bean 을 이해하기 위해선 싱글톤에 대한 이해가 필수라 조금 주저리 했습니다. Bean 과 일반적인 싱글톤 객체는 같은 싱글톤이지만, 누가 관리하느냐의 차이라고 말씀 드렸죠. Bean 과 다르게 일반 싱글톤은 생성과 소멸을 개발자가 직접 처리 해야 합니다. 하지만 Bean 은 자체적인 생명주기가 존재합니다. 정리해볼까요?

1. BeanDefinition 등록
2. Bean 인스턴스 생성 (Constructor 호출)
3. 의존성 주입 (@Autowired, setter)
4. 초기화 콜백 호출 (@PostConstruct, InitializingBean, initMethod)
5. Bean 사용
6. 컨테이너 종료 시 소멸 콜백 호출 (@PreDestroy, DisposableBean, destroyMethod)

Bean 생명주기는 총 여섯 단계로 이루어져있습니다. IoC Container 가 이 생명주기 관리를 담당하지만, 개발자는 이 생명주기 각각에 콜백함수를 넣어줄 수 있습니다.

```kotlin
import jakarta.annotation.PostConstruct
import jakarta.annotation.PreDestroy

class LifeCycleBean {

    init {
        println("1. 생성자 호출: 빈 인스턴스 생성")
    }

    fun setDependency(dependency: String) {
        println("2. 의존성 주입: setDependency 호출 - $dependency")
    }

    @PostConstruct
    fun init() {
        println("3. 초기화 작업: @PostConstruct 호출")
    }

    @PreDestroy
    fun destroy() {
        println("6. 소멸 직전: @PreDestroy 호출")
    }

    fun customInit() {
        println("3-2. 초기화 작업 (initMethod): customInit 호출")
    }

    fun customDestroy() {
        println("6-2. 소멸 작업 (destroyMethod): customDestroy 호출")
    }
}
```

```kotlin
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration

@Configuration
class AppConfig {

    @Bean
    fun lifeCycleBean(): LifeCycleBean {
        val bean = LifeCycleBean()
        bean.setDependency("Some Dependency")
        return bean
    }
}
```

Bean 의 생명주기 각각의 콜백함수와 등록까지의 예제를 살펴 보았습니다. Bean 을 실제로 사용하는 과정에서 생명주기마다 각각의 콜백이 호출 됩니다. 우리는 이 콜백을 활용해서 Bean 이 각자의 생명주기에 해야 할 행동들을 정의할 수 있죠.

예를 들면 애플리케이션이 켜지자마자 제일 먼저 특정한 컴포넌트가 특정 테이블의 데이터를 가져와서 `ConcurrentHashMap` 과 같은 컬렉션에 데이터를 저장해야 하는 경우에는 `@PostConstruct`, 즉 생성자 처리 이후에 호출되는 콜백에 프로그래밍 할 수 있습니다. 필드값들을 테이블 내 데이터 상태에 따라 다르게 셋업해야 하는 경우 유용하죠.

`@PostConstruct` 외에도 InitializingBean, initMethod 등을 지정해줄 수 있습니다. Bean 컴포넌트 초기화는 총 세 가지 단계로 구성되고 순서대로 호출됩니다. 표로 한번 정리 해 보았습니다.

| 방식                 | 설명                       | 예시                                  |
| ------------------ | ------------------------ | ----------------------------------- |
| `@PostConstruct`   | JSR-250 어노테이션, 가장 널리 사용됨 | `@PostConstruct public void init()` |
| `InitializingBean` | Spring 인터페이스 구현 방식       | `afterPropertiesSet()` 메소드 구현       |
| `initMethod`       | XML 또는 자바 설정에서 명시        | `@Bean(initMethod = "initMethod")`  |

소멸 단계도 마찬가지입니다. 총 세 가지 단계로 구성되어있습니다. `@PreDestroy` 콜백을 통해 소멸되기 전의 콜백을 지정해줄 수 있고 각각 순서대로 호출됩니다.

| 방식               | 설명                 | 예시                                       |
| ---------------- | ------------------ | ---------------------------------------- |
| `@PreDestroy`    | JSR-250 어노테이션      | `@PreDestroy public void destroy()`      |
| `DisposableBean` | Spring 인터페이스 구현 방식 | `destroy()` 메서드 구현                       |
| `destroyMethod`  | 자바 설정에서 명시         | `@Bean(destroyMethod = "customDestroy")` |

사실 Java 에는 소멸자라는 게 따로 없이 GC (Garbage Collector) 가 Heap 메모리에 존재하는 인스턴스들을 주기적으로 정리합니다. 일반적인 싱글톤 객체가 GC 의 영향권 밖에 있을 수 있는 이유가 static 영역 (흔히 말씀하시는 Method Area) 에 저장되기 때문입니다. 그럼 Bean 이 소멸되는 시점은 IoC Container 가 종료되는 시점입니다.

```kotlin
import org.springframework.context.annotation.AnnotationConfigApplicationContext

fun main() {
    val context = AnnotationConfigApplicationContext(AppConfig::class.java)

    println("5. Bean 사용")
    context.getBean(LifeCycleBean::class.java)

    context.close() // 6. 컨테이너 종료 -> @PreDestroy + destroyMethod 호출
}
```

Spring 없이 `ApplicationContext` 만 불러와서 context 를 닫는 예시입니다. Bean 이 소멸 되기 직전에 `@PreDestroy` 를 호출하고 그 외에 `destroyMethod` 와 같은 소멸 작업을 진행하죠.

```kotlin
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration

@Configuration
class AppConfig {

    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    fun lifeCycleBean(): LifeCycleBean {
        val bean = LifeCycleBean()
        bean.setDependency("Some Dependency")
        return bean
    }
}
```

@Bean 컴포넌트에는 그래서 `initMethod`, `destroyMethod` 와 같은 생명주기 내에 실행해야 할 콜백을 지정할 수 있습니다.

# 🧊 ApplicationContext

## 🧊 ApplicationContext 개요

지금까지 Bean 에 대해 알아보았습니다. 이번 세션에서는 Spring 의 핵심 요소인 `ApplicationContext` 에 대해 알아 보겠습니다.

`ApplicationContext` 는 Spring에서 가장 중심이 되는 IoC 컨테이너입니다. Bean들을 생성하고 관리하며, 필요한 경우 이를 주입해줍니다. Spring Framework 의 핵심 개념인 IoC 의 중심에 위치하며 Spring 애플리케이션의 전반적인 구동 기반을 제공합니다.

Spring Boot 에서는 개발자가 따로 명시하지 않아도 자동 생성됩니다. Spring 애플리케이션의 전체 라이프사이클과 DI 를 책임지는 중앙 허브 역할을 하죠.

## 🧊 ApplicationContext 의 구현체

ApplicationContext 는 사실 다양한 구현체가 존재합니다. 정리하면 아래와 같습니다.

| 구현체                                  | 설명                                    |
| ------------------------------------ | ------------------------------------- |
| `AnnotationConfigApplicationContext` | 자바 기반 설정 클래스(@Configuration) 사용 시 |
| `ClassPathXmlApplicationContext`     | XML 기반 설정 사용 시                        |
| `GenericWebApplicationContext`       | Spring Web에서 DispatcherServlet과 함께 사용 |
| `WebApplicationContext`              | Spring MVC 환경의 전용 ApplicationContext  |
| `ConfigurableApplicationContext`     | 종료(close), 리프레시(refresh) 등 수명주기 제어 가능 |

```kotlin
val context = AnnotationConfigApplicationContext(AppConfig::class.java)
val myService = context.getBean(MyService::class.java)
```

`@Configuration` 기반 클래스를 사용 시 위와 같이 생성할 수 있죠.

하지만 Spring Boot 에서는 개발자가 명시적으로 ApplicationContext 를 생성 할 필요는 없습니다.

```kotlin
@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    val context = SpringApplication.run(MyApplication::class.java, *args)
    val bean = context.getBean(MyService::class.java)
}
```

`SpringApplication.run()` 메소드는 내부적으로 ApplicationContext 를 구성하고 실행합니다. 테스트에서도 `@SpringBootTest` 를 사용하면 자동으로 주입되죠.

## 🧊 ApplicationContext 의 기능

ApplicationContext 는 크게 다음의 기능을 제공합니다.

| 기능       | 설명                                        |
| -------- | ----------------------------------------- |
| Bean 관리  | 등록된 Bean 객체를 생성, 주입, 초기화, 소멸까지 관리  |
| 국제화 지원   | 메시지 소스를 통한 다국어 처리 지원 |
| 이벤트 처리   | ApplicationEventPublisher로 이벤트 발행 및 수신 가능 |
| 환경 정보 제공 | `Environment`를 통한 설정값 관리 |
| 리소스 로딩   | 파일, URL, 클래스패스 등 다양한 리소스 접근 지원  |

ApplicationContext 는 흔히 BeanFactory 의 상위개념이라고 합니다. 앞서 설명 드렸듯이 Bean 의 관리가 사실 ApplicationContext 의 핵심 기능이긴 합니다만, 그 외에도 다른 기능들도 제공하니까요. 정리 되어있다시피 이벤트 발행 및 수신, 환경 정보 접근 등의 기능 또한 ApplicationContext 가 제공 하는 기능입니다.

평소에 모든 기능들을 전부 쓰지는 않지만, 이 시리즈의 취지에 걸맞게 한번 깊게 파고 들어보겠습니다.

### 🎉 ApplicationContext 의 기능 : 이벤트 처리

#### 🎉 이벤트 기능 개요

```kotlin
class MyEvent(val message: String) : ApplicationEvent(message)

@Component
class MyEventListener : ApplicationListener<MyEvent> {
    override fun onApplicationEvent(event: MyEvent) {
        println("이벤트 수신: ${event.message}")
    }
}
```

혹시 Spring 애플리케이션 개발하시면서 이벤트 처리를 해보신 적 있나요? 저는 트랜잭션하고 크게 상관없는 외부 모듈 (예를 들면 SMTP나 Push서버와의 통신) 과 연동할 때 자주 씁니다. 트랜잭션과는 아무 상관없고 트랜잭션이 필요없어서 분리 할 수도 없는 이벤트를 처리할 때 자주 쓰죠.

이처럼 ApplicationContext 는 이벤트 기반 비동기 메시징 기능을 지원합니다. 서로 느슨하게 연결 된 컴포넌트 간에 통신이 가능하고, 복잡한 흐름을 간결하게 나눌 수 있습니다.

#### 🎉 구조

이벤트 처리 기능을 이해하기 전에 알아둬야 할 개념들을 미리 정리하고 가겠습니다.

| 용어                          | 설명                         |
| --------------------------- | -------------------------- |
| `ApplicationEvent`          | 이벤트 객체. 전달하고자 하는 메시지 정보 포함 |
| `ApplicationEventPublisher` | 이벤트 발행자. 이벤트를 시스템에 알림      |
| `ApplicationListener`       | 이벤트 수신자. 이벤트 발생 시 동작 수행    |
| `@EventListener`            | 어노테이션 기반 이벤트 리스너 선언 방법     |

기본적인 구조는 아래와 같습니다.

```text
+---------------------------+
|   ApplicationContext      |
|  (이벤트 발행 & 수신)      |
+---------------------------+
     ▲                 ▲
     |                 |
     |         +---------------------+
     |         |  ApplicationListener|
     |         +---------------------+
     |
+--------------------------+
| ApplicationEventPublisher|
+--------------------------+
           ▲
           |
+--------------------------+
|    사용자 정의 이벤트    |
|   (ApplicationEvent)     |
+--------------------------+
```

우리가 직접 구현하고 처리할 수 있는 건 ApplicationContext 를 제외한 나머지입니다. 우리가 정의한 이벤트가 ApplicationEventPublisher 를 통해 발행 되었다면, ApplicationContext 가 이 이벤트를 발행해서 적절한 ApplicationListener 로 전달하죠.

#### 🎉 기본 예시

```kotlin
// 사용자 정의 이벤트
class UserRegisteredEvent(
    source: Any,
    val username: String
) : ApplicationEvent(source)
```

```kotlin
// 이벤트 리스너 (구현체 방식)
@Component
class UserRegisteredListener : ApplicationListener<UserRegisteredEvent> {
    private val log = getLogger()

    override fun onApplicationEvent(event: UserRegisteredEvent) {
        log.info("📥 [Listener] 새 유저 등록: ${event.username}")
    }
}
```

```kotlin
// 이벤트 발행
@Service
class UserService(val publisher: ApplicationEventPublisher) {

    fun registerUser(username: String) {
        println("✅ 유저 등록 처리 중: $username")
        publisher.publishEvent(UserRegisteredEvent(this, username))
    }
}
```

구현체로 처리할 수도 있지만, 보통은 일반적으로 `@EventListener` 어노테이션을 사용합니다. 해당 어노테이션은 내부적으로 `ApplicationListener` 를 대체하죠. 해당 방식은 여러 이벤트를 하나의 클래스에서 처리할 수 있습니다.

```kotlin
@Component
class EventHandler {
    
     private val log = getLogger()

    @EventListener
    fun handle(event: UserRegisteredEvent) {
        log.info("📥 [@EventListener] 유저 등록됨: ${event.username}")
    }
}
```

#### 🎉 심화 예시

비동기 이벤트 또한 사용 가능합니다. `@Async` 어노테이션을 쓰면 되죠. 물론 그 전에 `@EnableAsync` 어노테이션을 활성화 시켜야 합니다.

```kotlin
@Component
class AsyncEventHandler {

    private val log = getLogger()

    @Async
    @EventListener
    fun handleAsync(event: UserRegisteredEvent) {
        Thread.sleep(1000)
        log.info("⏱ [비동기] 유저 등록 알림: ${event.username}")
    }
}

@Configuration
@EnableAsync
class AppConfig
```

@Async는 스프링의 TaskExecutor를 통해 이벤트를 별도 스레드에서 처리합니다.  
이벤트 수신자의 실행이 느릴 경우에도 메인 로직은 지연되지 않죠.

조건에 따라 이벤트를 처리할 수도 있습니다. `SpEL` (Spring Expression Language) 를 통해 조건식을 설정할 수 있어요.

```kotlin
@EventListener(condition = "#event.username == 'admin'")
fun handleAdminUser(event: UserRegisteredEvent) {
    log.info("🔒 관리자 등록 감지: ${event.username}")
}
```

이렇게 `SpEL` 을 활용하면, 복잡한 조건문을 처리할 필요가 없어집니다. 이벤트 조건에 따라 다른 리스너를 호출할 수 있어요.

> 💡음 다 좋은데...정말 순수하게 트랜잭션하고 분리하기는 어렵지 않나요? Spring 의 @Transactional 어노테이션은 메소드가 마지막까지 모두 실행되어야 트랜잭션 commit 이 일어날텐데 아무리 @Async 을 날린다고 해도 막상 트랜잭션이 실패해 Rollback 처리 되면 어떻게 해야하나요? 트랜잭션의 상태에 따라 이벤트를 조건적으로 보내고 싶어요.

이런 경우에는 `TransactionalEventListener` 기능을 사용할 수 있습니다. 트랜잭션이 commit 된 이후에만 이벤트를 발행하죠.

```kotlin
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
@Async
fun handleAfterCommit(event: UserRegisteredEvent) {
    log.info("📨 트랜잭션 커밋 후 알림: ${event.username}")
}
```

이 경우 @Async 를 같이 쓰는 것을 좀 더 추천드립니다. 트랜잭션이 커밋 된 이후에 조건적으로 다른 스레드에 이벤트를 비동기적으로 처리하는 패턴이죠. 트랜잭션 안정성은 @TransactionalEventListener 에 AFTER_COMMIT 조건을 주는걸로도 보장되지만, 비즈니스의 성능을 아예 분리시켜버릴 수 있으니까요.

```kotlin
@Transactional
fun order() {
    orderRepository.save(order)
    eventPublisher.publishEvent(OrderCreatedEvent(...))  // AFTER_COMMIT
    // 커밋은 빠르게 끝났지만...
}
```

예를 들어 이런 상황에서 만약 커밋은 빨리 끝났지만, 이메일 전송이나 푸시알림 등 무거운 작업을 한다면 아무리 커밋 이후라고 해도 요청의 응답이 느려질 수 있습니다. 그러나 @Async를 함께 붙이면, 트랜잭션 커밋 직후에 백그라운드 스레드에서 리스너 로직을 비동기로 실행할 수 있어 응답 성능도 향상됩니다.

#### 🎉 이벤트 생명주기 요약 및 주의할 점

지금까지 ApplicationContext 에서 이벤트를 처리하는 방법에 대해 상세히 알아봤습니다. 이벤트의 생명주기를 요약하면 아래와 같습니다.

| 단계            | 설명                                           |
| ------------- | -------------------------------------------- |
| 1️⃣ 이벤트 객체 생성 | `UserRegisteredEvent("admin")`  |
| 2️⃣ 발행        | `publisher.publishEvent(event)`   |
| 3️⃣ 수신자 탐색    | `@EventListener`, `ApplicationListener` 등 탐색 |
| 4️⃣ 조건 평가     | (조건이 있다면 평가 후 실행 여부 결정)     |
| 5️⃣ 메서드 실행    | 동기/비동기 방식으로 호출    |
| 6️⃣ 예외 처리     | 기본적으로 예외는 전파되지 않음 (비동기라면 로깅만)    |

하지만 이 기능을 쓰기 위해선 몇가지 주의해야 할 게 필요합니다. @Async 는 반드시 @EnableAsync 를 선언해야 합니다. 그리고 이벤트 리스너 내 예외는 기본적으로 전파되지 않습니다. 또한 비동기 이벤트에서 트랜잭션 의존 처리 시 조심해야 하고, Bean 등록 순서에 따라 리스너가 등록되지 않을 수도 있습니다.

여기서 이벤트 리스너가 Bean 등록 순서에 영향을 받을 수 있다는 문장에 주목 해 볼까요? 종종 리스너가 ApplicationContext 에 등록되기 전에 이벤트가 먼저 발행되는 경우 발생합니다.

Spring은 ApplicationContext 초기화 시 `@Component`, `@Bean` 등으로 선언된 Bean들을 등록합니다. `@EventListener` 또는 `ApplicationListener` 로 등록된 Bean은 이 시점에 리스너로 등록됩니다. 이후 `publishEvent(...)`가 호출되면, 이미 등록된 리스너에게 이벤트가 전달됩니다.

```kotlin
@Configuration
class InitConfig(val publisher: ApplicationEventPublisher) {

    @Bean
    fun triggerOnStartup(): String {
        publisher.publishEvent(MyEvent("초기화 중 이벤트 발생!"))
        return "OK"
    }
}
```

이 코드는 Spring이 모든 빈을 다 등록하기 전, 즉 리스너가 ApplicationContext에 fully 등록되기 이전에 이벤트를 발행합니다.
이때는 리스너가 아직 없어서 이벤트가 무시되거나 도달하지 않습니다.

하지만 이렇게 @Bean 을 초기화하는 시점에 이벤트를 발행하고 싶을 수도 있죠. 이 경우에는 `ApplicationRunner` 나 `CommandLineRunner` 를 사용하는 것이 좋습니다.

```kotlin
@Component
class StartupEventTrigger(val publisher: ApplicationEventPublisher) : ApplicationRunner {
    override fun run(args: ApplicationArguments?) {
        publisher.publishEvent(MyEvent("이제 안전하게 이벤트 발행"))
    }
}
```

이들은 ApplicationContext 가 완전히 초기화 된 후 실행되므로 안전합니다.

또 다른 방법으로는 SmartInitializingSingleton 을 사용하는 것입니다. 모든 싱글톤 Bean 이 초기화 된 후 호출되는 특수 콜백 인터페이스 입니다.

```kotlin
@Component
class SafeEventPublisher(
    private val publisher: ApplicationEventPublisher
) : SmartInitializingSingleton {
    override fun afterSingletonsInstantiated() {
        publisher.publishEvent(MyEvent("모든 Bean 등록 후 안전한 발행"))
    }
}
```

### 🌐 ApplicationContext 의 기능 : 국제화

#### 🌐 국제화 (i18n) 기능 개요

Spring 은 ApplicationContext 를 통해 국제화 (i18n) 을 지원하며, 개발자가 리소스 메시지를 언어별로 분리하고, 런타임에 적절한 언어를 선택하여 메시지를 출력할 수 있도록 해줍니다.

이 기능은 다양한 지역의 사용자가 다양한 언어를 통해 애플리케이션을 사용할 수 있도록 메시지, 날짜, 숫자 등을 지역화 시켜줄 수 있습니다. Spring은 이 기능을 ApplicationContext 내부의 MessageSource라는 컴포넌트로 제공합니다.

#### 🌐 기본예시

```text
ApplicationContext
   └── MessageSource
         ├── messages.properties (기본)
         ├── messages_ko.properties
         ├── messages_en.properties
         └── messages_ja.properties ...
```

ApplicationContext는 MessageSource를 상속하고 있어 메시지 국제화 기능을 기본 제공합니다. Spring Boot에서는 자동으로 MessageSource Bean이 등록되며, messages.properties 파일을 읽습니다.

```yaml
spring:
  messages:
    basename: messages
    encoding: UTF-8
```

application.yml 의 옵션 값에 basename을 어떻게 지정 해 주느냐에 따라 접두어가 달라지는데, 해당 접두어에 대해 properties 파일들이 자동 매핑됩니다. (messages -> messages_ko.properties, messages_en.properties)

해당 설정 파일들의 기본 위치는 `src/main/resources/messages*.properties` 입니다.

```text
// messages.properties
greeting=Hello
farewell=Goodbye
```

```text
// messages_ko.properties
greeting=안녕하세요
farewell=안녕히가세요
```

```kotlin
@Component
class GreetingService(@Autowired private val messageSource: MessageSource) {

    fun greet(locale: Locale): String {
        return messageSource.getMessage("greeting", null, locale)
    }
}
```

```kotlin
val greeting = greetingService.greet(Locale.KOREA)
println(greeting) // 출력: 안녕하세요
```

#### 🌐 심화예시 : 웹 애플리케이션에서 자동 언어 감지

Spring MVC 는 HTTP 요청의 Accept-Language 헤더를 통해 언어를 자동 감지합니다. 이를 위해선 LocaleResolver와 LocaleChangeInterceptor 컴포넌트를 사용할 수 있습니다.
  
먼저 LocalResolver 부터 살펴 보겠습니다. LocaleResolver 는 클라이언트의 요청에서 언어 정보를 추출하는 전략 **인터페이스** 입니다. 기본적으로는 AcceptHeaderLocaleResolver 를 사용합니다. 앞서 설명 드렸다시피 Accept-Language 헤더를 사용하고 무상태성을 가집니다.

하지만 상황에 따라 상태를 가지게 하고 싶을 수 있습니다. 이 경우 SessionLocaleResolver (세션 기반), 혹은 CookieLocaleResolver (쿠키 기반) 을 사용할 수 있습니다.

```kotlin
@Configuration
class WebConfig : WebMvcConfigurer {

    @Bean
    fun localeResolver(): LocaleResolver {
        val resolver = SessionLocaleResolver()
        resolver.defaultLocale = Locale.KOREA
        return resolver
    }
}
```

이 경우엔 세션마다 언어 상태를 유지할 수 있어 사용자 마다 독립적인 언서 설정이 가능합니다.

LocaleChangeInterceptor 는 HTTP 요청 파라미터로 Locale을 변경할 수 있도록 지원합니다.

```kotlin
@Bean
fun localeChangeInterceptor(): LocaleChangeInterceptor {
    val interceptor = LocaleChangeInterceptor()
    interceptor.paramName = "lang"  // e.g., /hello?lang=en
    return interceptor
}

override fun addInterceptors(registry: InterceptorRegistry) {
    registry.addInterceptor(localeChangeInterceptor())
}
```

이제 브라우저에서 ?lang=ko, ?lang=en 과 같이 요청하면 자동으로 언어가 변경됩니다.

> ⚠️ SessionLocaleResolver 또는 CookieLocaleResolver 와 함께 사용해야 언어 변경이 유지됩니다.

#### 🌐 심화예시 : 다중 MessageSource 설정

application.yml 에서 spring.message.basename 속성은 단일 prefix 만 가능합니다. 하지만 @Bean 을 통한 설정에서는 여러개의 메시지 소스를 조합할 수 있습니다.

```kotlin
@Bean
fun messageSource(): MessageSource {
    val source = ReloadableResourceBundleMessageSource()
    source.setBasenames(
        "classpath:messages",
        "classpath:errors",
        "classpath:labels"
    )
    source.setDefaultEncoding("UTF-8")
    source.setCacheSeconds(10)  // 개발 중 변경 반영을 위한 캐시 무효화 주기
    return source
}
```

이 경우 여러 파일을 계층적으로 조합 할 수 있습니다. 또한 파일 분리를 통해 도메인 별 메시지 관리를 할 수 있습니다.

`ReloadableResourceBundleMessageSource` 는 기본 캐시 기능을 제공하여, 파일 변경 후 자동 반영이 가능합니다.

```kotlin
source.setCacheSeconds(0) // 항상 새로 읽음 (개발 중에만 사용)
```

운영 환경에서는 캐시 주기를 길게 설정하는 것이 좋습니다. (예: 3600초)

#### 🌐 주의할 점

당연히 해당 기능 또한 주의해야 할 사항이 존재합니다. 표로 한번 정리 해 보았습니다.

| 항목  | 설명 | 예시 / 해결책 |
| ---- | ------ | ------------- |
| ✅ 키 누락 시 예외  | 존재하지 않는 메시지 키는 예외 발생   | `getMessage(code, args, defaultMessage, locale)` 사용 |
| ✅ 다국어 키 통일 | 언어 파일 간 키 일관성 유지 필요   | 메시지 키 추출기/자동화 도구 활용   |
| ✅ `LocaleResolver` 필수 | 언어 변경 지원하려면 `SessionLocaleResolver` 등 필요 | 기본은 `AcceptHeaderLocaleResolver` (변경 불가)  |
| ✅ 리소스 캐시  | 변경된 메시지가 반영 안 될 수 있음   | `ReloadableResourceBundleMessageSource` 사용 + 캐시 조절  |
| ✅ 프로파일 분기 불가 | 메시지는 프로파일마다 분리할 수 없음  | 조건 메시지를 코드 내부에서 분기    |
| ✅ 테스트 환경 차이  | Locale 기본값이 테스트와 서버에서 다를 수 있음   | 명시적으로 `Locale` 지정 필요 |

### ⚙️ ApplicationContext 의 기능 : Envorinment 관리

#### ⚙️ Environment 개요

Spring은 애플리케이션 전반의 구성을 관리하기 위해 핵심 인프라 객체들을 제공합니다. 그중 ApplicationContext는 핵심 컨테이너, Environment는 실행 환경의 설정값 추상화를 담당합니다.

이 둘은 서로 독립적이지만, 서로 밀접하게 연결되어 있어 함께 이해해야 실무에서 유용하게 활용할 수 있습니다.

Environment는 애플리케이션의 실행 환경을 추상화하는 컴포넌트로, 프로퍼티(property)와 프로파일(profile) 을 관리합니다. 구성요소는 아래와 같습니다.

| 구성 요소   | 설명  |
| --------------- | -------------------------------- |
| PropertySources | 설정 값들 (application.yml, 환경 변수 등) |
| Profiles        | 현재 활성화된 설정 그룹 (dev, prod 등)  |

ApplicationContext는 내부적으로 Environment를 포함하고 있습니다. 즉, context.environment를 통해 Environment에 접근 이 가능합니다. `@Value`, `@ConfigurationProperties` 등은 내부적으로 Environment를 통해 값을 주입하게 됩니다.

#### ⚙️ 사용 예시

간단하게 Environment 컴포넌트의 기능을 요약 해 보겠습니다.

| 기능             | 예시                                           |
| -------------- | -------------------------------------------- |
| 프로퍼티 접근        | `env.getProperty("spring.application.name")` |
| 프로파일 분기        | `env.acceptsProfiles("prod")`                |
| 커스텀 프로퍼티 소스 등록 | 시스템 속성, 외부 파일 등 추가 가능                        |

```yaml
spring:
  profiles:
    active: dev

my:
  greeting: "Hello from dev"
```

만약에 application.yml 파일을 이렇게 구성했다고 가정 해볼게요.

```kotlin
@Component
class MyService(@Autowired val env: Environment) {

    @PostConstruct
    fun init() {
        println("✅ 환경 설정: ${env.getProperty("my.greeting")}")
    }
}
```

```shell
✅ 환경 설정: Hello from dev
```

그럼 Environment 컴포넌트를 통해 우리가 지정했던 property 를 가져올 수 있습니다.

Environment 의 activeProfile 에 따라 우리가 등록하고 싶은 Bean 을 결정할 수 있습니다. context.environment.activeProfiles 로 현재 Profile 을 확인할 수 있죠.

```kotlin
@Profile("dev")
@Component
class DevDataSource : DataSource { ... }

@Profile("prod")
@Component
class ProdDataSource : DataSource { ... }
```

또한 우리가 꼭 application.yml 등에 등록하지 않더라도 Java / Kotlin 코드 상에서도 등록할 수 있습니다.

```kotlin
val env = StandardEnvironment()
env.setActiveProfiles("prod")
env.propertySources.addFirst(
    MapPropertySource("custom", mapOf("custom.key" to "value"))
)
```

# 😊 Outro

지금까지 Spring 의 Core에 해당하는 Bean, ApplicationContext, Environment 에 대해서 조금 상세하게 알아보았습니다.  

이미 많은 분들이 잘 알고 계실 것이라 생각하고 있습니다만, 깊게 파고 들면 파고 들 수록 결국 모든 독립적인 개체들이 연결되어있다는 사실이 와닫네요. 우리가 Core 를 이해하는 만큼 우리의 Spring 애플리케이션이 더욱 안정적이고 유지보수하기 쉬워질 수 있습니다.

다음 주제는 JPA 로 넘어 가 보겠습니다. 9번의 시리즈 동안 아키텍처, 시큐리티, 코어 기능들에 대해 최대한 다뤘는데 정말 많이 쓰이는 JPA 에 대해 아직 다루지 않았더라구요. 다음 달에도 커피 한잔과 함께 돌아오겠습니다. 읽어주셔서 감사합니다.
