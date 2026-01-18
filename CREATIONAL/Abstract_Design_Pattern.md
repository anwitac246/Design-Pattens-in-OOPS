# Abstract Factory Design Pattern

## Before We Begin: Why Design Principles Matter (SRP & OCP)

Before jumping into the **Abstract Factory Design Pattern**, it is important to understand **why patterns exist at all**. Most design patterns are simply **practical applications of SOLID principles**, especially:

* **Single Responsibility Principle (SRP)**
* **Open/Closed Principle (OCP)**

Understanding these will make Abstract Factory feel natural instead of complicated.

---

## Single Responsibility Principle (SRP)

### Definition

A class should have **only one reason to change**, meaning it should have **one clearly defined responsibility**.

### Why SRP Is Important

* Keeps classes small and focused
* Makes code easier to understand and test
* Changes in one area do not break unrelated behavior

### Example Violation

```java
class User {
    void saveUser() {}
    void validatePassword() {}
    void sendWelcomeEmail() {}
}
```

This class:

* Manages user data
* Handles validation
* Sends emails

Too many responsibilities in one place.

### SRP-Compliant Solution

```java
class User {
    void saveUser() {}
}

class PasswordValidator {
    void validate(String password) {}
}

class EmailService {
    void sendWelcomeEmail() {}
}
```

Each class now has **one responsibility**.

---

## Open/Closed Principle (OCP)

### Definition

Software entities should be **open for extension but closed for modification**.

### Why OCP Is Important

* Reduces bugs
* Prevents breaking existing, tested code
* Encourages adding new behavior via new classes

### Example Violation

```java
class PaymentProcessor {
    void pay(String type) {
        if (type.equals("CARD")) {}
        else if (type.equals("PAYPAL")) {}
    }
}
```

Adding a new payment type means modifying this class.

### OCP-Compliant Solution

```java
interface PaymentMethod {
    void pay();
}

class CardPayment implements PaymentMethod {
    public void pay() {}
}

class PaypalPayment implements PaymentMethod {
    public void pay() {}
}

class PaymentProcessor {
    void process(PaymentMethod method) {
        method.pay();
    }
}
```

New payment methods are added **without changing existing code**.

---

## How SRP and OCP Relate to Factory Patterns

* **SRP**: Factories handle object creation, products handle behavior
* **OCP**: New products are added via new classes, not by modifying old code

Abstract Factory is a direct result of applying both principles together.

---

## What Is the Abstract Factory Design Pattern?

The **Abstract Factory Design Pattern** is very similar to the Factory Method pattern, but with **grouping**.

Think of it like this:

* Factory Method was for **one factory creating one type of product**
* Abstract Factory is for **one factory creating multiple related products together**

---

## Simple Restaurant Analogy

Initially, a restaurant only served burgers.

Later, it wants to serve:

* Burger
* Fries
* Coke

And it wants to serve them in **multiple styles**:

* American style
* Italian style

We do **not** want to mix styles accidentally.
We also want to keep the system **loosely coupled** so it can scale.

This is exactly where Abstract Factory fits.

---

## Step-by-Step Industry-Style Example

We will build a **meal system** using Abstract Factory.

---

## Step 1: Define Product Interfaces

Each product has a single responsibility (SRP).

```java
public interface Burger {
    void prepare();
}
```

```java
public interface Fries {
    void fry();
}
```

```java
public interface Drink {
    void pour();
}
```

---

## Step 2: Create Concrete Products (American Style)

```java
public class AmericanBurger implements Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing American Burger");
    }
}
```

```java
public class AmericanFries implements Fries {
    @Override
    public void fry() {
        System.out.println("Frying American Fries");
    }
}
```

```java
public class AmericanDrink implements Drink {
    @Override
    public void pour() {
        System.out.println("Pouring American Drink");
    }
}
```

---

## Step 3: Create Concrete Products (Italian Style)

```java
public class ItalianBurger implements Burger {
    @Override
    public void prepare() {
        System.out.println("Preparing Italian Burger");
    }
}
```

```java
public class ItalianFries implements Fries {
    @Override
    public void fry() {
        System.out.println("Frying Italian Fries");
    }
}
```

```java
public class ItalianDrink implements Drink {
    @Override
    public void pour() {
        System.out.println("Pouring Italian Drink");
    }
}
```

---

## Step 4: Define the Abstract Factory

This factory defines **what products belong together**.

```java
public interface MealFactory {
    Burger createBurger();
    Fries createFries();
    Drink createDrink();
}
```

---

## Step 5: Create Concrete Factories

Each factory produces a **consistent family of products**.

```java
public class AmericanMealFactory implements MealFactory {

    @Override
    public Burger createBurger() {
        return new AmericanBurger();
    }

    @Override
    public Fries createFries() {
        return new AmericanFries();
    }

    @Override
    public Drink createDrink() {
        return new AmericanDrink();
    }
}
```

```java
public class ItalianMealFactory implements MealFactory {

    @Override
    public Burger createBurger() {
        return new ItalianBurger();
    }

    @Override
    public Fries createFries() {
        return new ItalianFries();
    }

    @Override
    public Drink createDrink() {
        return new ItalianDrink();
    }
}
```

---

## Step 6: Client Code

The client depends **only on abstractions**.

```java
public class RestaurantApp {
    public static void main(String[] args) {
        MealFactory factory = new AmericanMealFactory();

        Burger burger = factory.createBurger();
        Fries fries = factory.createFries();
        Drink drink = factory.createDrink();

        burger.prepare();
        fries.fry();
        drink.pour();
    }
}
```

Switching styles means switching factories, not modifying logic.

---

## Why This Design Is Good

### SRP

* Each class has one responsibility
* Factories create objects
* Products implement behavior

### OCP

* New styles are added by creating new factories and products
* Existing code remains untouched

---

## When Should You Use Abstract Factory?

* When you need **families of related objects**
* When consistency between products matters
* When you want easy theme or vendor switching

---

## Drawbacks

* More classes and interfaces
* Adding a new product type affects all factories

---

## Final Takeaway

Abstract Factory is:

* Factory Method + grouping
* A direct application of SRP and OCP
* Best used when scaling systems with multiple related products

Understanding this pattern helps you think like a system designer, not just a programmer.
