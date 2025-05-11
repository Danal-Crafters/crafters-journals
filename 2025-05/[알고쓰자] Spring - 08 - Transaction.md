# 🍃 Intro

다시 돌아왔습니다. Spring 으로 열심히 JSON 을 상하차 중인 손우진입니다.  
백엔드 개발자에게 Transaction 이라는 키워드는 죽어도 땔 수 없습니다. 데이터를 DB에서 적절하게 가공해서 가져오거나 저장하는 모든 일련의 과정은 하나의 작업 단위로 나뉩니다. 우리는 이렇게 나눌 수 없는 하나의 작업 단위를 `Transaction` 이라고 부릅니다.

Spring 에서는 Transaction 을 어떻게 관리할까요? 아무것도 설정하지 않은 경우엔 `Auto Commit` 모드가 활성화 됩니다. 날아가는 쿼리 족족 자동으로 커밋이 되어버리죠. 사실 그냥 **Read** 만 하는 경우라면 크게 문제가 없을 수도 있습니다만 **Create**, **Update**, **Delete** 에서는 문제가 될 수 있습니다. 만약 문제가 발생한 경우 롤백하기가 상당히 곤란해질 수 있겠죠. 사람이 일일히 손으로 롤백해주는 경우가 일어나길 바라는 사람은 많지 않을 것이라 생각합니다.

이번 포스팅에서는 이런 문제를 해결해줄 수 있는 `@Transactional` 어노테이션에 대해 다뤄 보겠습니다.

# ⚙️ Transaction 이 가지는 주요 속성

전공에서 혹은 정보처리기사 시험 등에서 데이터베이스 이론에 대해 배울 때 반드시 공부하게 되는 것이 Transaction 의 주요 속성 **ACID** 입니다. 다들 잘 아시겠지만 리마인드 삼아 한번 짚고 넘어가겠습니다.

- **원자성 (Atomicity)** : 모든 작업이 성공하거나 모두 실패해야 함.
- **일관성 (Consistency)** : 트랜잭션이 완료되면 데이터가 일관된 상태여야 함.
- **고립성 (Isolation)** : 트랜잭션 간의 작업이 서로 간섭하지 않음.
- **지속성 (Durability)** : 트랜잭션이 완료되면 그 결과는 영구적으로 반영됨.

# 🧐 @Transactional 동작 원리

지난 포스팅에서 Spring 의 AOP 에 대해 자세히 다룬 적이 있습니다. Spring 의 `@Transactional` 어노테이션은 프록시 기반 AOP 를 사용해서 메소드 실행 전후에 Transaction 경계를 설정할 수 있습니다.

아마도...지난 포스팅을 다시 자세히 보시기엔 시간이 빡빡하거나 할 수 있으니 간단하게만 리마인드 하자면, Spring IOC 가 관리하는 Beans 들에 대해 Proxy 가 적용됩니다. Proxy 가 원본 컴포넌트를 감싸고 있는 구조입니다. `@Transactional` 어노테이션도 마찬가지의 원리로 구현 됩니다.

```kotlin
val proxyFactory = ProxyFactory()
proxyFactory.addInterface(TestService::class)
proxyFactory.addAdvice(advice)
proxyFactory.addAdvisor(advisor)
```

@Transactional 어노테이션을 사용하게 된다면 해당 메소드 시작 시 Transaction 을 시작하고, 성공 시 커밋, 실패 시 롤백합니다. 기본적으로 Runtime Exception 이 발생한다면 롤백하게 됩니다.

```
@Transactional
public void processOrder(Order order) {
    orderRepository.save(order);
    paymentService.processPayment(order);
}
```

간단하죠? 이 어노테이션을 달아 주지 않는다면 앞서 말씀 드렸다시피 `Auto Commit` 모드로 처리 됩니다. 이 기능이 굳이 필요 없다면 사용하지 않아도 무관합니다.

> 💡 좋은 기능인 것 같은데...DB 의 Transaction 과 정확히 무슨 차이인가요?

Spring의 @Transactional은 Physical Transaction 과 Logical Transaction 두 가지 관점에서 이해할 필요가 있습니다.

| 개념          | 설명                                                                       |
| ----------- | ------------------------------------------------------------------------ |
| **Physical Transaction** | DB Connection 단위로 묶이는 실제 DB와의 커넥션 기반 Transaction.|
| **Logical Transaction** | Spring 애플리케이션 레벨에서 관리되는 Transaction 의 논리적 단위. 여러 Logical Transaction 이 하나의 Physical Transaction 에 매핑될 수 있음. |

