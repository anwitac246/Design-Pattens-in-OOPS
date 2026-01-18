# Singleton Design Pattern

## What Is the Singleton Design Pattern?

The **Singleton Design Pattern** ensures that **only one object (instance) of a class exists** during the entire lifetime of an application and that this instance is **accessible from anywhere** in the program.

In simple words:

* You create the object **once**
* You reuse the **same object everywhere**

---

## Why Do We Need Singleton?

Sometimes, creating multiple objects of a class does not make sense and can even be harmful.

Examples:

* Database connection
* Logger
* Configuration settings
* Cache manager

These are things where:

* Multiple instances waste resources
* Multiple instances can cause inconsistent behavior

---

## Key Features of Singleton

* Only **one instance** exists at any time
* The instance is **globally accessible**
* Object creation is **controlled by the class itself**
* Often created **lazily** (only when needed)

---

## Real-Life Analogy

Think of a **printer in an office**:

* There is only one printer
* Everyone sends print requests to the same printer
* You don’t buy a new printer for every employee

That is how a Singleton works.

---

## Problems Singleton Solves

### 1. Accidental Object Creation

If anyone can create objects freely, the Singleton rule breaks.

### 2. Multiple Instances in Multithreading

If multiple threads create the object at the same time, more than one instance may be created.

---

## How Singleton Prevents These Problems

* Constructor is made **private**
* Instance is stored in a **static variable**
* Object creation happens in a **controlled method**

---

## Very Simple Singleton Example (Beginner Friendly)

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

---

## Step-by-Step Explanation of the Code

1. `private static Singleton instance;`

   * Stores the only object of the class
   * `static` means shared across the application

2. `private Singleton()`

   * Prevents usage of `new Singleton()` outside the class

3. `getInstance()` method

   * Creates the object only once
   * Returns the same object every time

---

## Limitation of the Simple Singleton

This implementation is **not thread-safe**.

If two threads enter `getInstance()` at the same time, both may see `instance == null` and create **two different objects**, breaking the Singleton rule.

To fix this, Java provides multiple approaches.

---

## Thread-Safe Singleton Using Synchronization

### Idea

Allow only **one thread at a time** to execute `getInstance()`.

### Implementation

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

### How This Helps

* `synchronized` ensures only one thread can access the method at a time
* Prevents multiple object creation

### Drawback

* Every call is synchronized
* Causes unnecessary performance overhead after the instance is created

---

## Double-Checked Locking (DCL) Singleton

### Why DCL Exists

We only need synchronization **during object creation**, not every time the instance is requested.

### Implementation

```java
class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### How This Helps

* First check avoids synchronization when instance already exists
* Synchronization happens only once
* `volatile` prevents partially created objects from being visible to other threads

---

## Enum-Based Singleton (Best and Safest Approach)

### Idea

Java guarantees that enum values are instantiated **only once**.

### Implementation

```java
enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("Doing work");
    }
}
```

### Usage

```java
Singleton s = Singleton.INSTANCE;
s.doSomething();
```

### How This Helps

* Thread-safe by design
* Safe against serialization and reflection
* No synchronization required

---

## When Should You Use Singleton?

Use Singleton when:

* Exactly one object is needed
* The object manages shared resources
* Global access is required

---

## When Should You Avoid Singleton?

Avoid Singleton when:

* You need multiple configurations
* You want easy unit testing
* You want loosely coupled code

---

## Beginner Takeaway

* Singleton controls object creation
* Only one instance exists
* Thread safety is crucial
* Enum Singleton is the safest choice
* Use Singleton carefully and intentionally
