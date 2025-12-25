Here’s the **same Open–Closed Principle (OCP)** explained with a **completely different, real-world example**—no boxes, no volume—so the concept becomes crystal clear.

---

## SOLID – Open Closed Principle (OCP)

> **“Software entities should be open for extension but closed for modification.”**
> — Bertrand Meyer

Meaning:

* ✅ You **can add new behavior**
* ❌ You **should not change existing, tested code**

---

# Example: **Notification System**

Imagine you are building a system that sends notifications.

Initially, the system only supports **Email notifications**.

---

## ❌ Bad Design (Violates OCP)

### Initial Code

```java
class NotificationService {

    public void send(String type, String message) {
        if (type.equals("EMAIL")) {
            System.out.println("Sending Email: " + message);
        }
    }
}
```

### Later Requirement

Now business asks for:

* SMS notifications
* Push notifications

### Modified Code (Problem)

```java
class NotificationService {

    public void send(String type, String message) {
        if (type.equals("EMAIL")) {
            System.out.println("Sending Email: " + message);
        } else if (type.equals("SMS")) {
            System.out.println("Sending SMS: " + message);
        } else if (type.equals("PUSH")) {
            System.out.println("Sending Push Notification: " + message);
        }
    }
}
```

### ❌ What’s Wrong?

* Every new notification type **modifies existing code**
* High risk of bugs
* Class keeps growing
* Violates **Open–Closed Principle**

---

# ✅ Good Design (Follows OCP)

We **separate behavior** using **abstraction + polymorphism**.

---

## Step 1: Create an abstraction

```java
interface Notification {
    void send(String message);
}
```

---

## Step 2: Implement concrete classes

### Email Notification

```java
class EmailNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending Email: " + message);
    }
}
```

### SMS Notification

```java
class SmsNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}
```

### Push Notification

```java
class PushNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending Push Notification: " + message);
    }
}
```

---

## Step 3: Core Service (Closed for modification)

```java
class NotificationService {

    public void send(Notification notification, String message) {
        notification.send(message);
    }
}
```

---

## Usage

```java
public class Main {
    public static void main(String[] args) {
        NotificationService service = new NotificationService();

        service.send(new EmailNotification(), "Welcome!");
        service.send(new SmsNotification(), "OTP: 1234");
        service.send(new PushNotification(), "New Offer Available");
    }
}
```

---

## ✅ Why This Follows OCP

| Principle               | Explanation                              |
| ----------------------- | ---------------------------------------- |
| Open for extension      | Add WhatsApp, Slack, Teams notifications |
| Closed for modification | `NotificationService` never changes      |
| Loose coupling          | Service depends on interface             |
| Polymorphism            | Behavior decided at runtime              |

---

## Adding a New Feature (Without Touching Old Code)

### WhatsApp Notification (NEW)

```java
class WhatsAppNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending WhatsApp Message: " + message);
    }
}
```

✅ **No change** to:

* `NotificationService`
* Existing notification classes