우리가 흔히 DB 에서 생각하는 것이 Physical Transaction, 애플리케이션 레벨에서 관리되는 영역이 Logical Transaction 이죠.  
Physical Transaction 은 JDBC 또는 JPA(Hibernate), MyBatis 와 같은 ORM Framework 와 연결 된 데이터베이스 레벨에서 실행됩니다.

Spring 은 DataSource 를 통해 Physical Transaction 을 생성합니다.

```kotlin
val connection = dataSource.connection
connection.autoCommit = false // Transactional 어노테이션을 사용한 경우 Auto-commit 비활성화 
```

정상적으로 메소드가 완료되면 commit, 예외가 발생한 경우 rollback 이 호출됩니다.

```kotlin
try {
    // SQL 실행
    connection.commit()
} catch (e: Exception) {
    connection.rollback()
    throw e
} finally {
    connection.close()
}
```

Logical Transaction 은 Spring Transaction 관리자가 관리합니다. 여러 서비스 메소드가 하나의 Physical Transaction 에 포함될 수 있죠.  
조금 뒤에서 자세히 설명 하겠지만, 애플리케이션 레벨에서 이 Transaction 이 어느 메소드 까지 전파되어야 할 지를 개발자가 설정해줄 수 있죠. 이를 전파 레벨 (Propagation Level) 이라고 합니다.

Spring 은 이런 Logical Transaction 을 관리하고 필요한 경우 Physical Transaction 에 반영합니다. 앞서 말씀 드린 전파레벨에 따라 하나의 Physical Transaction 에는 여러개의 Logical Transaction 이 존재할 수 있죠.

# ⚙️ @Transactional 어노테이션의 설정 값

@Transactional 어노테이션은 네 가지의 설정 파라미터들을 제공합니다.

- Propagation : 트랜잭션이 어떻게 전파될지를 결정 (예: REQUIRED, REQUIRES_NEW)
- Isolation : 트랜잭션이 다른 트랜잭션과 어떻게 격리될지 설정 (예: READ_COMMITTED)
- RollbackFor : 특정 예외에 대해 롤백 여부를 설정
- ReadOnly : 트랜잭션을 읽기 전용으로 설정하여 성능 최적화

주요 속성들에 대해 한번 짚고 넘어가겠습니다.

## 📢 Propagation

앞서 말씀드린 Logical Transaction 을 설정하는 옵션입니다. 메소드의 흐름에 따라 개발자가 지정해줄 수 있습니다.

| Propagation Level | 설명                                                     |
| ----------------- | ------------------------------------------------------ |
| `REQUIRED`        | 이미 트랜잭션이 있으면 참여하고, 없으면 새로 생성 (Default).                |
| `REQUIRES_NEW`    | 무조건 새로운 트랜잭션을 생성. 기존 트랜잭션은 일시 중단.                      |
| `NESTED`          | 부모 트랜잭션의 일부로 동작하되, 개별 커밋/롤백이 가능. (Savepoint를 통해 롤백 가능) |
| `SUPPORTS`        | 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 실행.                        |
| `NOT_SUPPORTED`   | 트랜잭션이 있으면 일시 중단하고 비트랜잭션으로 실행.                          |
| `NEVER`           | 트랜잭션이 있으면 예외 발생.                                       |

예를 하나 들어 보겠습니다. 두 개의 서비스 메소드가 만약 `REQUIRED` (기본값) 로 선언 된다면 동일한 Physical Transaction 을 공유합니다.

```kotlin
@Transactional
fun methodA() {
    // 물리 트랜잭션 시작
    repository.save(entityA) 
    methodB()
    // 커밋 또는 롤백
}

@Transactional(propagation = Propagation.REQUIRED)
fun methodB() {
    // 같은 물리 트랜잭션 사용
    repository.save(entityB)
}
```

하지만 `REQUIRED_NEW` 로 선언 된다면 어떻게 될까요? 무조건 매 메소드 마다 새로운 Physical Transaction 으로 독립됩니다.  

```kotlin
@Transactional
fun parentMethod() {
    orderService.placeOrder() // 기존 트랜잭션
    paymentService.processPayment() // 새 트랜잭션 시작
}

@Transactional(propagation = Propagation.REQUIRED_NEW)
class PaymentService {
    fun processPayment() {
        // Do Something
    }
}
```

이 경우 각각 메소드에서 예외가 발생해도 서로 영향을 주지 않습니다.  
`processPayment` 에 예외가 발생해도 `placeOrder` 의 Transaction 은 롤백되지 않습니다.

조금 복잡한 예시를 하나 더 보겠습니다. `NESTED` 로 선언 된다면 Savepoint 를 생성해서 부분 롤백이 가능합니다.

