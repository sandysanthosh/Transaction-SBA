The `@Transactional` annotation in Spring Boot is used to manage transactions declaratively. It allows you to specify the transactional boundaries of methods, ensuring that a group of operations are executed within a transaction context. If any operation within the transaction fails, the entire transaction can be rolled back to maintain data integrity.

### How `@Transactional` Works

1. **Declarative Transaction Management**:
   - By annotating a method with `@Transactional`, you tell Spring that the method should be executed within a transaction. Spring will automatically handle the beginning, committing, and rolling back of transactions.

2. **Transaction Propagation**:
   - The `@Transactional` annotation allows you to specify the propagation behavior, which defines how transactions should be managed when calling other transactional methods. Common propagation types include:
     - `REQUIRED`: Use an existing transaction if one exists; otherwise, create a new one.
     - `REQUIRES_NEW`: Always create a new transaction, suspending any existing transaction.
     - `MANDATORY`: Must be called within an existing transaction; otherwise, an exception is thrown.
     - `SUPPORTS`: Execute within a transaction if one exists; otherwise, execute non-transactionally.
     - `NOT_SUPPORTED`: Execute non-transactionally, suspending any existing transaction.
     - `NEVER`: Execute non-transactionally; throw an exception if a transaction exists.
     - `NESTED`: Execute within a nested transaction if one exists; otherwise, create a new one.

3. **Transaction Rollback**:
   - You can specify the types of exceptions that should trigger a transaction rollback using the `rollbackFor` and `noRollbackFor` attributes. By default, transactions are rolled back on runtime exceptions (`unchecked exceptions`) and not on checked exceptions.

### Example Usage

Here is a basic example to illustrate how `@Transactional` is used in a Spring Boot application:

#### Maven Dependencies

Make sure you have the necessary dependencies in your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

#### Application Configuration

```java
@SpringBootApplication
@EnableTransactionManagement
public class TransactionDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(TransactionDemoApplication.class, args);
    }
}
```

#### Entity Class

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private Double balance;

    // getters and setters
}
```

#### Repository Interface

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface AccountRepository extends JpaRepository<Account, Long> {
}
```

#### Service Class

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class AccountService {

    @Autowired
    private AccountRepository accountRepository;

    @Transactional
    public void transfer(Long fromAccountId, Long toAccountId, Double amount) {
        Account fromAccount = accountRepository.findById(fromAccountId).orElseThrow(() -> new RuntimeException("Account not found"));
        Account toAccount = accountRepository.findById(toAccountId).orElseThrow(() -> new RuntimeException("Account not found"));

        if (fromAccount.getBalance() < amount) {
            throw new RuntimeException("Insufficient balance");
        }

        fromAccount.setBalance(fromAccount.getBalance() - amount);
        toAccount.setBalance(toAccount.getBalance() + amount);

        accountRepository.save(fromAccount);
        accountRepository.save(toAccount);
    }
}
```

#### Controller Class

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AccountController {

    @Autowired
    private AccountService accountService;

    @PostMapping("/transfer")
    public String transfer(@RequestParam Long fromAccountId, @RequestParam Long toAccountId, @RequestParam Double amount) {
        try {
            accountService.transfer(fromAccountId, toAccountId, amount);
            return "Transfer successful";
        } catch (Exception e) {
            return "Transfer failed: " + e.getMessage();
        }
    }
}
```

### Key Points to Remember

- **Transactional Methods**: Ensure that `@Transactional` is applied to public methods. Spring will not manage transactions for protected, private, or package-visible methods.
- **Proxy Mechanism**: Spring uses proxies to manage transactions. Thus, self-invocation (calling a transactional method from within the same class) will not go through the proxy, and the transaction will not be managed.
- **Isolation Levels**: You can control the transaction's isolation level using the `isolation` attribute of `@Transactional`, which determines how data access operations are isolated from each other.
- **Read-Only Transactions**: You can mark a transaction as read-only using the `readOnly` attribute, which hints to the underlying database to optimize transaction handling.

By understanding and effectively using the `@Transactional` annotation, you can ensure robust transaction management in your Spring Boot applications.
