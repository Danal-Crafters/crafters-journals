# 자바 개발자를 위한 Rust 동시성 살펴보기 (1)

 안녕하세요, Rust 동시성 프로그래밍을 찍먹하고 감탄한 BE플랫폼 개발팀 정찬미입니다.  

이 글은 자바의 동시성을 복습하고, Rust의 동시성을 받아들이기 위한 준비 과정입니다. 이번 편에서는 자바 동시성 리마인드 하고, 다음 편에서 Rust의 동시성과의 차이 및 Rust의 동시성 설계를 살펴볼 예정입니다.

## 1. 프로그램의 흐름과 스레드

프로그램은 강물이 바다를 향해 흐르듯 프로그래머가 정의한 방향으로 움직입니다. 자바에서는 이 흐름을 **스레드(Thread)** 라고 합니다. 하나의 흐름만 존재하는 프로그램을 싱글 스레드라 하고, 여러 독립된 흐름이 동시에 존재하는 프로그램을 멀티 스레드라 부릅니다.

예를 들어, 키보드로 입력받은 숫자를 출력하는 프로그램은 다음과 같은 단순한 흐름을 가집니다.  이와 같이 평소에 간단하게 작성하는 프로그램들은 대부분 싱글 스레드입니다. 즉, 프로그램의 흐름이 하나만 있습니다.

 프로그램의 독립적 흐름이 여러개 존재하면 이를 멀티 스레드라고 합니다. 멀티 스레드는 프로세스 내부에서 여러 개의 스레드가 존재할 수 있으며, 각각의 stack을 가지기 때문에 동시에 프로그램의 독립적 진행이 가능한 것을 의미합니다.

사실 동시에라는 단어를 사용했지만 실제로 '동시에' 돌아가는 것은 아니며, 아주 짧은 시간 단위로 실행되는 스레드가 달라지기_switching_ 때문에 사람이 보기에 동시에 실행되는 것처럼 보일 뿐입니다.

### Concurrency vs Parallelism 

특정 시간 지점에서는 독립된 하나의 스레드만 진행되지만, 짧은 시간 단위로 스레드 스위칭이 이루어지는 프로그램의 특성을 동시성_Concurrency_이라고 합니다. 반면, 실제 '물리적으로' 별개의 코어에서 테스크 각각이 진행되는 것을 병렬성_Parallelism_이라고 합니다. 그러므로 이번 포스팅은 동시성에 대한 이야기입니다.

