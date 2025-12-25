# SOLID – Liskov Substitution Principle (LSP)

> **“Objects of a superclass should be replaceable with objects of its subclasses
> without breaking the correctness of the program.”**

In simple terms:

* If **B is a subtype of A**, then **B must behave like A**
* Subclasses **must not break expectations** of the parent class

---

# Example: **File Storage System**

Imagine a system that works with **files**.

---

## ❌ Bad Design (Violates LSP)

### Base Class

```java
class FileStorage {
    public void write(String data) {
        System.out.println("Writing data to file");
    }
}
```

---

### Subclass: Read-Only File

```java
class ReadOnlyFile extends FileStorage {

    @Override
    public void write(String data) {
        throw new UnsupportedOperationException("Cannot write to read-only file");
    }
}
```

---

### Client Code

```java
void saveData(FileStorage file) {
    file.write("Hello");
}
```

---

### ❌ What’s the Problem?

```java
FileStorage file = new ReadOnlyFile();
saveData(file); // Runtime exception!
```

| Expectation                | Reality                      |
| -------------------------- | ---------------------------- |
| FileStorage allows writing | ReadOnlyFile rejects writing |
| Substitution should work   | Program breaks               |

➡️ **Subclass cannot substitute superclass → LSP violated**

---

# ✅ Good Design (Follows LSP)

We **refine the abstraction** so behavior is consistent.

---

## Step 1: Separate Abstractions

```java
abstract class File {
    String name;
}
```

---

## Step 2: Writable Files

```java
abstract class WritableFile extends File {
    abstract void write(String data);
}
```

---

## Step 3: Concrete Implementations

### Normal File

```java
class NormalFile extends WritableFile {
    @Override
    void write(String data) {
        System.out.println("Writing data to normal file");
    }
}
```

---

### Read-Only File

```java
class ReadOnlyFile extends File {
    // No write() method here
}
```

---

## Step 4: Client Code (Safe)

```java
void saveData(WritableFile file) {
    file.write("Hello");
}
```

---

## ✅ Why This Follows LSP

| Rule                             | Satisfied |
| -------------------------------- | --------- |
| Subclasses honor parent contract | ✅         |
| No unexpected exceptions         | ✅         |
| Behavior preserved               | ✅         |

`ReadOnlyFile` is **not forced** to pretend it can write.
