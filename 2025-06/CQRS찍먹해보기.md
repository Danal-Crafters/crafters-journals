# Intro

우리가 흔히 웹 애플리케이션 아키텍처를 구성할 때 크게 세 가지 구성을 가집니다. 웹 서버, 웹 애플리케이션 서버, DB 서버죠. 우리가 뭔가 기똥차게 구성한다고 해도 이 구성을 크게 벗어나지는 않습니다.

> ...웹 애플리케이션 서버 포트를 그대로 개방하는 행위를 한다고 해도 말리지는 않습니다만, 보안 부서에서 많은 질책을 받을 각오를 하셔야 합니다.

아직 개요 세션이니 각 구성요소들의 역할을 한번 짚고 넘어 가 보겠습니다. 웹 서버는 서버의 리소스를 제공해주는 역할을 합니다. 우리가 공개하기로 결정 한 리소스들을 공개할 수 있죠. 웹 페이지에 필요한 이미지도 그 중 하나입니다. 웹 애플리케이션 서버는 애플리케이션을 구동 시키는 서버에요. DB 서버는 DBMS 를 구동 시키는 서버죠.

웹 서버 없이 웹 애플리케이션 서버만 있어도 Spring 앱 하나만 서비스 한다고 생각 해 보면 돌아는 갑니다. 그렇게까지 권장하는 구성은 아닌 게, 사실 이렇게 구성한다고 해도 TLS 옵션을 사용해서 HTTPS 를 지원합니다. 단, 관리의 측면에서 권장되지 않습니다. 게다가 요청 포트를 그대로 노출 시켜야 하죠.

다시 본론으로 돌아와서 서비스를 출시하고 시간이 지나면, 처음에 생각했던 것 과는 다른 시선을 가지게 됩니다. 유저들이 어떤 요청을 많이 보내는 지 부터 해서 비즈니스 로직의 추가 요구사항이 나올 수 있죠. 점점 서비스는 복잡해질 수 있습니다. 단순 게시판을 서비스한다고 해도 마찬가지입니다. 처음엔 글을 올리고 댓글을 달다가 좋아요를 누르기 시작 할 것이고, 회원 등급제가 생기기 시작 하겠죠. 이 경우 우리는 단순하게 개발했던 **우리의 서비스를 반드시 개선해야만 하는 상황에 놓이게 됩니다.**

조금 더 구체적으로 들어 가 볼게요. 우선 트래픽이 특정 요청에 몰릴 수 있습니다. 특히 조회 요청에 상당히 많은 비중이 몰릴 거에요. 유저 정보 조회부터 게시글 조회 까지 웹 서비스를 브라우저에 띄우기까지는 많은 데이터를 읽어 와야하니까요.

또한 등록, 수정 등의 비즈니스 로직들이 점점 복잡해질 것입니다. 게시글 업로드 까지는 그렇다치는데 댓글, 대댓글, 좋아요가 들어가기 시작하는 순간 과거의 잘못 된 설계를 후회해야 하는 경우도 생기죠. (사실 그 당시에는 그게 최선이었을 겁니다. 갑자기 기능이 추가될 줄은 몰랐죠... 🤣)

가장 처음에 논했던 **단순한 웹 서버, 웹 애플리케이션 서버, DB 서버 구성**에서는 어떤 이슈가 생길 수 있을까요? **먼저 읽기에 병목이 일어날 수 있습니다.** 예를 들어서 우리가 게시판을 운영 중이라 가정 해 볼게요.

```kotlin
@Entity
class Post(
    @Id @GeneratedValue
    val id: Long = 0,

    val title: String,
    val content: String,

    @ManyToOne(fetch = FetchType.LAZY)
    val author: User,

    @OneToMany(mappedBy = "post", cascade = [CascadeType.ALL])
    val comments: List<Comment> = listOf(),

    val createdAt: LocalDateTime,
    val updatedAt: LocalDateTime,
)
```

이 구조는 Post 를 수정 및 등록 할 때 적절하지만, 만약 읽기 요청이 폭증하면 어떤 일이 벌어질까요?

```kotlin
data class PostSummary(
    val id: Long,
    val title: String,
    val authorName: String,
    val commentCount: Int
)
```

실제로 읽고 싶은 정보는 이게 다입니다. 하지만, JPA 로 조회한다고 가정 해 보면 Post 전체, 연관된 유저, Comment 등 반드시 필요하지 않은 정보들까지 로딩해야 합니다.

N+1 을 회피하기 위해 FetchJoin 을 쓰거나, 필요한 데이터만 가져오기 위해 DTO Projection 을 사용할 수 있지만, 쿼리 생성이 복잡 해 지고 성능 튜닝이 필요해집니다.

**또 다른 문제로는 비즈니스 로직의 복잡도가 늘어날 수록 도메인 중심으로 로직을 설계해야하는 요구가 생겨날 수 있습니다.** 예를 들어서 단순한 주문 로직을 구성해볼게요.