![Concurrency vs Parallelism](https://images.velog.io/images/maigumi/post/ed9796a1-b649-44f0-aa6a-0afaf9063db8/concurrency.png)

### 메인 스레드  

메인 스레드는 어플리케이션 스레드라고도 불리며, 일반적인 싱글 스레드 프로그램은 메인 스레드 하나만 존재합니다. 자바 프로그램이 실행되면, 자바 플랫폼 내부적으로는 다음과 같은 일이 일어납니다.
  
>1. JVM이 프로그램을 실행할 준비를 한다.
> 2. 메인 함수가 있는 클래스가 클래스로딩이 완료되면 메인 스레드가 실행된다.
> 3. 메인 스레드의 자바 interpreter가 `메인함수가 있는 클래스::main()`의 바이트 코드를 읽는다.

즉 자바 플랫폼은 스레드별로 interpreter가 있으며, JVM은 메인 스레드를 최우선적으로 실행하도록 구성되어 있다는 사실을 알 수 있습니다. 

스레드 자세한 내용은 바로 아래에 언급하겠지만, 스레드는 일반적으로 `start()` 메소드를 호출해서 스레드를 시작시켜야 하는데 메인 함수는 그런 조작이 필요없는 이유이기도 합니다.

```java
Thread t = new Thread(() -> {System.out.println("Hello Thread");});
t.start();
```

### java.lang.Thread

자바는 다중 스레드 프로그래밍을 지원합니다. `java.lang` 패키지에 속해있으므로 import할 필요가 없다는 뜻입니다. 다중 스레드는 `Thread` 클래스를 이용해 구현할 수 있습니다. 또는 `run()`메소드를 가진 `Runnable` 인터페이스를 상속받아 구현합니다. `Thread` 클래스는 이미 `Runnalbe`의 자식 클래스입니다.

### 싱글 스레드로 충분하지 않은 상황  

싱글스레드로 키보드로 입력받은 숫자를 출력하는 프로그램입니다. 어렵지 않은 흐름입니다.

```java
public static void main(String[] args) throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    int a = Integer.parseInt(br.readLine());
    for(int i = 0; i < a; i++) {
        System.out.println(i);
    }
}
```

하지만, 초당 수천 건 이상의 API 요청을 동시에 처리해야 하는 상황이라면 단순한 싱글 스레드로는 성능상 한계에 직면하게 됩니다. 이러한 대용량 트래픽 환경에서는 멀티 스레드/비동기 처리하는 방식이 필요해집니다.   

물론 스프링 환경에서는 Thread Pool을 잘 관리해주고 더 고도화가 필요하다면 여러가지 대안이 있기 때문에 간단한 서버 프로그램을 작성할 때는 그리 신경쓰지 않긴 하죠.

## 2. 자바에서의 멀티 스레드 구현

스프링 포스팅은 아니니, 순수한 자바에서의 멀티 스레드를 이야기하겠습니다. 
스레드를 생성하는 방식은 다양합니다.

- **Runnable 인터페이스 활용:**
```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        performApiRequest();
    }
};
new Thread(runnable).start();
```

- **람다 표현식 활용:**  

```java
new Thread(() -> performApiRequest()).start();
```

- **Thread 클래스 상속:**  

```java
class ApiRequestThread extends Thread {
    @Override
    public void run() {
        performApiRequest();
    }
}
new ApiRequestThread().start();
```

위 코드에서 `performApiRequest()` 메서드는 concurrent하게 API 요청을 수행하며, 각각의 스레드는 독립적으로 실행될 수 있습니다.

### 3. 자바 스레드 라이프사이클과 상태 전환

스레드는 다음과 같은 생명 주기를 가집니다.

- **NEW**: 생성되었으나 실행되지 않은 상태
- **RUNNABLE**: 실행 준비가 완료된 상태
- **BLOCKED**: 동기화 자원(Lock)을 기다리는 상태
- **WAITING**: `wait()`, `join()` 등으로 다른 스레드를 기다리는 상태
- **TIMED_WAITING**: `sleep(ms)` 또는 시간 제한이 있는 대기 상태
- **TERMINATED**: 작업이 끝나 종료된 상태

이 상태를 명확히 이해하면 효율적인 동시성 관리가 가능합니다.

### 4. 자바 동시성 문제의 본질과 경합

멀티 스레드 프로그래밍에서 큰 이슈는  데이터 경합(race condition)입니다. 다음 예시를 살펴봅시다.

온라인 쇼핑몰에서 남은 상품 수량을 관리하는 코드입니다.

```java
public class Inventory {
    private int stock = 10;

    public void purchase() {
        if(stock > 0) {
            stock--;
        }
    }

    public int getStock() {
        return stock;
    }
}

Inventory inventory = new Inventory();
ExecutorService executor = Executors.newFixedThreadPool(5);

for (int i = 0; i < 20; i++) {
    executor.submit(inventory::purchase);
}

executor.shutdown();
executor.awaitTermination(1, TimeUnit.SECONDS);
System.out.println("남은 재고: " + inventory.getStock());
```

방어로직을 넣었으니, 기대하는 결과는 재고가 0보다 작아질 수 없지만, 데이터 경합으로 인해 실제로는 음수가 될 수도 있습니다.

### 5. 데이터 경합 해결법: 동기화 / 원자 연산

- **Synchronized 키워드 사용:**

```java
public synchronized void purchase() {
    if(stock > 0) {
        stock--;
    }
}
```

- **AtomicInteger 사용:**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicInventory {
    private final AtomicInteger stock = new AtomicInteger(10);

    public void purchase() {
        stock.updateAndGet(current -> current > 0 ? current - 1 : current);
    }

    public int getStock() {
        return stock.get();
    }
}
``` 

### 6. 교착상태(Deadlock)

데이터 경합 문제를 해결하려고 동기화를 과도하게 사용하면 이번에는 교착상태(deadlock) 라는 새로운 문제가 발생할 수 있습니다. 여러 스레드가 서로의 자원을 기다리며 아무 작업도 진행하지 못하는 상태로, 이는 전체 프로그램의 사고로 이어집니다.

이 문제를 피하려면 교착상태 발생의 네 가지 조건(상호 배제, 보유 및 대기, 비선점, 환형 대기) 중 하나라도 해소해야 합니다.
그래서 설계 단계에서부터 리소스 획득 순서에 주의하거나 타임아웃(timeout)을 활용하는 전략이 필요합니다.

### 7. 자바 8 이후의 대안

#### 동시성 vs 비동기성  

지금까지 언급한 동시성 프로그래밍은 여러 개의 작업을 동시에 처리하여 전체적으로 성능을 높이는 아이디어 입니다.

반면, 비동기 프로그래밍은 단일 작업을 처리할 때 작업의 완료를 기다리지 않고 다른 작업을 먼저 수행함으로써, 개별 작업 단위에서의 효율을 높입니다. 즉, 동시성은 다수의 작업을 병렬적으로 실행하는 거고, 비동기는 특정 작업이 완료될 때까지 블로킹되지 않고 효율적으로 작업을 관리합니다.

#### CompletableFuture의 장단점과 현실적인 쓰임새

자바 8에서 등장한 CompletableFuture는 이런 비동기 처리 방식을 지원하며, 작업의 연결과 조합을 쉽게 만들어주는 기능을 제공합니다.  

- 콜백(Callback) 지원: 작업 완료 후 자동 호출되는 콜백 메서드를 지원하여 비동기 작업의 결과 처리 방식을 크게 개선했습니다.
- 작업의 손쉬운 조합과 연결: 여러 비동기 작업을 직관적으로 연결하고 병렬 처리한 후 그 결과를 쉽게 조합할 수 있습니다.
- 명확한 예외 처리: 작업 실패 시 후속 조치를 체계적으로 구현할 수 있어 개발자의 선택지와 안정성이 늘어났습니다.

```java
CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> fetchUser(userId));
CompletableFuture<String> orderFuture = CompletableFuture.supplyAsync(() -> fetchOrderDetails(userId));

