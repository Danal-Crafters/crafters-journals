# 🍃 Intro

다시 돌아왔습니다. 여전히 JSON 을 상하차하며, 때로는 JS 를 상하차 중인 소프트웨어 잡부 손우진입니다. 이번 시리즈의 주제는 Auto Configuration 입니다.  
Spring 에서 Auto Configuration 기능은 정말 강력합니다. 애플리케이션 개발을 좀 더 편리하게 만들어주죠. 
이번 포스팅에서는 Auto Configuration 의 개념, 동작원리, 커스텀 Auto Configuration 을 작성하는 법에 대해 알아보겠습니다. 

> 시리즈가 오래 지속되다보니 슬 서론 쓰는 게 힘들어졌습니다 하하...

# ⚙️ Configuration?

Auto Configuration 에 대해 이야기 해 보기 전에 Configuration 그 자체에 대해 한번 생각해볼까 합니다.  
애플리케이션에는 여러가지 설정 값 들이 필요하죠. JDBC 커넥션은 그냥 맺어지지 않습니다. DB 의 주소, 그리고 계정 정보가 있어야죠. 

```kotlin
@Configuration
class CustomCacheConfiguration {

    @Bean
    fun cacheManager(): CacheManager {
        println("Custom CacheManager Configured")
        return ConcurrentMapCacheManager("custom")
    }
}

@RestController
@RequestMapping("/cache")
class CacheController(private val cacheManager: CacheManager) {

    @GetMapping("/get")
    fun getCache(): String {
        val cache = cacheManager.getCache("custom")
        cache?.put("key", "value")
        return cache?.get("key")?.get()?.toString() ?: "Cache not found"
    }
}
```

Spring 에서는 개발자가 @Bean 객체로 등록한 설정 값 들을 우리는 DI 를 통해 받아올 수 있습니다. 

이런 방법들은 간편하지만, 때로는 개발자들의 공수를 늘립니다. @Configuration 들을 통해 무언가 설정하기 시작하면, 관련 된 코드들이 늘어나고, 관리 공수가 늘어나죠. 

Spring Boot 는 이런 문제를 AutoConfiguration 을 통해 해결합니다. 

> 정말 Spring Boot 애플리케이션을 구성하고 있는 모든 설정값들을 저렇게 구성한다면 얼마나 많은 코드들이 생길까요...전부 우리 관리 영역에 들어온다고 생각 해 보면 많은 생각들이 듭니다...

# 🚀 Auto Configuration 개요

AutoConfiguration은 Spring Boot에서 제공하는 기능으로, Spring 컨텍스트가 초기화되는 동안 필요한 Bean들을 자동으로 구성하고 등록하는 역할을 합니다. 이를 통해 개발자는 복잡한 설정을 직접 작성하지 않고도 다양한 기능을 활용할 수 있습니다.

주로 `@EnableAutoConfiguration` 혹은 `@SpringBootApplication` 어노테이션을 사용함으로써 활성화됩니다. `@SpringBootApplication` 은 내부적으로 `@EnableAutoConfiguration` 을 포함하고 있습니다. 

```kotlin
@SpringBootApplication
class Application

fun main(args: Array<String>) {
    runApplication<Application>(*args)
}
```

# 🧐 Auto Configuration 의 동작 원리

## 아키텍처 구성 요소

Spring AutoConfiguration은 크게 다음과 같은 아키텍처 요소로 구성됩니다.

- **SpringFactoriesLoader** : 모든 AutoConfiguration 클래스를 검색하여 등록하는 핵심 구성 요소.
- **ApplicationContext** : AutoConfiguration 클래스를 빈으로 등록하는 컨테이너.
- **Condition Evaluation System** : @Conditional 어노테이션을 기반으로 AutoConfiguration 적용 여부를 결정.
- **Environment** : 프로퍼티 값 및 Profile 설정을 기반으로 빈 등록을 조정하는 데 사용됨.

## 동작 흐름

1. `ApplicationContext` 초기화 단계
    - Spring Boot 애플리케이션이 실행되면, SpringApplication.run() 메서드를 통해 `ApplicationContext`가 생성되고 초기화됩니다.

2. `SpringFactoriesLoader` 탐색
    - META-INF/spring.factories 파일을 읽어서 모든 `EnableAutoConfiguration` 클래스를 검색합니다.
    - 이 과정은 클래스패스 상에 존재하는 모든 JAR 파일을 탐색하여 spring.factories 파일을 찾는 방식으로 이루어집니다.

