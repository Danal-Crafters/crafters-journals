[ 크래프터스 : \[알고쓰자\] Spring - 03 아키텍처 ]
===============================================================

<div align="center">   
  <img src="https://github.com/user-attachments/assets/38ae7c1f-9724-4658-9ff2-2bd282c9bb47" width="800"/>  
</div>

Intro 
=====

다시 돌아왔습니다. 12월 호를 벌써 작성해야 하는 시점이 되었네요. 😀

지난 알고쓰자 Spring 의 주제는 업데이트 내역이었습니다.  
Spring 의 변천사를 통해 IT 서비스 트랜드에 따라 Spring 프레임워크도 변화한다는 것을 알게 되었죠.

Spring 의 내부 구조에 대해 궁금증을 가졌던 시점은 2022년 어느날이었습니다.  
아마 아시는 분은 알겠지만, 처음부터 웹 개발자로 커리어를 이어가려는 생각이 있었던 건 아니었어요. 
그 당시엔 일단은 백엔드 개발 업무를 쳐내기 위해 Spring 공부를 시작했고, 프레임워커로 몇달 간 지내다보니 아키텍처를 모름으로써 생기는 이슈가 발생했어요.

Controller 영역으로 들어왔다는 것은 이미 Servlet 영역에서
HttpServletRequest 인스턴스를 HttpMessageConverter 가 확인했다는
이야기죠. 이 이후의 로직 흐름에 대한 디버깅은 크게 문제가 없었으나,
때로는 Controller 영역 이전에서 에러가 발생하곤 합니다. 혼자서 Spring
Security 를 셋업하고 공부하던 도중 이런 에러를 많이 마주했어요.

내부 아키텍처를 잘 이해하지 않고 사용한다면, Spring 에 전달 된 로직이
어떤 과정으로 처리되는 지 알기 어려워요. 아마 많은 개발자 분들이 잘 알고
계실 거라 생각하는 주제지만...혹시나 Spring 아키텍처를 잘 모르신다면
이번 포스팅이 도움이 되실 것이라 생각해요.

이번 포스팅은 크게 두 단계로 나뉠 것 같아요. 먼저 Web 애플리케이션의
요청을 처리하는 영역에 대해 알아본 후, 전체 아키텍처를 크게 돌아보며
마무리 될 것 같아요.

***알고쓰자 Spring!*** 이번 달도 한번 달려보겠습니다!

MVC 패턴? 
=========

<div align="center">
  <img src="https://github.com/user-attachments/assets/9549664d-4018-4611-b02d-1ee93e548976" />
</div>

MVC 패턴에 대해 아시나요? Model View Controller 패턴이라고 하죠. 유저는
View를 통해 결과물을 보고, Controller 에게 요청을 보내는 구조입니다.
Model 에서는 요청에 대한 결과물을 반환하죠.

Spring Boot 를 처음 셋업할 때 아마 Spring Boot Starter Web 을 포함
하셨을거에요. 해당 Dependency 에서는 아래의 요소들을 포함하고 있죠.

-   Embedded Tomcat

-   Spring MVC

-   Jackson

> 😎 : 잠시만 전 Webflux 를 가져왔는데요?  
> 😊👊

우리의 Spring 애플리케이션에서 MVC 패턴을 적용시켜주는 역할을 Spring
MVC가 수행합니다.

즉 Spring Boot Starter Web을 사용하셨다면, Tomcat 기반으로 동작하는
Spring MVC를 설정한 것과 마찬가지고, MVC 패턴을 사용하고 계신다는
의미입니다.

그럼 Spring MVC는 어떤 구조로 구현 되어있을까요?

Spring MVC
==========

<div align="center">  
  <img src="https://github.com/user-attachments/assets/27135142-e3c0-4db1-8bad-ac63ae011462" />
</div>

Spring MVC 의 대략적인 구조는 위와 같습니다. 간단하게 동작 과정을
정리하면 아래와 같죠.

1.  요청 정보를 Dispatcher Servlet 이 전달 받고 Handler Mapping 에게
    어떤 Controller 에 전달해야 할 요청인 지 확인받는다.

2.  Handler Mapping 으로 전달받은 정보를 토대로 Controller 에게 Request
    를 보낸다.