```kotlin
fun placeOrder(userId: Long, productId: Long, quantity: Int) {
    val user = userRepository.findById(userId)
    val product = productRepository.findById(productId)

    val order = Order(
        user = user,
        product = product,
        quantity = quantity,
        status = OrderStatus.PENDING
    )

    orderRepository.save(order)
}
```

정말 단순하게 유저가 상품을 주문한다. 라는 로직이죠? 하지만 서비스가 성장하면서 기획, 운영, 사업 측의 요구로 인해 도메인 지식이 개입 된 복잡한 비즈니스 규칙이 반영될 수 있습니다.

|요구사항|예시 비즈니스 로직|
|---|---|
|✅ 상품별 재고 차감|`product.decreaseStock(quantity)` 호출 필요|
|✅ 사용자 등급별 할인 적용|`if (user.isVIP()) applyDiscount()`|
|✅ 프로모션 쿠폰 소진 처리|쿠폰 유효성 검증 + 상태 변경|
|✅ 결제 수단별 제한|무통장입금 → Pending 상태, 카드결제 → 즉시 승인|
|✅ 주문 중복 방지|같은 상품을 5분 내 중복 주문 시 Reject|
|✅ 이벤트 적립금 지급|등급/이벤트 조건 충족 시 적립금 엔트리 발행|
이 경우 도메인 중심 설계를 통해 문제를 해결해야 합니다. 로직을 단순히 도메인 내부에 옮기는 것도 중요하지만, 이제 검증, 정책 판단, 외부 시스템 연동, 이벤트 처리 등의 다양한 규칙들이 몰릴 수 있죠.

마지막으로 서비스가 커져서 팀이 분리되기 시작하는 경우 이 구조가 문제가 될 수 있습니다. 너무 서비스가 잘 되어서 유저 도메인, 게시글 도메인, 검색 팀 등 서비스에 따라 팀이 분리되면 이런 단순한 구조에서는 서비스를 운영, 유지보수 하기 매우 어려워집니다. 하나의 서비스 내에서도 도메인 간에 비즈니스 요구가 달라지고 읽기, 쓰기의 관심사가 달라지죠.

조금 설명이 길었습니다만, CQRS 의 필요성에 대해 필요한 내용이라 조금 구구절절 설명 드렸습니다. 이번 포스팅의 주제는 CQRS 에 대해서 입니다. 재밌게 읽어주세요.

# CQRS 란?

## CQRS 개요

CQRS (Command and Query Responsibility Segragation) 은 명령 (Command) 와 조회 (Query) 의 책임을 분리하는 아키텍처 패턴입니다.

전통적인 애플리케이션에서는 동일한 모델을 사용해서 데이터를 수정하고 조회하는 일이 일반적입니다. 하지만 CQRS는 이 두 가지 역할을 분리하여, **명령은 데이터를 변경하는 책임만**, **조회는 데이터를 반환하는 책임만** 가지게 합니다.

- **Command**: 데이터를 변경하는 작업 (예: 등록, 수정, 삭제 등)
- **Query**: 데이터를 조회하는 작업 (예: 목록 보기, 상세 보기 등)

이 패턴은 Domain-Driven Design(DDD)에서 자주 사용되며, 복잡한 도메인 로직과 고성능 요구사항을 가진 시스템에서 특히 유용합니다.

## 왜 사용하는 걸까요?

그럼 CQRS 를 왜 사용하는 걸까요? 크게 세 가지 이유가 있습니다.

먼저 모델 복잡도 해소가 있습니다. 읽기/쓰기 요구사항이 복잡하거나 상이할 경우, 하나의 모델로 이 두 역할을 처리하면 유지보수가 어려워집니다. CQRS는 두 모델을 명확히 분리하여 각각의 목적에 최적화된 설계를 가능하게 합니다.

다음으로 성능과 확장성 향상이 있습니다. Command와 Query를 물리적으로 분리하면, 각각 독립적으로 확장할 수 있습니다. 예를 들어, 조회 트래픽이 많을 경우 Query 서비스만 수평 확장하면 됩니다.

마지막으로 보안과 권한 분리가 있습니다. 조회와 수정을 분리함으로써, 역할 기반 권한 관리가 더 직관적으로 설계됩니다.

## 반드시 사용해야 할까요?

꼭 그렇지만은 않습니다. 앞서 들었던 예시들은 결국 비즈니스의 성장을 가정하고 있습니다. 단순한 게시판 하나를 위해 CQRS 를 사용 할 필요는 없죠.

특히 서비스 초기에는 이런 구조를 고려하기 어려울 가능성이 매우 높습니다. 얼른 제품을 출시해서 시장의 반응을 봐야 할 테니까요. 아직 출시 되지도 않은 서비스에 벌써부터 다양한 장치들을 달기 시작하면 재미는 있겠지만....사업 측에서 매우 싫어할겁니다 하하...동료 개발자들도 아마 피곤해할 수 있죠.

