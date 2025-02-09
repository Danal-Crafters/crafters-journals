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


# 객체지향 관점에서의 DI와 IoC


# 동작과정


# 프레임워크 없이 DI, IoC 구현 해 보기


# Outro