3. `AutoConfiguration` 클래스 로딩
    - spring.factories 파일에서 찾은 모든 AutoConfiguration 클래스를 로딩합니다.
    - 이 클래스들은 일반적으로 @Configuration으로 선언되어 있으며, @Conditional 어노테이션을 통해 활성화 조건을 명시합니다.

4. Condition Evaluation System 평가
    - 각 AutoConfiguration 클래스에 선언된 조건(@Conditional)을 평가합니다.
    - 예를 들어, `@ConditionalOnClass`, `@ConditionalOnBean`, `@ConditionalOnProperty` 등을 기반으로 클래스의 활성화 여부를 결정합니다.

5. Bean Definition 등록 및 `ApplicationContext` 구성
    - 조건을 만족하는 `AutoConfiguration` 클래스의 빈 정의(Bean Definition)를 `ApplicationContext`에 등록합니다.
    - 이후 `ApplicationContext`의 refresh() 메서드를 통해 빈들이 초기화되고, 의존성 주입이 이루어집니다.

## 아키텍처 다이어그램
```shell
+---------------------+
|  ApplicationContext |
|  (빈 컨테이너)       |
+----------+----------+
           |
           v
+-------------------------+
| SpringFactoriesLoader   |
| (spring.factories 탐색) |
+----------+--------------+
           |
           v
+---------------------+
| AutoConfiguration   |
| Class Loading       |
+----------+----------+
           |
           v
+---------------------+
| Condition Evaluation |
| System               |
+----------+----------+
           |
           v
+-----------------------+
| Bean Definition 등록  |
| ApplicationContext    |
+-----------------------+
```

## Spring Boot 는 왜 이 방식을 채택했을까요?

Spring Boot는 모든 **AutoConfiguration** 클래스를 한 번에 로드하지 않고, 조건을 평가하여 필요한 경우에만 등록합니다. 이를 통해 불필요한 빈 등록을 방지하고 메모리 사용을 줄이며, 초기화 시간을 단축할 수 있습니다. 특히 `@Conditional` 어노테이션을 활용하여 커스텀 조건을 정의할 수 있기 때문에 확장성이 좋습니다.  

# 😱 Auto Configuration 없이 Configuration 설정하기

AutoConfiguration을 사용하지 않는 경우, 모든 설정을 수동으로 작성해야 합니다. 예를 들어, 앞서 소개했던 것 처럼 @Bean 객체를 등록해서서 DataSource를 설정하는 경우를 비교해 보겠습니다.

```kotlin
@Configuration
class DataSourceConfig {

    @Bean
    fun dataSource(): DataSource {
        val config = HikariConfig()
        config.jdbcUrl = "jdbc:mysql://localhost:3306/mydb"
        config.username = "root"
        config.password = "password"
        config.driverClassName = "com.mysql.cj.jdbc.Driver"
        return HikariDataSource(config)
    }
}
```

위의 설정 클래스는 수동으로 DataSource를 구성하고 빈으로 등록하는 방식입니다. 모든 설정을 명시적으로 지정해야 하기 때문에 반복 작업이 많이 필요합니다.

하지만 Auto Configuration 을 사용한다면, `application.yml` 에 이렇게 설정할 수 있습니다. 

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
```

Spring Boot는 위 설정을 기반으로 자동으로 DataSource Bean을 등록합니다. HikariDataSource는 기본적으로 사용되며, 개발자가 별도로 등록하지 않아도 됩니다. 

# 🤩 Auto Configuration 실무 활용 1 : 컨픽 자동 설정

Redis 를 사용하는 애플리케이션에서 RedisTemplate 을 자동으로 설정하도록 해 보겠습니다. 

먼저 AutoConfiguration 없이 Redis Config 코드를 작성 해 보겠습니다. 

```kotlin
// RedisConfig.kt
@Configuration
class RedisConfig {

    @Bean
    fun redisConnectionFactory(): LettuceConnectionFactory {
        return LettuceConnectionFactory(RedisStandaloneConfiguration("localhost", 6379))
    }