3.  Controller 는 결과값을 Dispatcher Servlet 에 전달하고, Dispatcher
    Servlet 은 View Resolver 에게 어떤 View 에 데이터를 전달할 지
    확인한다.

4.  View 에 결과물을 전달한다.

만약 REST Controller 를 사용한다면 ViewResolver 를 따로 사용하지 않고
그대로 데이터를 리턴하게 됩니다.

Spring MVC 의 한계?
===================

**MVC 패턴은 충분히 훌륭합니다.** 하지만 Spring 에서는 Spring MVC의
구조적 이슈로 인해 대용량 요청 처리에 불리하다는 단점을 가집니다.

<div align="center">  
  <img src="https://github.com/user-attachments/assets/a85a13dc-969a-47c7-a486-133e3623244b" />
</div>

Spring MVC는 기본적으로 **Tomcat** 을 사용합니다. Tomcat 은 Thread Pool
에 할당 된 스레드들을 요청에 할당하고, 요청이 마무리 되어 응답이
전달되면 그때 스레드가 다시 Thread Pool 에 반환되는 구조입니다. MVC 패턴
그 자체가 Thread Pool 을 쓴다! 라는 건 아니에요. 우리가 Spring MVC를
쓰고 Spring MVC 가 Tomcat 을 사용하기 때문에 이런 방법을 사용하죠.

이제 요청이 엄청나게 늘어난다고 가정 해 보겠습니다. **RPS (Response Per
Second)** 가 한 100 정도 된다고 생각해볼게요.

보통 Thread Pool 의 크기를 구하는 공식은 아래와 같습니다.

> **스레드 풀 크기 = 요청 량 X 요청 처리 평균 시간 X 안전계수**

요청 처리 평균 시간을 비관적으로 잡아서 **1초**라고 해볼게요. *(와
느리다...)* 안전 계수도 보수적으로 잡아서 **1.5**를 잡아보면 스레드 풀
크기는 ***100 X 1 X 1.5 = 150*** 이 됩니다.

서비스가 갑자기 잘 되어서 *RPS 가 200으로 늘어났습니다*. 그럼 스레드 풀
크기는 **300**이 되겠죠.

> 여기서 알 수 있는 사실은 ***'Spring MVC는 Tomcat 의 스레드 풀을
> 사용하는 만큼, 서비스 요청 량이 늘어나면 더 많은 시스템 자원을 사용할
> 수 밖에 없다.'*** 입니다.

결국 하드웨어 자원을 늘리는 형태로 문제를 해결할 수 밖에 없다는
의미인데, **비용 절감이 중요한 회사 입장에서는** 쉽게 받아들이기 어려운
의사결정일 수 있겠죠.

<div align="center">
  <img src="https://github.com/user-attachments/assets/86ebef1a-1126-4bbc-b74f-725df6d850a9" />  
</div>

또한 각각의 스레드들은 Blocking IO 방식을 사용합니다. 무작정 스레드풀을
늘리는 형태로 문제가 해결 가능할 진 몰라도 스레드 각각이 물고있는
로직들이 끝날 때 까지 스레드가 빠지지 않는다는 것은 언젠가 Thread Pool
의 스레드가 고갈 되는 경우가 생길 수 있다는 이야기죠. 결국 해결 방법은
자원을 높게 할당하는 것 뿐이라는 건데...좀 더 자원을 효율적으로 쓸 수
있는 방법은 없을까요?

Reactive
========

이 경우 Event Loop 를 사용하는 **Reactive 패턴**을 통해 시스템 자원을
절약하는 방법을 고려해볼 수 있습니다. Spring 에서 Reactive 패턴을
사용하고 싶다면 Spring Webflux 를 도입하여 사용해보실 수 있습니다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/baa6f28a-260c-4979-beec-3dbaf2adda9a" />
</div>

Webflux 는 기본적으로 Netty 를 기반으로 동작하는데, 내장 된 Event Queue
에 요청을 적재하고, Event Loop 쓰레드에서 요청을 처리하는 구조입니다.
Netty 는 Non Blocking I/O 방식을 사용하는 비동기 이벤트 기반 네트워크
애플리케이션 프레임워크입니다.