```kotlin
@Transactional
fun parentMethod() {
    try {
        childMethod()
    } catch (e: Exception) {
        // childMethod 롤백, parentMethod는 유지
    }
}

@Transactional(propagation = Propagation.NESTED)
fun childMethod() {
    // Savepoint 생성됨
    repository.save(entity)
    throw RuntimeException("Rollback only child")
}
```

만약 `childMethod` 에서 예외가 발생한다면 Savepoint 까지 롤백 됩니다. `parentMethod` 는 영향을 받지 않고 계속 진행하죠.

## 🔒 Isolation

Isolation level 은 여러 Transaction 이 동시에 처리될 때 서로 간섭하지 않고 독립성을 유지하는 수준을 의미합니다.
SQL 표준에서는 다음과 같은 네 가지 격리 수준을 제공합니다.

| Isolation Level    | 설명   | 문제 발생 가능성   |
| ------------------ | ----------------------------------------- | --------------------------------------------- |
| `READ_UNCOMMITTED` | 커밋되지 않은 변경 내용도 다른 Transaction 이 읽을 수 있음   | Dirty Read, Non-repeatable Read, Phantom Read |
| `READ_COMMITTED`   | 커밋된 데이터만 읽을 수 있음. SQL Server의 기본값   | Non-repeatable Read, Phantom Read |
| `REPEATABLE_READ`  | 같은 Transaction 내에서는 동일한 데이터를 반복해서 읽을 때 값이 변하지 않음 | Phantom Read |
| `SERIALIZABLE`     | Transaction 을 순차적으로 실행하는 것처럼 격리. 가장 높은 격리 수준      | 없음 (성능 저하가 큼) |

여기서 정의 된 문제들에 대해 자세히 알아볼게요.  

- Dirty Read
  - 다른 Transaction 커밋하지 않은 데이터를 읽을 수 있습니다.
  - Transaction이 롤백되면 읽은 데이터는 무효한 데이터가 됩니다.

- Non-repeatable Read
  - 같은 Transaction 내에서 두 번 조회 시 값이 다르게 보입니다.
  - 다른 Transaction 이 커밋하면 변경된 값이 반영됩니다.

- Phantom Read
  - 같은 Transaction 내에서 범위 쿼리의 결과가 다르게 보입니다.
  - 다른 Transaction 이 커밋하면서 새로운 데이터가 추가되거나 삭제됩니다.

@Transactional 어노테이션에서 이 속성을 조정해줄 수 있습니다. 자세히 알아볼까요?
앞서 설명드린 네 가지 옵션 외에 Default 옵션이 존재합니다.

- `DEFAULT` : DBMS의 기본 Isolation Level을 따름. 보통 **READ\_COMMITTED** 또는 **REPEATABLE\_READ**
- `READ_UNCOMMITTED` : 커밋되지 않은 데이터를 읽을 수 있음. Dirty Read가 발생할 수 있음
- `READ_COMMITTED` : 커밋된 데이터만 읽을 수 있음. SQL Server의 기본값
- `REPEATABLE_READ` : Transaction 내에서 동일 데이터를 반복해서 읽으면 항상 같은 값이 보임  
- `SERIALIZABLE` : Transaction 이 순차적으로 실행되는 것처럼 동작. 동시성 성능이 떨어짐

### DEFAULT

별도로 설정하지 않으면 Default 속성으로 들어갑니다.

```kotlin
@Transactional(isolation = Isolation.DEFAULT)
fun defaultIsolationExample() {
    val account = accountRepository.findById(1L).get()
    account.balance -= 100
    accountRepository.save(account)
}
```

별도로 설정하지 않으면 DB의 기본 격리 수준이 적용됩니다.  
MySQL에서는 REPEATABLE_READ, PostgreSQL에서는 READ_COMMITTED로 실행됩니다.  
동시성 제어는 DBMS 설정에 따라 달라집니다.

### READ_UNCOMMITED

READ_UNCOMMITED 는 앞서 설명과 같이 커밋되지 않은 데이터를 다른 Transaction 이 읽을 수 있습니다. Dirty Read 가 발생할 수 있죠.

```kotlin
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
fun readUncommittedExample() {
    val balance = accountRepository.findBalanceById(1L)
    println("Balance: $balance")
}
```

만약 Transaction A 에서 값을 변경했다고 가정 해 보겠습니다. 커밋을 하지 않은 상태입니다.

```sql
-- Transaction A
BEGIN;
UPDATE accounts SET balance = 1000 WHERE id = 1;
```

Transaction B 에서 읽기 쿼리가 날아갔습니다.