    @Bean
    fun redisTemplate(redisConnectionFactory: RedisConnectionFactory): RedisTemplate<String, Any> {
        val template = RedisTemplate<String, Any>()
        template.setConnectionFactory(redisConnectionFactory)
        return template
    }
}
```
일반적으로는 이렇게 많이 작성 하실 것이라 생각이 들어요. 하지만 단점이 있죠. 매번 Redis 설정을 수동으로 작성해야 하고, 설정 변경 시 애플리케이션 전체에 영향을 미치기 어렵고 환경 별 설정(dev, prod) 를 관리하기 어렵습니다. 

물론 @ProfileActive 와 같은 어노테이션을 쓸 수 있겠지만, 여러개의 클래스를 생성해야 하는 상황이 생길 수 있습니다. 

그럼 Redis 설정을 자동으로 구성하는 AutoConfiguration 을 작성 해 보겠습니다.

```kotlin
// RedisAutoConfiguration.kt
@Configuration
@ConditionalOnClass(RedisTemplate::class)
@AutoConfiguration
@EnableConfigurationProperties(RedisProperties::class)
class RedisAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(RedisConnectionFactory::class)
    fun redisConnectionFactory(properties: RedisProperties): LettuceConnectionFactory {
        val config = RedisStandaloneConfiguration()
        config.hostName = properties.host ?: "localhost"
        config.port = properties.port ?: 6379
        return LettuceConnectionFactory(config)
    }

    @Bean
    @ConditionalOnMissingBean(RedisTemplate::class)
    fun redisTemplate(redisConnectionFactory: RedisConnectionFactory): RedisTemplate<String, Any> {
        val template = RedisTemplate<String, Any>()
        template.setConnectionFactory(redisConnectionFactory)
        return template
    }
}
```
앞서 원리를 설명 드렸듯이 `spring.factories` 파일을 통해 Spring Boot가 자동으로 `RedisAutoConfiguration`을 탐지합니다.  
`@ConditionalOnClass(RedisTemplate::class)` 어노테이션을 사용하여 Redis 라이브러리가 클래스패스에 존재할 때만 설정을 활성화합니다.  
`@ConditionalOnMissingBean` 어노테이션으로 사용자가 직접 설정을 정의하지 않은 경우에만 자동으로 구성합니다.
```yaml
# application.yml
spring:
  redis:
    host: redis.example.com
    port: 6379