> 사실 Tomcat 3.1 부터는 비동기 논블로킹을 지원하긴 합니다. Webflux 를
> 처음 셋업했을 때 기본 옵션은 Netty 입니다.

Netty 에는 Event Loop 라는 개념이 존재하는데 Event Loop 는 요청마다
쓰레드를 생성하는 대신, 하나의 이벤트 루프가 여러 요청을 순차적으로
처리합니다. 이때 각각의 이벤트 루프는 비동기적으로 동작합니다.

각 이벤트 루프는 **이벤트큐** 를 가지고 있으며, 네트워크 요청이 들어오면 이벤트 큐에 작업을 추가합니다. 
이때 작업은 Task 형태로 이벤트 큐에 등록됩니다. 
큐에 등록된 작업들은 이벤트 루프에 의해 차례로 실행되며, 비동기 작업이므로 I/O 작업(예: 파일 읽기, 네트워크 통신)은 완료될 때까지 블로킹되지 않습니다.

Netty 에서 I/O 요청이 발생하면, 작업이 이벤트 큐에 등록되고, 비동기
적으로 작업이 처리됩니다. 작업이 완료되면, 완료 결과를 promise 객체에
전달하고, promise 는 결과값을 이벤트 루프에 전달하죠.

이벤트 큐 덕분에 Netty 는 Non Blocking I/O, 즉 요청에 대한 Blocking 을
하지 않고도 비동기적으로 작업을 처리할 수 있습니다. 작업이 끝날 때 까지
스레드를 물고 있는 Blocking I/O 보다는 자원을 효율적으로 사용하죠.

즉 Spring MVC, Blocking I/O 로 감당이 어려운 수준으로 트래픽이 발생하고,
마냥 하드웨어 자원을 늘리는 형태로 문제를 해결하기 어려운 경우 Webflux를
통해 기존 자원을 효율적으로 소비하도록 시도해볼 수 있습니다.


> **여기서 주의해야 할 점은 Webflux 자체는 좋은 시도이나, 러닝커브가
존재합니다.**  
> 기존의 Spring 애플리케이션 코드에 변화가 생기기도 해요.  
> 시스템 자원을 우선 늘려보고, 정말 필요하다고 판단이 될 때 시도하는 것을
고려하길 권장해요.  
> 우선 삽으로 최대한 땅을 잘 파보고 포크레인이 필요한 만큼 감당이 어렵다면
포크레인을 배워서 써야하는 법이니까요...  
> ***무작정 포크레인을 몰다가
아무것도 못 하는 경우가 생긴다면 그게 더 슬플겁니다...***


Spring Webflux 
==============

<div align="center">
  <img src="https://github.com/user-attachments/assets/6d54dd93-1773-4250-81af-8081a0913d97" />  
</div>

Webflux 내에서 HTTP Request 가 수신 된다면, Netty 의 이벤트 루프가
작업을 감지합니다. 작업은 Netty 이벤트 큐에 등록 되어 처리되고, 결과는
`Mono` 혹은 `Flux` 에 감싸져서 처리되죠.

그 외에 Spring 내부에서는 비슷하게 동작합니다. Dispatcher Servlet 이
아닌 HttpHandler 가 요청을 받고, WebHandler 가 어떤 Controller 를 사용할
지 선택한 후, Handler Adapter를 통해 Controller 에게 요청을 보내죠.
작업이 마무리되면 HandlerResultHandler 에게 요청을 전달하고, WebHandler
를 통해 HttpHandler로 Response 가 전달됩니다.

Layered Architecture 
====================

지금까지 WebMVC, Webflux 등을 사용하였을 경우 전반적인 Spring 의 구조에
대해 알아보았습니다.

하지만 이게 다는 아닙니다. 우린 지금까지 Controller 영역 외부의 이야기만
했으니까요. Controller 라고 부르는 컴포넌트는 MVC, Webflux 와 무관하게
Handler 로 간주됩니다. HTTP 요청의 URI 중에 어떤 핸들러와 매핑 되었는
지에 따라 특정 Controller 의 메소드가 호출됩니다. 호출 된 메소드를 타고
요청에 대한 결과값이 리턴되죠.