## 언제 사용해야 할까요?

판단 기준이 모호한 경우도 있을 거에요. 초석이 나중에 끼치는 영향을 생각 해 보면, 가능하면 초반에 좋은 구조를 가져가는 게 나중에 고생을 하지 않을 수 있죠.

가볍게 판단 기준을 정리 해 보았습니다.

| 조건     | CQRS 도입 권장 |
| ------------------------------ | ---------- |
| 읽기/쓰기 요구사항이 명확히 다를 때 | ✅ |
| 조회 트래픽이 쓰기보다 압도적으로 많을 때   | ✅  |
| 도메인 로직이 점점 복잡해질 때  | ✅ |
| 비즈니스 요구사항 변경이 잦고 잦은 테스트가 필요할 때 | ✅ |
| 단순 CRUD, 빠른 프로토타이핑이 중요한 시점| ❌ No CQRS  |
| 개발 리소스가 부족하고 구조 유지가 어려울 때   | ❌ No CQRS  |

# 가볍게 CQRS 를 써 볼까요?

지금까지 우리는 CQRS 의 개념에 대해 알아보았습니다.  
반드시 CQRS 를 도입해야 하는 것은 아니지만, 나중에 도입을 하게 될 것을 대비해서 CQRS 를 지향하는 구조를 고려하는 게 도움이 됩니다.

우리가 자주 사용하는 Spring 으로 CQRS 를 단계적으로 도입 해 볼까요?

## 1. Command, Query API 분리

우선은 명령 (Command) 와 조회 (Query) 를 분리해야 합니다.

```kotlin
@PostMapping("/users")  // Command
fun registerUser(...) { ... }

@GetMapping("/users/{id}")  // Query
fun getUserDetail(...) { ... }
```

*요청을 먼저 명확히 나누는 것을 우선으로 합니다.* 이 과정에서 우리는 RESTful API 설계 원칙 중 하나를 활용하면 좋습니다.

GET, POST, PUT, DELETE 각각의 메소드 마다 규칙이 있지요? 우선은 HTTP Method 를 통해 요청들의 목적을 명확히 분리합니다.

혹시나 궁금해하실 분들을 위해 각 메소드들의 목적을 한번 정리 해 볼까요?

- GET: 조회 (받겠다)
- POST: 리소스 생성 (보내겠다)
- PUT: 리소스 전체 갱신(놓겠다/넣겠다)
- DELETE: 리소스 삭제 (지정한 서버의 파일을 삭제하겠다)

> 물론 상황에 따라서 (특히 보안 정책에 따라서는) POST 만 사용해야 하는 경우도 있습니다. 그렇다고 우리가 RESTful 을 모두 버릴 필요는 없습니다. URL 로 최대한 나눠 보죠 하하...

## 2. 비즈니스 로직의 Command, Query 분리

REST API 를 명확히 목적에 따라 나눴다면, Controller 레벨에서도 Command 와 Query 를 최대한 분리하는 게 좋습니다. 예를 들면 이렇게 말이죠.

```kotlin

class UserCommandController {
    @PostMapping("/users")  // Command
    registerUser(...) { ... }
}

class UserQueryController {
    @GetMapping("/users/{id}")  // Query
    fun getUserDetail(...) { ... }
}
```

이후 서비스 레이어를 분리해야 합니다. `UserCommandService` 와 `UserQueryService` 를 분리하여 각각 책임에 맞는 로직만 담당하도록 분리하죠.

```kotlin
class UserCommandService {
    fun register(...) { /* 등록, 수정 */ }
}

class UserQueryService {
    fun getDetail(...) { /* 조회 */ }
}
```

## 3. 읽기 / 쓰기 모델의 분리

다음으로 모델을 분리합니다. Command 에는 도메인 중심의 복잡한 모델을, Query 에는 DTO 기반의 경량 모델을 사용합니다.

```kotlin

class UserCommandService (
    private val userRepository : UserRepository
) {
    fun editUser(user : User) { /* 등록, 수정 */ }
}


data class UserDetailDto(val nickname: String, val imageUrl: String)

class UserQueryService(
    private val userRepository : UserRepository
) {
    fun getDetail(userId : UUID) : UserDetailDto { /* 조회 */ }
}
```

SELECT 문에 어떤 컬럼을 얼마나 넣느냐도 조회 성능에 영향을 끼칩니다. 그러므로 Query 역할의 메소드의 리턴 타입은 가능하면 필요한 필드만 담은 DTO 를 두는 게 좋습니다.

특별히 우리가 복잡한 장치를 두지 않더라도 여기까진 쉽게 할 수 있습니다. API 의 목적만 분리해도 상당히 깔끔해졌죠.  
이제 필요하다면, 읽기 저장소를 분리할 수 있습니다.