```sql
-- Transaction B
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT balance FROM accounts WHERE id = 1; 
-- 💡 1000이 보임 (Dirty Read)
```

자 뭔가 잘못되었네요. Transaction A 에서 롤백처리를 했습니다.

```sql
-- Transaction A
ROLLBACK;
```

이 경우 Transaction B 는 커밋되지 않은 1000을 읽었죠? 이 Transaction 을 사용한 로직은 Dirty Read 로 인해 데이터 일관성이 깨질 수 있습니다.

### READ_COMMITTED

READ_COMMITTED 는 커밋 된 값만 읽을 수 있습니다. Non repeatable Read 가 발생할 수 있고, PostgreSQL, Oracle 의 기본 값입니다.

```kotlin
@Transactional(isolation = Isolation.READ_COMMITTED)
fun readCommittedExample() {
    val balance = accountRepository.findBalanceById(1L)
    println("Balance: $balance")
}
```

마찬가지로 예시를 볼게요. Transaction A 에서 값을 조회했습니다.

```sql
-- Transaction A
BEGIN;
SELECT balance FROM accounts WHERE id = 1; -- 💡 500
```

그 후 Transaction B 에서 값 수정 후 커밋이 일어났습니다.

```sql
-- Transaction B
BEGIN;
UPDATE accounts SET balance = 700 WHERE id = 1;
COMMIT;
```

Transaction A 에서 값을 다시 조회하니 값이 변경 되었네요? Non repeatable Read 가 발생했습니다.

```sql
-- Transaction A
SELECT balance FROM accounts WHERE id = 1; -- 💡 700 (값이 변경됨, Non-repeatable Read)
COMMIT;
```

### REPEATABLE_READ

같은 Transaction 내에서는 반복 조회 시 동일한 값이 보입니다. MySQL 의 기본 격리 수준이죠. 이 경우 Phantom Read 가 발생할 수 있습니다.

```kotlin
@Transactional(isolation = Isolation.REPEATABLE_READ)
fun repeatableReadExample() {
    val balance1 = accountRepository.findBalanceById(1L)
    println("First Read: $balance1")

    // 다른 트랜잭션이 값을 변경해도 보이지 않음
    val balance2 = accountRepository.findBalanceById(1L)
    println("Second Read: $balance2")
}
```

Transaction A 에서 값을 조회했습니다.

```sql
-- Transaction A
BEGIN;
SELECT balance FROM accounts WHERE id = 1; -- 💡 500
```

Transaction B 에서 값을 변경했습니다.

```sql
-- Transaction B
BEGIN;
UPDATE accounts SET balance = 800 WHERE id = 1;
COMMIT;
```

Transaction A 에서 다시 값을 조회하니 값이 그대로 입니다.

```sql
-- Transaction A
SELECT balance FROM accounts WHERE id = 1; -- 💡 500 (변경된 값이 보이지 않음)
COMMIT;
```

### SERIALIZABLE

마지막으로 Serializable 입니다. 모든 Transaction 이 순차적으로 실행되는 것 처럼 동작하죠. 이 경우 Phantom Read 가 발생하지는 않지만, 가장 성능이 낮죠.

```kotlin
@Transactional(isolation = Isolation.SERIALIZABLE)
fun serializableExample() {
    val accounts = accountRepository.findAll()
    println("Accounts: $accounts")
}
```

Transaction A 에서 값을 조회했습니다.

```sql
-- Transaction A
BEGIN;
SELECT * FROM accounts WHERE balance > 500;
```

Transaction B 가 값 변경을 시도 했습니다. 이 경우 Block 이 걸립니다.

```sql
-- Transaction B
BEGIN;
INSERT INTO accounts (id, balance) VALUES (10, 600); -- 🚫 Blocked!
```

Transaction A 가 커밋되기 전 까지 Transaction B 가 대기하게 됩니다.

### 정리

| Isolation Level    | Dirty Read | Non-repeatable Read | Phantom Read | 성능        |
| ------------------ | ---------- | ------------------- | ------------ | --------- |
| `DEFAULT`          | DB 설정에 따름  | DB 설정에 따름           | DB 설정에 따름    | DB 설정에 따름 |
| `READ_UNCOMMITTED` | ✅          | ✅                   | ✅            | 👍👍👍👍  |
| `READ_COMMITTED`   | ❌          | ✅                   | ✅            | 👍👍👍    |
| `REPEATABLE_READ`  | ❌          | ❌                   | ✅            | 👍👍      |
| `SERIALIZABLE`     | ❌          | ❌                   | ❌            | 👍        |