Layered Architecture 라는 용어를 들어보셨을 지 모르겠습니다. Spring
Framework 그 자체는 특정한 아키텍쳐를 종속하지 않습니다. 하지만 Layered
Architecture 를 지원하고, 일반적으로는 Layered Architecture 를 사용 하실
것입니다.

그럼 이 일반적으로 사용하는 Layered Architecture 란 무엇인가요? **말
그대로 계층으로 구분 된 구조를 의미합니다.** 예를 들면 **OSI 7 Layer**와
마찬가지죠. **OSI 7 Layer 는 각각의 계층들의 책임을 분리하는 구조를
가집니다. 다른 계층에 영향을 끼칠 수도 없습니다.** 실제로 데이터 패킷은
계층을 지날때마다, 송 수신 될 때마다 각 계층에 필요한 데이터 헤더가
추가되거나 제거됩니다. 계층마다 보는 데이터 헤더가 다르고, 필요한
데이터만 헤더에서 확인하고 다음 계층으로 넘기죠.

***여기서 다른 계층에 영향을 끼칠 수 없고, 각 계층의 책임이
분리되어있다는 말에 주목 해볼까요?***

Spring Boot 를 사용하신다면 기본적으로 Layered Architecture 를
채택했다는 전제를 가집니다. Spring Boot 자체가 사실 Spring Framework 의
초기 셋업을 모두 진행해주므로 아키텍쳐 또한 Layered Architecture 를
채택하고 있습니다.

<div align="center">
  <img src="https://github.com/user-attachments/assets/a61ddb0a-ab44-4d3f-bcc7-9d4267ec7ffa" />
</div>

Spring Boot는 네 가지 계층을 가집니다.

-   Presentation Layer

-   Business Layer

-   Persistence Layer

-   Database Layer

이전에 이야기했던 WebMVC, Webflux 구조는 **Presentation Layer**의
이야기입니다. Presentation Layer 에서는 HTTP 요청 처리, 인증 처리 등의
책임을 가지죠. 또한 JSON 데이터 등을 객체로 변환하거나 역으로 객체를
변환하는 역할을 합니다. `@Controller` 클래스 까지가 Presentation
Layer라고 할 수 있습니다.

**Business Layer**는 우리가 흔히 `@Service` 컴포넌트라고 부르는 클래스가
담당하는 영역입니다. `@Controller` 는 `@Service` 영역의 메소드를
호출하여 비즈니스 로직을 수행하죠.

**Persistence Layer** 는 Data Access Layer 라고도 합니다. 비즈니스
로직의 결과물을 데이터베이스에 전달하는 역할을 합니다.

**Database Layer**는 우리가 사용하는 Database 의 영역입니다. 실제로
Database 에 데이터를 CRUD 하는 역할을 하죠.

각각의 Layer 들은 서로 간섭하지 않습니다. 필요한 경우 서로의 API를
호출할 뿐이죠. `@Controller` 컴포넌트는 `@Service` 의 메소드를 호출할 뿐
직접적인 비즈니스 로직에 관여하지 않습니다. 이하도 마찬가지죠.

만약 우리가 웹페이지에서 요청하고자 하는 URI 가 바뀌거나, 요청의
결과물이 바뀌었다고 가정해보죠. 이 경우 v1 API를 v2로 바꾸거나 Return
Object를 변경하는 등 Presentation Layer는 변경될 수 있습니다. 하지만
그렇다고 비즈니스 로직이 변하는 건 아니니 Business Layer 이후로는 변화가
없습니다.

물론 코딩을 어떻게 했느냐에 따라 Business Layer의 메소드의 Return
Object가 바뀔 수는 있습니다. 이 경우 틀렸다고 볼 수는 없지만, 결합도가
강하다고 볼 수 있죠. 그러므로 가능하면 Layer 간에 데이터는 **DTO(Data
Transfer Object)**라는 객체를 통해 전달하는 패턴을 권장합니다.