## 4. 읽기 저장소 분리

지금까지 잘 따라 와주셨습니다. 하지만 우리는 여전히 하나의 저장소에서 Command 와 Query 를 진행하고 있습니다.  
읽기 성능을 최적화 하기 위해 자원에 여유가 있다면 RDB(Read Replica) 나 Elasticsearch, Redis 등 조회 성능에 특화 된 저장소를 사용할 수 있습니다.

아래는 이벤트 기반 동기화 패턴 (Eventual Consistency) 를 적용한 예시입니다.

```kotlin
@PostPersist
fun onUserRegistered() {
    // Kafka/EventBus 발행 후 Redis에 조회용 모델 반영
}
```

## 5. 마이크로서비스 또는 독립 배포 구조로 확장

여기까지 잘 따라 와 주셨습니다. 이제 좀 더 상황이 복잡해진다면, 사용자 등록 서비스와 조회 서비스를 백엔드 애플리케이션 단위로 분리할 수도 있습니다. 흔히 말하는 마이크로서비스 (MSA) 패턴을 써야하는 상황에 사용하죠.

> ❓ 언제 MSA 패턴을 쓰는 게 좋을까요?  
>
> 조회 트래픽이 너무나 압도적으로 많아서 모놀리식 애플리케이션에서 더는 견딜 수 없는 경우, 그래서 서비스 자체를 분리하고 스케일 아웃(서비스 애플리케이션 확장 또는 서버 장비 확장)을 통해 트래픽을 분산해야 하는 경우 사용합니다.  
>
> 물론 스케일 업 (장비 사양 업그레이드) 를 통해서 처리할 수도 있겠지만, 너무나 조회 트래픽이 압도적으로 많은 경우엔 한 서버 장비, 한 애플리케이션에서 두 서비스가 함께 죽을 수도 있겠죠.

# 조금 더 깊게 들어가볼까요? : 소셜 피드 서비스 예시

지금까지는 좀 간략한 예시를 들었습니다. 아마 크게 어려움 없이 이해하실 것 같지만, 4번 부터는 마냥 개발자 혼자서 해결 할 수 있는 영역은 아닐 수도 있겠죠. 게다가 케이스가 딱히 와닫 지 않을 수 있을 것이라 생각이 드네요.

> 😊 물론 혼자서 AWS나 GCP, Azure 에서 테스트 해 볼 수는 있을겁니다. 집에 좋은 사양의 PC가 있다면 적당히 Docker 로 셋업해서 테스트 해 볼 수도 있죠. 테스트 해 보시기 전에 최대한 정보 얻고 가세요!

이번 세션에서는 구체적인 상황 예시를 통해 CQRS 를 도입 한 케이스를 소개 해 드리겠습니다.

**예를 들어서 우리가 소셜 피드 서비스를 운영하고 있다고 가정 해 볼까요?**

## 문제상황 : 읽기 병목

서비스 초기에 다음과 같은 도메인 중심의 JPA Entity를 사용했다고 가정 해 보겠습니다.

```kotlin
@Entity
class Post(
    @Id @GeneratedValue
    val id: Long = 0,

    val title: String,
    val content: String,

    @ManyToOne(fetch = FetchType.LAZY)
    val author: User,

    @OneToMany(mappedBy = "post", cascade = [CascadeType.ALL])
    val comments: List<Comment> = listOf(),

    val createdAt: LocalDateTime,
    val updatedAt: LocalDateTime,
)
```

이 구조는 POST /posts나 PUT /posts/{id}에서 포스트 등록/수정할 때는 적절합니다. 그러나 점점 사용자가 늘어나고, GET /feed 요청이 폭증하면서 다음과 같은 문제가 발생하죠.

```kotlin
data class PostSummaryDto(
    val id: Long,
    val title: String,
    val authorName: String,
    val commentCount: Int
)
```

우리가 실제로 필요한 데이터는 이 정도입니다. JPA로 조회 시 Post Entity 전체 + 연관된 User, Comment까지 로딩해야 합니다.

```kotlin
postRepository.findAll() 
```

SELECT 하는 컬럼의 갯수도 문제지만, @ManyToOne 을 주목해볼까요? N+1 문제가 발생할 수 있습니다.

> 💡 여기서 N+1 문제란? 1번 조회해야할 것을 N개 종류의 데이터 각각을 추가로 조회하게 되서 총 N+1번 DB조회를 하게 되는 문제입니다.

N+1 문제를 피하기 위해서 fetch join 혹은 DTO Projection 을 사용하지만, 복잡한 쿼리가 생성되고 성능 튜닝이 필요할 수 있습니다.

> 일반 join은 연관 엔티티를 조건에만 사용하고 로딩은 하지 않지만, fetch join은 연관 엔티티를 즉시 로딩해서 N+1 문제를 방지하는 데 사용됩니다.