MySQL, PostgreSQL, Oracle, SQL Server 등 각각의 DBMS 에 따라 기본 값 Isolation Level 은 다릅니다. 로직의 성격에 따라 다르겠지만, 개인적으론 유저 한명에게 1대1로 할당 되는 데이터라면 `REPEATABLE_READ` 를 써도 괜찮지 않나 라는 생각을 합니다.  
배치 로직 등을 작성할 때는 Isolation Level 에 따른 에러 발생을 막기 위해 더이상 Update 되지 않을 법 한 데이터를 최대한 읽는 전략을 사용하곤 합니다.

## ⏪ RollbackFor

@Transactional 은 기본적으로 아래와 같은 규칙을 따릅니다.

| Exception 유형            | 기본 동작 | 설명     |
| ----------------------- | ----- | -------------------------------------------------------- |
| **Unchecked Exception** | 롤백    | `RuntimeException`, `IllegalArgumentException` 등 발생 시 롤백 |
| **Checked Exception**   | 커밋    | `Exception`, `IOException` 등 발생 시 커밋 진행   |
| **Error**               | 롤백 안됨 | 시스템 에러 (`OutOfMemoryError`)는 롤백되지 않음    |

> 💡여기서 잠깐! Unchecked / Checked Exception 의 차이가 뭔가요?  
> 컴파일러가 예외 처리 여부를 검사하는 지 여부에 따라 나뉩니다.
> Unchecked Exceptoin 은 대표적으로 `NullPointerException`, `ArrayIndexOutOfBoundsException`, `ArthmeticException`이 있고, Checked Exception 은 `IOException`, `SQLException`, `ClassNotFoundException` 이 있습니다.  
> 소프트웨어의 안정성에 따라 나뉩니다. Unchecked Exception 은 보통은 프로그래머의 실수에 의해 많이 발생하죠.

RollbackFor 옵션은 명시적으로 특정한 예외가 발생하는 경우 롤백할 수 있도록 설정합니다.

```kotlin
import org.springframework.stereotype.Service
import org.springframework.transaction.annotation.Transactional
import java.io.IOException
import java.sql.SQLException

@Service
class PaymentService(private val paymentRepository: PaymentRepository) {

    @Transactional(rollbackFor = [IOException::class, SQLException::class])
    fun processMultipleExceptions(payment: Payment) {
        paymentRepository.save(payment)
        
        if (payment.amount > 1000) {
            throw IOException("Amount exceeded limit")
        } else {
            throw SQLException("Database error occurred")
        }
    }
}
```

rollbackFor 옵션을 통해 Checked Exception 또한 롤백 처리할 수 있습니다. 좀 더 응용해서 여러 예외에 대해 롤백 처리를 할 수도 있죠.

반대로 noRollbackFor 옵션도 있습니다. 특정 예외가 발생하더라도 롤백하지 않도록 설정할 수 있죠.

```kotlin
import org.springframework.stereotype.Service
import org.springframework.transaction.annotation.Transactional
import java.io.IOException

@Service
class PaymentService(private val paymentRepository: PaymentRepository) {

    @Transactional(noRollbackFor = [IOException::class])
    fun processWithNoRollback(payment: Payment) {
        paymentRepository.save(payment)
        throw IOException("Checked Exception 발생") // 💡 롤백되지 않음
    }
}
```

## 📖 ReadOnly

@Transactional에서 제공하는 readOnly 옵션은 해당 트랜잭션이 오직 읽기 전용 작업만 수행할 것을 명시합니다.

SQL SELECT 문과 같은 읽기 작업만 수행할 때 최적화할 수 있습니다. 그리고 JPA, Hibernate와 같은 ORM에서 영속성 컨텍스트를 Dirty Checking (변경 감지) 하지 않습니다.그래서 데이터 수정이 발생할 경우 예외를 발생시킬 수 있습니다.

```kotlin
@Transactional(readOnly = true)
fun findUserById(id: Long): User {
    return userRepository.findById(id).orElseThrow()
}
```

이 경우 JPA 는 엔티티 변경을 감지하지 않습니다. 그러므로 Flush 작업을 하지 않아요.

만약 ReadOnly 가 활성화되면 Hibernate 세션이 `FlushMode.MANUAL` 로 설정됩니다. 기본값은 `FlushMode.AUTO` 죠. 앞서 말씀드렸다시피 Dirty Checking 이 이뤄지지 않고 DB 에 UPDATE, INSERT 가 발생하지 않습니다.

```kotlin
import org.springframework.stereotype.Service
import org.springframework.transaction.annotation.Transactional

@Service
class UserService(private val userRepository: UserRepository) {

    @Transactional(readOnly = true)
    fun updateUserEmail(userId: Long, newEmail: String) {
        val user = userRepository.findById(userId).orElseThrow()
        user.email = newEmail
        userRepository.save(user) // 💡 예외 발생 가능성
    }
}
```