CompletableFuture<Void> combinedFuture = userFuture.thenCombine(orderFuture, (user, order) -> {
    return "User: " + user + ", Order: " + order;
}).thenAccept(result -> {
    System.out.println(result);
}).exceptionally(ex -> {
    System.out.println("Error: " + ex.getMessage());
    return null;
});
```

하지만, CompletableFuture는 몇 가지 단점이 있습니다.

- 복잡한 API: 메서드 체이닝으로 인해 코드가 복잡해지고 읽기 어려워집니다.
- 디버깅의 어려움: 스택 트레이스가 명확하지 않아 문제 파악이 어렵습니다.

### 8. 현재 자바 진영에서 많이 사용되는 비동기 처리

그래서 실무에서는 보다 직관적이고 효율적인 비동기 처리를 위해 다음과 같은 대안을 사용합니다.

- 리액티브 스트림 API (Reactor, RxJava): 복잡한 비동기 흐름을 간결하고 명확하게 다룰 수 있어 Spring WebFlux 등에서  활용됩니다.

- Spring WebFlux: 비동기적이고 논블로킹 방식의 웹 서버 개발을 지원합니다. 성능과 확장성이 이점입니다.

- 코루틴(Kotlin): 직관적인 비동기 프로그래밍을 지원해서 인기가 많습니다.

### 8. Rust의 동시성은?

Rust는 메모리 안전성과 동시성 안전성을 언어 차원에서 지원하며, 데이터 경합을 런타임이 아닌, 컴파일 타임에 원천적으로 방지하는 방식을 제공합니다!  

다음 편에서는 Rust의 동시성이 자바와 어떤 차이가 있는지, 그리고 Rust만의 소유권(ownership) 모델과 메시지 전달 방식(message passing) 같은 핵심 개념을 살펴보겠습니다.