```sql
select p.id, p.title, u.name, count(c.id)
from post p
join user u on p.author_id = u.id
left join comment c on c.post_id = p.id
group by p.id
```

사실은 복잡하지 않지만, JPA 를 사용하고 있는 입장에서는 이 쿼리 하나를 생성하기 위해 여러모로 귀찮은 작업을 해야 합니다. 사실 우리가 원하는 건 저 네가지의 데이터 뿐인걸요.

> ??? : 이래서 그냥 NativeQuery 쓰면 좋지 않나요? 쉽게 해결될 것 같은데...

NativeQuery 를 쓰는 상황 자체가 꼭 나쁘진 않지만, 때로는 저 쿼리보다 훨씬 복잡한 쿼리를 작성해야 하죠. 또한 JPA 의
엔티티 중심 설계 (객체 지향적인 도메인 모델), 그리고 영속성 컨텍스트를 제대로 살리지 못 할 수 있습니다.

> DB 에 종속적이다는 말은 하지 않겠습니다...DBMS 를 바꿔야 하는 상황은 자주 일어나진 않지만, 그것만이 JPA 를 쓰는 이유는 아니니까요.
>
> 물론 JPA 는 만능은 아니라고 생각합니다. 도메인 모델을 개발자가 얼마나 깊게 관여하느냐에 따라 JPA 의 활용 가능성이 달라진다 생각합니다.

## 해결방법 : 읽기 모델 분리

만약 CQRS 를 도입하게 된다면 조회 전용 모델을 따로 만들어버릴 수 있습니다.

```kotlin
// 단순 DTO 기반 읽기 전용 모델
data class PostFeedView(
    val postId: Long,
    val title: String,
    val authorName: String,
    val commentCount: Int,
    val thumbnailUrl: String?,
    val createdAt: String
)
```

이를 Redis나 Elasticsearch, 혹은 읽기 전용 DB 에 별도로 유지하면서 조회 전용 API는 이 저장소를 사용하게 됩니다.  
이 경우 장점은 아래와 같습니다.  

- 기존의 DB에서 복잡한 JOIN 없이 데이터를 빠르게 조회할 수 있습니다.
- 필요한 필드만 적재하므로 네트워크 비용이 절감됩니다.
- 조회 트래픽이 급증한다고 해도 쓰기 트랜잭션에 영향을 주지 않을 수 있습니다.

# 더 구체적인 예시 : CQRS + 이벤트 소싱 도입 시나리오

지금까지 구체적인 하나의 케이스에서 CQRS 를 도입해서 읽기 병목을 줄이는 케이스를 한번 살펴보았습니다.  
하지만 조금 더 깊게 들어가볼 필요가 있습니다. 지금까지는 읽기 모델을 분리하는 것 까지 진행했고, 실제로 읽기 모델을 다른 저장소로 분리하는 경우, 하나의 트랜잭션에서 모두 처리하기 보다는 이벤트 소싱이라는 방법을 사용합니다.

## 여기서 잠깐! 이벤트 소싱이란 무엇일까요?

이벤트 소싱은 “지금 이 순간의 상태”만 저장하는 대신, 무슨 일이 있었는지를 하나하나 기록해 두는 방식입니다. 예를 들어 은행 계좌를 생각해 보면, 잔액만 저장하는 전통적 방식은 **'현재 100만 원'** 만 남아 있지만, 이벤트 소싱은 **'10만 원 입금'**, **'5만 원 출금'** 같은 모든 거래 내역을 순서대로 남겨 둡니다.

여기서 입금, 출금 등의 일들을 이벤트로써 기록하게 됩니다. 그리고 상태를 재구성해야 하는 경우 이벤트를 처음부터 차례로 재생하듯 불러와서 현재의 상태를 계산하게되죠. 우리가 타이핑을 할 때 Ctrl + Z 기능을 쓰잖아요? 문장 하나를 만들 때 이 타이핑 하나하나가 이벤트라고 생각하면 조금 더 이해하기 쉬울까요?

이벤트 소싱은 우리가 뭔가 상태를 만들 때 이력을 보존할 수 있고, 디버깅이나 롤백이 쉽다는 장점이 있습니다. 이게 CQRS 랑 무슨 상관인가 생각 해보면, 쓰기 모델과 읽기 모델이 분리 되어있는 CQRS 에서 서로의 로직을 비동기적으로 처리할 수 있다는 장점이 있습니다. 구체적인 예시를 한번 볼까요?

## 소셜 피드 서비스에 이벤트 소싱을 더해볼까요?

소셜 피드 서비스 구현 시나리오를 아래와 같이 잡아 보겠습니다.