한번 readOnly 옵션을 걸어놓고 save 를 시도 해 볼까요?

```shell
org.springframework.transaction.TransactionSystemException:
Could not commit JPA transaction; nested exception is javax.persistence.TransactionRequiredException
```

당연히 DB에 반영되지 않습니다. Hibernate 가 flush 하지 않기 때문이죠. 강제로 변경을 감지하려고 하면 예외가 발생할 수 있습니다.

이 옵션을 좀 더 응용하면, Spring Data JPA 에서 Repository 메소드에 개별적으로 지정해줄 수 있습니다.

```kotlin
import org.springframework.data.jpa.repository.Query
import org.springframework.data.repository.CrudRepository
import org.springframework.transaction.annotation.Transactional

interface UserRepository : CrudRepository<User, Long> {

    @Transactional(readOnly = true)
    @Query("SELECT u FROM User u WHERE u.email = :email")
    fun findByEmail(email: String): User?
}
```

@Query와 @Transactional(readOnly = true)를 결합하면 읽기 전용 트랜잭션이 적용됩니다. 이 경우 플러시가 발생하지 않습니다.

해당 옵션을 사용하면 성능 최적화에 이점을 가질 수 있습니다. 이유는 아래와 같습니다.

- Dirty Checking을 생략
  - Hibernate가 영속성 컨텍스트의 엔티티 변경 여부를 감지하지 않습니다.
  - 메모리 사용량이 줄어들고 처리 속도가 빨라집니다.

- Flush를 생략
  - FlushMode.MANUAL로 설정되어 DB에 업데이트 시도하지 않습니다.
  - 네트워크 오버헤드가 감소합니다.

- MySQL에서 최적화
  - MySQL 8.0 이상에서는 읽기 전용 트랜잭션에 대한 최적화가 자동으로 적용됩니다.

# 😭 @Transaction 이 없으면 발생할 수 있는 문제

정말 간단한 Read 의 경우는 불 필요 할 수도 있습니다. 물론 필요한 경우도 존재합니다. 만약 우리가 ReadOnly Database 를 분리했다면 필요할 수 있죠. (자세한 사용법은 응용 파트에서 서술합니다.)

하지만 `Create`, `Update`, `Delete` 의 경우에는 사용하는 것을 적극 권장합니다. 사유는 아래 때문입니다.

- Partial Update
  - 여러 SQL이 실행 중 하나가 실패하면 이전에 실행된 SQL은 롤백되지 않습니다.
  - 예를 들어, A, B 두 테이블이 업데이트되는데, B에서 예외 발생 시 A의 변경 사항은 반영된 상태로 남습니다.

- Dirty Read, Non-repeatable Read, Phantom Read
  - Isolation Level 설정이 불가능하므로, 읽기 중간에 다른 트랜잭션이 변경한 값을 읽을 수 있습니다.
  - 반복 읽기 시 값이 달라지거나, 커밋되지 않은 데이터를 읽을 수 있습니다.
  
- Rollback 불가
  - @Transactional이 없으면 Checked Exception 발생 시 롤백이 자동으로 되지 않습니다.
  - 직접 rollback()을 호출하지 않으면 DB에는 변경 사항이 남습니다.

# 💡 응용 : Spring @Transactional 테스트 - Physical Transaction 분리 검증

간단한 예시를 하나 보겠습니다. 실제 로직 테스트 코드를 작성할 때 Transaction 이 정상적으로 롤백되는 지 확인하는 예시들을 작성 해 보겠습니다.

```kotlin
import io.kotest.core.spec.style.DescribeSpec
import io.kotest.matchers.shouldBe
import io.kotest.assertions.throwables.shouldThrow
import io.mockk.*
import jakarta.transaction.Transactional
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.boot.test.mock.mockito.MockBean

@SpringBootTest
@Transactional
class OrderServiceTest @Autowired constructor(
    private val orderRepository: OrderRepository
) : DescribeSpec({

    // OrderService를 Spy로 생성 (실제 메서드는 실행하되 일부는 Mock)
    val orderService = spyk(OrderService(orderRepository))

    describe("OrderService - 트랜잭션 롤백 테스트") {

        context("Order 생성 시 예외가 발생하면") {

            val order = Order(description = "Test Order")
            val initialCount = orderRepository.count()

            // Mocking: save() 호출 시 예외 발생하도록 설정
            every { orderRepository.save(any()) } throws RuntimeException("DB Error")

            it("트랜잭션이 롤백되어 개수가 증가하지 않는다") {
                shouldThrow<RuntimeException> {
                    orderService.createOrder(order)
                }

                val finalCount = orderRepository.count()
                finalCount shouldBe initialCount // 롤백되었으므로 동일해야 함
            }

            it("save() 메서드는 한 번만 호출된다") {
                verify(exactly = 1) { orderRepository.save(any()) }
            }

            afterEach {
                // MockK 상태 초기화
                clearMocks(orderRepository)
            }
        }
    }
})

```

