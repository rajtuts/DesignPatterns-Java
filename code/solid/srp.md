# Single Responsibility Principle (SRP)

# SOLID – Single Responsibility Principle (SRP)

> **“A class should have only one reason to change.”**
> — Robert C. Martin

In simple words:

* **One class → One job**
* If a class changes for **multiple reasons**, it violates SRP.

---

# Example: **User Registration System**

Imagine you are building a system to **register users**.

---

## ❌ Bad Design (Violates SRP)

### User class doing too much

```java
class UserService {

    public void registerUser(String username, String email) {
        // 1. Validate user
        if (username == null || email == null) {
            throw new IllegalArgumentException("Invalid data");
        }

        // 2. Save user to database
        System.out.println("Saving user to database");

        // 3. Send welcome email
        System.out.println("Sending welcome email");

        // 4. Log activity
        System.out.println("Logging registration activity");
    }
}
```

### ❌ What’s wrong here?

This class has **multiple responsibilities**:

| Responsibility | Reason to Change                   |
| -------------- | ---------------------------------- |
| Validation     | Validation rules may change        |
| Database logic | DB technology may change           |
| Email logic    | Email provider/template may change |
| Logging        | Logging format/tool may change     |

➡️ **One class, many reasons to change → SRP violated**

---

# ✅ Good Design (Follows SRP)

We split responsibilities into **focused classes**.

---

## 1️⃣ Validation Responsibility

```java
class UserValidator {
    public void validate(String username, String email) {
        if (username == null || email == null) {
            throw new IllegalArgumentException("Invalid data");
        }
    }
}
```

---

## 2️⃣ Persistence Responsibility

```java
class UserRepository {
    public void save(String username, String email) {
        System.out.println("Saving user to database");
    }
}
```

---

## 3️⃣ Email Responsibility

```java
class EmailService {
    public void sendWelcomeEmail(String email) {
        System.out.println("Sending welcome email");
    }
}
```

---

## 4️⃣ Logging Responsibility

```java
class AuditLogger {
    public void logRegistration(String username) {
        System.out.println("Logging registration for " + username);
    }
}
```

---

## 5️⃣ Orchestrator (Still SRP-compliant)

```java
class UserRegistrationService {

    private final UserValidator validator = new UserValidator();
    private final UserRepository repository = new UserRepository();
    private final EmailService emailService = new EmailService();
    private final AuditLogger logger = new AuditLogger();

    public void register(String username, String email) {
        validator.validate(username, email);
        repository.save(username, email);
        emailService.sendWelcomeEmail(email);
        logger.logRegistration(username);
    }
}
```

---

## ✅ Why This Follows SRP

| Class                   | Single Responsibility |
| ----------------------- | --------------------- |
| UserValidator           | Validate user input   |
| UserRepository          | Persist user data     |
| EmailService            | Send emails           |
| AuditLogger             | Log activity          |
| UserRegistrationService | Coordinate workflow   |

Each class has **exactly one reason to change**.