- 서비스는 유저가 게시글(Post)을 작성하면, 다른 유저들이 홈 피드에서 해당 글을 조회할 수 있습니다.
- 피드에서는 가장 최신의 피드 20건만 조회합니다.
- 피드에서는 100자 제한이 걸린 미리보기 (Snippet) 을 지원합니다.
- 게시글 등록은 복잡한 도메인 로직을 거치며,
- 조회는 빠르고 단순한 구조(피드용 View 모델)를 통해 제공됩니다.

### Command 모델

```kotlin
@Entity
class Post(
    @Id @GeneratedValue
    val id: Long = 0,

    val title: String,
    val content: String,

    @ManyToOne(fetch = FetchType.LAZY)
    val author: User,

    @OneToMany(mappedBy = "post", cascade = [CascadeType.ALL])
    val comments: List<Comment> = listOf(),

    val createdAt: LocalDateTime,
    val updatedAt: LocalDateTime,
) {
    fun publish() {
        // 도메인 규칙 적용 가능 (금지어 검사, 사용자 차단 여부 등)
        require(title.isNotBlank()) { "제목은 필수입니다" }
        require(content.length <= 5000) { "본문은 5000자 이하로 제한됩니다" }

    }
}
```

앞서 말씀드렸던 엔티티 모델에 뭔가 로직을 하나 추가했습니다. 게시글을 작성하는 경우 모든 비즈니스 로직을 해당 로직에서 처리하죠.

이렇게 도메인 중심으로 로직을 구현하면, 도매인 객체 내부에서 정책을 캡슐화 할 수 있다는 장점도 있습니다. 물론 복잡도가 높아질 수록 테스트 코드는 충분히 작성 되어야 하죠.

### Query 모델 (단순 피드 뷰 용도)

@RedisHash("PostFeed")
data class PostFeedView(
    @Id val postId: Long,
    val authorName: String,
    val title: String,
    val snippet: String,
    val createdAt: String
)

해당 모델은 피드에 필요한 최소한의 정보만을 담았습니다. 스토리지는 Redis 를 사용한다고 가정 해 보겠습니다.

@RedisHash("PostFeed") 애노테이션으로 Redis Hash 구조에 저장하게 됩니다.

```kotlin
interface PostFeedViewRepository : CrudRepository<PostFeedView, Long> {
    fun findTop20ByOrderByCreatedAtDesc(): List<PostFeedView>
}
```

Spring Data Redis 의 CrudRepository 를 사용해서 최신 피드 20건을 조회할 수 있는 메소드를 정의 해 둡니다.

### 게시글 등록 시 이벤트 발생

```kotlin

data class PostPublishedEvent(
    val postId: Long,
    val authorId: Long,
    val createdAt: LocalDateTime
)

@Service
class PostCommandService(
    private val postRepository: PostRepository,
    private val eventPublisher: ApplicationEventPublisher
) {
    fun writePost(cmd: WritePostCommand) {
        val post = Post(cmd.authorId, cmd.title, cmd.content).apply { publish() }
        postRepository.save(post)
        eventPublisher.publishEvent(
            PostPublishedEvent(post.id, post.authorId, post.createdAt)
        )
    }
}
```

`postRepository` 에 저장하기 까지 복잡한 도메인 로직을 거쳤습니다. 저장이 완료되는 대로 Redis 업데이트에 필요한 최소한의 정보 postId (게시글 식별 ID), createAt (작성 일자), authorId (작성자 식별 ID) 만 담습니다.

### 이벤트 리스너에서 조회 모델 갱신

```kotlin
@Component
class PostFeedProjectionListener(
    private val postRepository: PostRepository,
    private val userService: UserService,
    private val postFeedViewRepository: PostFeedViewRepository
) {
    @EventListener
    fun on(event: PostPublishedEvent) {
        // 1. 원본 Post와 authorName 조회
        val post = postRepository.findById(event.postId).orElseThrow()
        val authorName = userService.getUserName(post.authorId)
        // 2. snippet 생성
        val snippet = post.content
            .take(100)
            .let { if (post.content.length > 100) "$it…" else it }
        // 3. Redis용 뷰 모델 구성
        val view = PostFeedView(
            postId    = post.id,
            authorName= authorName,
            title     = post.title,
            snippet   = snippet,
            createdAt = event.createdAt.format(DateTimeFormatter.ISO_DATE_TIME)
        )
        // 4. Redis에 저장 (upsert)
        postFeedViewRepository.save(view)
    }
}
```

4번 save 메소드가 실행되기 전 까지 Redis 에 들어가게 될 View 데이터를 가공합니다. 이벤트로 전달 된 데이터가 실제로 DB 상에 있는 지 확인한 후, 해당 데이터를 토대로 View 데이터를 만들어 Redis Hash 에 저장합니다.

만약 동일한 이벤트가 재수신 된다고 해도 덮어쓰기 방식 (Idepotence) 으로 처리됩니다.

### 클라이언트 조회