일부러 Exception을 발생시키는 예시입니다. 일반적인 Transaction 테스트의 경우 이렇게 간단히 테스트해볼 수 있죠. `mockK` 를 활용해서 대상 메소드에서 예외를 던지도록 spy 처리를 한 후 롤백되는 과정을 확인 하는 코드입니다.

좀 더 응용해서 `REQUIRES_NEW` 를 호출하는 케이스를 테스트 해 보겠습니다.

```kotlin
@Service
class OrderService(
    private val orderRepository: OrderRepository,
    private val logService: LogService
) {

    @Transactional
    fun placeOrderWithLog(order: Order) {
        orderRepository.save(order)
        
        // Propagation.REQUIRES_NEW로 별도의 트랜잭션에서 처리됨
        logService.saveLog("Order ${order.description} created")
    }
}

@Service
class LogService(private val logRepository: LogRepository) {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    fun saveLog(message: String) {
        val log = OrderLog(message = message)
        logRepository.save(log)
    }
}
```

예를 들어, 주문 정보를 저장하는 케이스에서 예외가 발생했다고 가정 해 보겠습니다. 로그성 데이터를 함께 적재해야하는 상황에서 로깅 Transaction 까지 롤백의 범위에 들어가서는 안되겠죠? 이런 케이스에서 REQUIRED_NEW 를 많이 사용합니다.

```kotlin
import io.kotest.core.spec.style.DescribeSpec
import io.kotest.matchers.shouldBe
import io.mockk.*
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest
import org.springframework.test.context.TestPropertySource

@DataJpaTest
@TestPropertySource(locations = ["classpath:application-test.properties"])
class OrderServiceLogTest @Autowired constructor(
    private val orderRepository: OrderRepository,
    private val logRepository: LogRepository
) : DescribeSpec({

    // MockK를 이용하여 LogService를 Spy로 생성
    val logService = spyk(LogService(logRepository))
    val orderService = OrderService(orderRepository, logService)

    describe("Propagation.REQUIRES_NEW 테스트") {

        context("메인 트랜잭션이 롤백되더라도 REQUIRES_NEW 로그는 DB에 남는다") {

            it("메인 트랜잭션은 정상 처리되지만 특정 조건에서 에러 로그만 남는다") {
                // Given
                val initialLogCount = logRepository.count()
                
                // Spy 설정: 특정 조건일 때만 예외처럼 동작하도록 설정
                every { logService.saveLog(match { it.contains("fail") }) } answers {
                    println("💡 Mocked Failure Log: $it")
                    val log = OrderLog(message = "Failed to process: $it")
                    logRepository.save(log)
                }

                // When
                orderService.placeOrderWithLog(Order(description = "fail-order"))

                // Then
                val finalLogCount = logRepository.count()
                finalLogCount shouldBe initialLogCount + 1

                // Verify: saveLog가 한 번만 호출되었는지 검증
                verify(exactly = 1) { logService.saveLog("Order fail-order created") }
            }

            afterEach {
                // MockK 초기화
                clearMocks(logService)
            }
        }
    }
})
```

이 경우엔 두 케이스를 모두 검증해야 합니다. 하나의 Transaction 이 실패해도 나머지 다른 Transaction 이 성공했는 지를 검증해야 합니다.

# 💡 응용 : @Transactional ReadOnly 를 활용한 Read/Write DB 분리

시스템의 규모가 커진다면 Read / Write 부하를 분산하기 위해 Read Replica (읽기 전용 DB), Write DB (쓰기 전용 DB) 을 흔히 나눠서 사용합니다. Read 에서는 SELECT, Write 에서는 INSERT, UPDATE, DELETE 를 담당하죠.

@Transaction 의 readOnly 옵션에 따라 Transaction 을 Read Replica 에 전달 할것인지 선택시킬 수 있습니다.

```yaml
spring:
  datasource:
    master:
      url: jdbc:mysql://localhost:3306/master_db
      username: root
      password: password
      driver-class-name: com.mysql.cj.jdbc.Driver

    replica:
      url: jdbc:mysql://localhost:3306/replica_db
      username: root
      password: password
      driver-class-name: com.mysql.cj.jdbc.Driver
```