```kotlin
@Controller
class TestController(
    private val testService : TestService
) {
  data class TestResponse (
      val name : String,
      val age : Int
  )

  @GetMapping("/api/v1/test")
  fun testApi() : ResponseEntity<TestResponse> {
    val result = testService.testMethod()
    return ResponseEntity().ok().body(TestResponse(
        name = result.name,
        age = result.age
    ))
  }
}

@Service
class TestServiceImpl(
    private val testRepository : TestRepository
) : TestService {
    
    data class TestEntityDTO (
        val name : String,
        val age : Int,
        val job : String,
        val 
    )
    
    fun testMethod() : TestEntityDTO {
        val entity = testRepository.findById(0L)
        return TestEntityDTO (
            name : entity.name,
            age : entity.age,
            job : entity.job
        )
    }
}

@Repository
interface TestRepository : JpaRepository<TestEntity, Long>
```

간단한 예시를 하나 보겠습니다. 특정 요청을 통해 엔티티 정보를 REST API
의 요구사항에 맞게 가공해서 전달 하는 예시입니다.

`@Controller`, `@Serivce`, `@Repository` 각각은 절대 직접적으로
연결되어있지 않습니다. (사실 할려면 할 수는 있겠지만, 권장되는 패턴은
아닙니다.) `@Controller` 는 인터페이스를 통해 `@Service` 와 통신하고,
`@Service` 또한 인터페이스를 통해 `@Repository` 와 통신하죠. 또한
`@Service` 의 testMethod 의 결과물은 DTO를 통해 `@Controller` 에
전달됩니다. 해당 DTO를 통해 `@Repository` 와 `@Controller` 의 직접적인
결합도를 낮출 수 있죠. `@Controller` 에서는 요구사항에 맞게 Response
클래스 를 수정하여 결과물을 전달할 수 있습니다. 만약 Response 클래스가
수정되는, 즉 **Presentation Layer**가 수정된다고 해도 **Business
Layer**, **Persistence Layer**는 영향이 없죠.


> **아마 Spring Boot 를 처음 입문 해 보신 분들은 Service Interface,
ServiceImpl 등의 패턴을 많이 보셨을 것입니다.**  
> Business Layer 를 기준으로 Service Interface 를 작성하는 이유가 바로 Presentation Layer 와 Business Layer 간 의존성을 낮추기 위함이에요.  
> 좀 더 깊게 나아가서 제공하고자 하는 기능에 따라 interface를 여러개로 분리할 수 있고, 해당 interface를 상황에 따라 다르게 구현해서 필요에 따라 다른 Service 컴포넌트를 연결 시켜줄 수도 있어요.


Outro
=====

지금까지 Spring MVC, Webflux 등의 구조, Spring Boot Layered Architecture
에 대해서 알아 보았습니다. 알고 쓰지 않는다면, 기계적으로 코드를
작성하거나 결합도가 높은 코드를 작성할 수 있는 개념들이라고 생각해요.

또한 Spring Framework 의 설계 철학을 엿볼 수 있어요. 이전 글에서
객체지향 5대 원칙에 대해 언급했는데, Spring Framework 는 이 객체지향 5대
원칙을 최대한 준수하는 구조로 설계 되어있어요. 우리가 직접적으로
프레임워크를 만들어서 쓰는 입장이 아니라면, 블랙박스를 잘 쓰는 것도
중요하지만 그 내부를 알고 쓴다면 더욱 유지보수하기 좋은 소프트웨어를
개발할 수 있지 않을까요?

다음 포스팅은 AOP 에 대해 다뤄보려고 합니다. 다음달에 또 봬요!! 😊

Reference
=========

<https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/mvc.html>

<https://www.geeksforgeeks.org/spring-mvc-framework/>

<https://www.geeksforgeeks.org/spring-boot-architecture/>

<https://medium.com/@diego.lucasilva/spring-webflux-under-the-hood-c6446c87ea84>

<https://medium.com/@burakkocakeu/spring-web-vs-spring-webflux-9224260c47b5>

------------
<div align="center">
    
|Profile|Link|Bio|
|--|--|--|
|<img src="https://github.com/user-attachments/assets/24d3f306-8014-41ae-98a3-a2f94bf6bdb9" width="100" height="100"/> | [github](https://github.com/swj9707) <br/> [LinkedIn](https://www.linkedin.com/in/woojin-son-18989b202/) |**"천천히, 하지만 꾸준히!"**|

</div>