```kotlin
@RestController
class FeedController(
    private val postFeedViewRepository: PostFeedViewRepository,
    private val redisTemplate: RedisTemplate<String, String>
) {
    @GetMapping("/feed")
    fun getFeed(): List<PostFeedView> {
        // ▼ 방법 1: Repository 메서드 사용
        return postFeedViewRepository.findTop20ByOrderByCreatedAtDesc()
    }
}
```

이제 조회 로직을 실행 해볼까요? 실제 테이블이 아닌 Redis 를 참조해서 할당 된 View 의 Redis Hash 를 조회해서 반환합니다.

### 핵심 포인트

이렇게 쓰기 (RDB) 와 읽기 (Redis) 분리로 DB 락 경쟁이나 N+1 문제를 해소할 수 있습니다. 예시에선 Redis 를 사용했잖아요? Redis Hash 로 조회하므로 훨씬 빠르게 조회할 수 있죠.

또한 이벤트 소싱 (PostPublishedEvent) 를 통해 비동기적으로 데이터를 동기화 시킬 수 있습니다. 또한 읽기 부하가 크다면, 읽기 저장소만 스케일 아웃 해서 트래픽에 대응할 수 있죠.

그리고 이런 이벤트 소싱 패턴은 최종적 일관성 (Eventual Consistency) 를 수용할 수 있습니다. 최종적 일관성은 분산 시스템에서 '모든 업데이트가 전파될 때까지는 잠깐 불일치가 있을 수 있지만, 더 이상의 업데이트가 없으면 결국에는 모든 복제본이 같은 상태로 수렴한다' 는 보장 모델입니다.

더이상 업데이트가 없으면 이벤트가 모두 반영되고 나면 모든 읽기 노드에서는 (서버가 이중화 되어있다고 해도) 같은 상태의 데이터를 조회할 수 있게 되죠. 게시글이 계속해서 올라온다면, 우리의 클라이언트에 따라 다른 피드가 조금 늦게 반영 될 수는 있겠으나, 더이상 피드가 올라오지 않는다면 우린 모두 같은 20개의 피드를 볼 수 있는거죠.

## 심화 : MSA 가 필요해

여기까지는 Spring 애플리케이션의 자체적인 기능을 통해 CQRS 와 이벤트 소싱을 구현 해 봤습니다. 하지만 트래픽이 너무나 감당할 수 없을 만큼 늘어나서 게시글 작성 서비스와 피드 조회 서비스를 MSA 로 분리하는 상황을 가정 해 볼게요.

지금까지 예시에서는 이런 경우에 L4 또는 L7 로드밸런서와 애플리케이션 스케일 아웃으로 처리할 수 있겠지만, 두 서비스가 하나의 모놀리식에 붙어있는 케이스라 두 서비스가 동시에 스케일 아웃 됩니다. 물론 이것도 훌륭한 방법이지만, 조회가 너무나 압도적으로 많은 경우엔 작성 서비스를 스케일 아웃 할 필요가 없을 수도 있죠. 이런 경우에는 MSA 를 고려해볼 만 합니다.

이번 세션에서는 간단한 MSA 를 구성했다고 가정한 예시를 한번 살펴 보겠습니다.

### 아키텍처 구성

간단하게 게시글 작성 서비스와 피드 조회 서비스를 MSA 로 나누고 Kafka 를 이벤트 브로커로 셋업 한 아키텍처를 그려 보겠습니다. 예시니까 너무 과하게 가진 말고 필요한 것들만 한번 넣어 볼게요.

```mermaid
flowchart TD
    Client["외부 클라이언트\n(웹/모바일)"]
    Web["Web Server\n(NGINX)"]

    subgraph Services
        direction TB
        PS["Post Service"]
        FS["Feed Service"]
    end

    subgraph Datastores
        MySQL[(MySQL)]
        Redis[(Redis)]
    end

    KafkaCluster["Kafka Cluster"]

    Client -->|HTTPS| Web
    Web -->|HTTP| PS
    Web -->|HTTP| FS

    PS -->|JDBC| MySQL
    FS -->|Redis Client| Redis

    PS -->|Kafka Producer| KafkaCluster
    KafkaCluster -->|Kafka Consumer| FS

```

정말 트래픽을 서비스 별로 조절하고 싶다면, Kubernetes 같은 걸 고려해볼 만 하지만, 이번 포스팅에서 다루기엔 얘기가 길어질 것 같으니 간단한 MSA 구조를 한번 그려 보았습니다.

### Kafka 설정

간단하게 Kafka 를 중간에 셋업 해 보겠습니다.

> ❓여기서 Kafka 란?
>
> Apache Kafka는 대용량 실시간 데이터 스트리밍을 처리하기 위한 분산 메시지 큐 시스템입니다.  
> 토픽(topic) 단위로 메시지를 분류하고, 파티션(partition) 으로 수평 분산 저장합니다.  
> 데이터는 Append-only 로그 형태로 디스크에 영구 보관 됩니다.  
> 프로듀서/컨슈머 모델로 느슨하게 결합된 생산·소비 구조를 가집니다.  
> 또한 브로커 클러스터링을 지원하고, 브로커 간 데이터 복제를 지원합니다.