예를 들어서 spring datasource 옵션을 이렇게 두 가지로 나누었다고 가정 해 보겠습니다.

```kotlin
import org.springframework.beans.factory.annotation.Qualifier
import org.springframework.boot.jdbc.DataSourceBuilder
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource
import javax.sql.DataSource

enum class DataSourceType {
    MASTER, REPLICA
}

object DataSourceContextHolder {
    private val contextHolder = ThreadLocal<DataSourceType>()

    fun set(dataSourceType: DataSourceType) {
        contextHolder.set(dataSourceType)
    }

    fun get(): DataSourceType {
        return contextHolder.get() ?: DataSourceType.MASTER
    }

    fun clear() {
        contextHolder.remove()
    }
}

@Configuration
class DataSourceConfig {

    @Bean
    @Qualifier("masterDataSource")
    fun masterDataSource(): DataSource {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/master_db")
            .username("root")
            .password("password")
            .driverClassName("com.mysql.cj.jdbc.Driver")
            .build()
    }

    @Bean
    @Qualifier("replicaDataSource")
    fun replicaDataSource(): DataSource {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/replica_db")
            .username("root")
            .password("password")
            .driverClassName("com.mysql.cj.jdbc.Driver")
            .build()
    }

    @Bean
    fun routingDataSource(
        @Qualifier("masterDataSource") masterDataSource: DataSource,
        @Qualifier("replicaDataSource") replicaDataSource: DataSource
    ): DataSource {
        val targetDataSources: MutableMap<Any, Any> = HashMap()
        targetDataSources[DataSourceType.MASTER] = masterDataSource
        targetDataSources[DataSourceType.REPLICA] = replicaDataSource

        val routingDataSource = object : AbstractRoutingDataSource() {
            override fun determineCurrentLookupKey(): Any {
                return DataSourceContextHolder.get()
            }
        }
        routingDataSource.setTargetDataSources(targetDataSources)
        routingDataSource.setDefaultTargetDataSource(masterDataSource)
        return routingDataSource
    }
}
```

이제 DataSource 옵션을 지정 해 주어야죠? 두 DataSource에 대해 각각의 @Bean 을 생성해준 후 routingDataSource Bean 에서 두 Bean 을 함께 사용할 수 있도록 지정합니다.

DataSourceContextHolder 는 Enum 값에 따라 DataSource 를 지정하도록 도와줍니다. Kotlin 에서 object 클래스는 Singleton 임을 보장해주는 역할을 합니다. *Spring IOC Container 의 관리가 굳이 필요없는 Singleton 이 필요한 경우 사용합니다.*

```kotlin
import org.aspectj.lang.annotation.Aspect
import org.aspectj.lang.annotation.Before
import org.springframework.stereotype.Component
import org.springframework.transaction.annotation.Transactional

@Aspect
@Component
class DataSourceRoutingAspect {

    @Before("@annotation(transactional) && execution(* com.example..*(..))")
    fun setDataSource(transactional: Transactional) {
        if (transactional.readOnly) {
            DataSourceContextHolder.set(DataSourceType.REPLICA)
        } else {
            DataSourceContextHolder.set(DataSourceType.MASTER)
        }
    }
}
```

AOP 를 통해 @Transactional 어노테이션의 옵션에 따라 DataSourceContextHolder 의 DataSourceType 을 바꿔줍니다.

```kotlin
import org.springframework.stereotype.Service
import org.springframework.transaction.annotation.Transactional

@Service
class UserService(private val userRepository: UserRepository) {

    private log = getLogger()

    @Transactional(readOnly = true)
    fun getUser(email: String): User {
        // 💡 Replica DB에서 조회
        log.info("🔍 Read from Replica")
        return userRepository.findByEmail(email) ?: throw RuntimeException("User not found")
    }

    @Transactional
    fun createUser(user: User): User {
        // 💡 Master DB에 저장
        log.info("📝 Write to Master")
        return userRepository.save(user)
    }
}
```

이제 readOnly 옵션에 따라 DB 를 분리해서 사용할 수 있습니다.

# Outro

지금까지 Spring 의 Transaction 어노테이션에 대해 자세히 알아 보았습니다.  
Spring 으로 로직 작성 하시면서 자주 사용하게 되지만, 얼마나 많은 옵션들이 있는 지 자세히 살펴고 써야 하는 기능이죠.

도움이 되셨다면 다행입니다. 다음 주제는 좀 더 Core 이야기를 해볼까 합니다. Spring Bean, ApplicationContext, Environment 에 대해서 깊게 딥다이브 해 보겠습니다.

# Reference

- [https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
