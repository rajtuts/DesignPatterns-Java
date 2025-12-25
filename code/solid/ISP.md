# SOLID – Interface Segregation Principle (ISP)

> **“Clients should not be forced to depend on methods they do not use.”**

In simple words:

* ❌ Don’t create **fat interfaces**
* ✅ Create **small, role-specific interfaces**

---

# Example: **Office Printer System**

Imagine you are designing software for **office machines**.

Some machines:

* Can **print**
* Can **scan**
* Can **fax**
* Can **only print** (basic printers)

---

## ❌ Bad Design (Violates ISP)

### One Big Interface

```java
interface OfficeMachine {
    void print();
    void scan();
    void fax();
}
```

---

### Basic Printer Implementation

```java
class BasicPrinter implements OfficeMachine {

    @Override
    public void print() {
        System.out.println("Printing document");
    }

    @Override
    public void scan() {
        throw new UnsupportedOperationException("Scan not supported");
    }

    @Override
    public void fax() {
        throw new UnsupportedOperationException("Fax not supported");
    }
}
```

---

### ❌ What’s Wrong?

| Problem            | Explanation                               |
| ------------------ | ----------------------------------------- |
| Forced methods     | Printer is forced to implement scan & fax |
| Runtime exceptions | UnsupportedOperationException             |
| Poor design        | Clients depend on unused methods          |
| ISP violated       | Fat interface                             |

➡️ **BasicPrinter depends on methods it doesn’t need**

---

# ✅ Good Design (Follows ISP)

We **split the interface** into **small, focused interfaces**.

---

## Step 1: Segregated Interfaces

```java
interface Printable {
    void print();
}

interface Scannable {
    void scan();
}

interface Faxable {
    void fax();
}
```

---

## Step 2: Implement Only What’s Needed

### Basic Printer

```java
class BasicPrinter implements Printable {

    @Override
    public void print() {
        System.out.println("Printing document");
    }
}
```

---

### Multi-Function Printer

```java
class MultiFunctionPrinter implements Printable, Scannable, Faxable {

    @Override
    public void print() {
        System.out.println("Printing document");
    }

    @Override
    public void scan() {
        System.out.println("Scanning document");
    }

    @Override
    public void fax() {
        System.out.println("Faxing document");
    }
}
```

---

## ✅ Why This Follows ISP

| Class                | Interfaces Implemented        |
| -------------------- | ----------------------------- |
| BasicPrinter         | Printable                     |
| MultiFunctionPrinter | Printable, Scannable, Faxable |

✔ No forced methods
✔ No dummy implementations
✔ Clean, flexible design