Kafka 를 굳이 가져 온 이유는 이벤트 소싱에 자주 사용되는 시스템이기 때문입니다. 발생한 모든 이벤트를 순차적·불변 로그로 남기기에 적합합니다. 또한 파티셔닝, 복제를 통해 쓰기·읽기 처리량을 수평 확장할 수 있습니다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.kafka:spring-kafka'
}
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      group-id: feed-projection-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: com.example.feed
```

각 옵션들에 대한 설명은 아래와 같습니다.

- key-serializer, key-deserializer : 메시지의 키(key) 를 직렬화 / 역직렬화 할 클래스 지정
- value-serializer, value-deserializer :  메시지의 값(value) 을 직렬화 / 역직렬화 할 클래스 지정
- group-id : 컨슈머가 속할 컨슈머 그룹 ID
  - 같은 그룹에 속한 컨슈머끼리는 파티션을 나눠 처리(로드밸런싱)하고, 서로 다른 그룹끼리는 같은 메시지를 각각 읽습니다(브로드캐스트).
- properties.spring.json.trusted.packages : 역직렬화 대상 클래스를 찾을 때, 허용된 패키지를 지정합니다.
  - 기본값(*)으로 하면 보안 위험이 있으므로, 실제 애플리케이션 패키지(com.example.feed)만 신뢰하도록 제한합니다.

### 이벤트 발행 (Produce) 및 소비 (Consume)

```kotlin
@Service
class PostCommandService(
    private val postRepository: PostRepository,
    private val kafkaTemplate: KafkaTemplate<String, PostPublishedEvent>
) {
    private val TOPIC = "post-published"

    fun writePost(cmd: WritePostCommand) {
        val post = Post(cmd.authorId, cmd.title, cmd.content).apply { publish() }
        postRepository.save(post)

        // Kafka로 이벤트 발행
        kafkaTemplate.send(TOPIC, PostPublishedEvent(
            postId    = post.id,
            authorId  = post.authorId,
            createdAt = post.createdAt
        ))
    }
}
```

게시글 작성 서비스에서 KafkaTemplate<String, PostPublishedEvent>를 주입하고, JSON 직렬화를 통해 post-published 토픽에 이벤트를 보냅니다.

```kotlin
@Component
class PostFeedProjectionConsumer(
    private val postRepository: PostRepository,
    private val userService: UserService,
    private val postFeedViewRepository: PostFeedViewRepository
) {
    @KafkaListener(topics = ["post-published"], groupId = "feed-projection-group")
    fun onPostPublished(event: PostPublishedEvent) {
        // 1. 원본 Post 및 authorName 조회
        val post = postRepository.findById(event.postId).orElseThrow()
        val authorName = userService.getUserName(post.authorId)

        // 2. snippet 생성
        val snippet = post.content
            .take(100)
            .let { if (post.content.length > 100) "$it…" else it }

        // 3. 뷰 모델 구성
        val view = PostFeedView(
            postId     = post.id,
            authorName = authorName,
            title      = post.title,
            snippet    = snippet,
            createdAt  = event.createdAt.format(DateTimeFormatter.ISO_DATE_TIME)
        )

        // 4. Redis에 저장 (upsert)
        postFeedViewRepository.save(view)
    }
}
```

조회 서비스에서는 @KafkaListener로 post-published 토픽을 구독하여, 이벤트 수신 시 Redis에 PostFeedView를 저장 합니다.

# Outro

지금까지 CQRS 에 대해 가볍게(?) 알아 보았습니다. 단계적으로 CQRS 를 적용하는 방법과 사례 예시를 알아 보았습니다. 유익 하셨나요? 😊 막상 글을 따라 읽다보면, 생각보다 도입하기 어려운 개념이 아니라는 것을 알 수 있으실거에요.  

CQRS 를 도입하기 위해 대단한 시스템 아키텍처를 가지고 있을 필요도 없고 필수로 도입해야 하는 것은 아니겠지만, 서비스가 성장하다보면 필연적으로 늘어나는 조회 트래픽에 대해 고려를 해야 할 상황이 찾아옵니다.

서비스가 성장한다는 것은 팀을 구성하는 모든 사람들에게 좋은 조짐이지만, 한편으로는 개발 및 운영에서 큰 챌린지들을 겪게 됩니다. 앞으로 더 나아질 우리의 서비스를 유지보수할 수 있는 역량있는 개발자가 되기 위해서 CQRS 에 대한 개념은 꼭 정리해보는 게 좋을 것 같아서 글을 작성 해보았습니다.

재밌게 읽어 주셨다면 정말 감사합니다. 피드백은 얼마든지 환영합니다!