```
```kotlin
// RedisProperties.kt
@ConfigurationProperties(prefix = "spring.redis")
data class RedisProperties(
    var host: String? = null,
    var port: Int? = null
)
```
`RedisProperties` 클래스는 `application.yml`의 설정 값을 주입받아 사용됩니다.
```kotlin
# resources/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.config.RedisAutoConfiguration
```
SpringBoot 는 resource/META-INF 내의 해당 파일을 읽어서 AutoConfiguration 클래스를 탐색합니다. 


이제 이 경우 application.yml 파일 별로 환경 설정이 가능해집니다. 그리고 설정의 일관성을 보장 해 주며, `@ConditionOnMissingBean` 으로 사용자가 직접 설정을 정의 할 경우 우선 적용되도록 할 수 있습니다. 

# 🤩 Auto Configuration 실무 활용 2 : 커스텀 Starter 만들기

아마 많이들 Spring Boot Starter 를 사용 중이실겁니다. Starter는 Spring Boot 프로젝트에서 특정 기능을 쉽게 사용할 수 있도록 미리 정의된 설정과 의존성을 제공하는 모듈입니다. 커스텀 Starter를 만들어 AutoConfiguration으로 자동 설정되도록 구성하는 예제를 하나 보여드리겠습니다. 

```shell
my-starter-parent/
│
├── my-starter/
│   ├── src/main/kotlin/com/example/mystarter/
│   │   ├── MyService.kt
│   │   ├── MyServiceAutoConfiguration.kt
│   │   └── MyStarterProperties.kt
│   ├── resources/META-INF/
│   │   └── spring.factories
│   ├── build.gradle.kts
│
├── my-starter-example/
│   ├── src/main/kotlin/com/example/demo/
│   │   └── DemoApplication.kt
│   ├── resources/application.yml
│   ├── build.gradle.kts
│
├── settings.gradle.kts
└── build.gradle.kts
```
MyStarter 프로젝트의 구성도입니다. my-starter 모듈 부터 살펴보겠습니다. 

```kotlin
plugins {
    kotlin("jvm") version "1.8.10"
    id("org.springframework.boot") version "3.1.8"
    id("io.spring.dependency-management") version "1.1.3"
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

dependencies {
    implementation("org.springframework.boot:spring-boot-autoconfigure")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}
```
my-starter 의 build.gradle 입니다. SpringBoot autoConfigure 패키지를 참조합니다. 

```kotlin
// MyService.kt
package com.example.mystarter

class MyService(private val message: String) {

    fun printMessage() {
        println("MyService says: $message")
    }
}
```
간단하게 메시지 하나 띄우는 로직을 가지는 클래스를 하나 생성 해 보겠습니다. 메시지는 클래스 생성자 파라미터로 받게 됩니다. 
```kotlin
// MyStarterProperties.kt
package com.example.mystarter

import org.springframework.boot.context.properties.ConfigurationProperties

@ConfigurationProperties(prefix = "mystarter")
data class MyStarterProperties(
    var message: String = "Hello from MyService!"
)
```
Property 클래스를 보겠습니다. @ConfigurationProperties 어노테이션을 통해 외부 설정파일 application.yml 등과 매핑할 수 있습니다. 설정 값의 접두사에 따라 해당 메시지가 변경될 수 있습니다. 

```kotlin
// MyServiceAutoConfiguration.kt
package com.example.mystarter

import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty
import org.springframework.boot.context.properties.EnableConfigurationProperties
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration

@Configuration
@EnableConfigurationProperties(MyStarterProperties::class)
@ConditionalOnProperty(prefix = "mystarter", name = ["enabled"], havingValue = "true", matchIfMissing = true)
class MyServiceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    fun myService(properties: MyStarterProperties): MyService {
        return MyService(properties.message)
    }
}
```
Configuration 클래스를 보겠습니다. MyService 클래스를 @Bean 으로 등록하는 역할을 합니다. @EnableConfigurationProperties 를 통해 @ConfigurationProperties 로 선언 된 클래스를 활성화 합니다. 여기서는 MyStarterProperties 클래스를 활성화하여, application.yml 또는 application.properties 파일의 설정을 자동으로 바인딩합니다.

@ConditionalOnProperty 를 통해 특정 Property 값에 따라 AutoConfiguration 을 활성화 혹은 비활성화 합니다. 여기서는 mystarter 의 enabled 에 따라 달라집니다. 

```kotlin
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.mystarter.MyServiceAutoConfiguration
```

마지막으로 resource/META-INF 디렉토리에 spring.factories 파일을 생성해서 AutoConfiguration 클래스를 등록합니다. 

그럼 Starter 를 사용 해 봐야겠죠? my-starter-example 모듈에서 Starter 를 불러와서 사용 해 보겠습니다. 

```kotlin
plugins {
    kotlin("jvm") version "1.8.10"
    id("org.springframework.boot") version "3.1.8"
    id("io.spring.dependency-management") version "1.1.3"
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

dependencies {
    implementation("org.springframework.boot:spring-boot-starter")
    implementation(project(":my-starter"))
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}
```
같은 모듈이니 project(":my-starter") 로 불러옵니다. 
```kotlin
// DemoApplication.kt
package com.example.demo

import com.example.mystarter.MyService
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.context.annotation.Bean

@SpringBootApplication
class DemoApplication(
    private val myService: MyService
) {

    @Bean
    fun run() = org.springframework.boot.CommandLineRunner {
        myService.printMessage()
    }
}

fun main(args: Array<String>) {
    runApplication<DemoApplication>(*args)
}
```
```yaml
# application.yml
mystarter:
  enabled: true
  message: "Hello from Custom Starter!"
```
application.yml 에는 설정 값들을 입력하고, main 클래스에서 해당 Service 를 DI 받아서 사용 합니다. 
앱을 실행시키면 아래와 같은 결과를 확인할 수 있습니다. 
```shell
MyService says: Hello from Custom Starter!
```
이제 우리는 따로 Bean 클래스를 선언하지 않고도 AutoConfiguration 을 통해 생성 된 Bean 클래스를 DI 받을 수 있습니다. 


# Outro

AutoConfiguration 에 대해서 딥다이브 까지는 아니더라도 발을 한번 담궈보았습니다. 최근 들어서 관리중인 애플리케이션들의 모듈들을 분리하고 라이브러리화 하려고 생각 중이라 겸사겸사 정리 해 보았습니다.  

AutoConfiguration 을 통해 우리만의 Starter 들을 만들어 보는 것 어떨까요? 이번 달도 유익하셨길 바랍니다.  

곧 또 뵙겠습니다. 😊

# Reference
https://docs.spring.io/spring-boot/reference/using/auto-configuration.html