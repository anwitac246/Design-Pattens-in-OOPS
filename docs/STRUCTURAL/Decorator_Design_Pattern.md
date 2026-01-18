# The Decorator Pattern: Adding Features Without the Mess

Picture this: You're at a coffee shop. You order a coffee. Simple enough. But then you want milk. Then sugar. Then a shot of vanilla. Then whipped cream. Are these four different types of coffee? Or is it one coffee with additions?

That's the Decorator pattern. Instead of creating a new class for every possible combination, you wrap objects with additional functionality.

## The Problem: Combinatorial Explosion

Let's say you're building a coffee shop app. You start simple:

```java
class SimpleCoffee {
    double cost() { return 2.0; }
    String description() { return "Simple Coffee"; }
}
```

Then customers want options:
- Coffee with milk
- Coffee with sugar
- Coffee with whipped cream

The naive approach is creating subclasses:

```java
class CoffeeWithMilk extends SimpleCoffee { }
class CoffeeWithSugar extends SimpleCoffee { }
class CoffeeWithWhippedCream extends SimpleCoffee { }
```

But wait, customers want combinations:
- Coffee with milk and sugar
- Coffee with milk and whipped cream
- Coffee with sugar and whipped cream
- Coffee with milk, sugar, and whipped cream

Now you need:
```java
class CoffeeWithMilkAndSugar extends SimpleCoffee { }
class CoffeeWithMilkAndWhippedCream extends SimpleCoffee { }
class CoffeeWithSugarAndWhippedCream extends SimpleCoffee { }
class CoffeeWithMilkSugarAndWhippedCream extends SimpleCoffee { }
```

Add two more toppings and you'd need 32 classes. Add five and you need over 1000. This is combinatorial explosion, and it's a nightmare to maintain.

## The Decorator Solution

Instead of creating classes for every combination, you wrap the base object with decorators that add functionality.

Think of it like dressing up:
- You start with basic clothes (base object)
- You add a jacket (decorator)
- You add a scarf (another decorator)
- You add gloves (another decorator)

Each layer adds something without changing what's underneath.

## Building the Coffee Decorator System

### Step 1: Define the Component Interface

```java
public interface Coffee {
    double cost();
    String description();
}
```

This is what all coffees (base and decorated) must implement.

### Step 2: Create the Base Component

```java
public class SimpleCoffee implements Coffee {
    
    @Override
    public double cost() {
        return 2.0;
    }
    
    @Override
    public String description() {
        return "Simple Coffee";
    }
}
```

This is your starting point. Plain coffee, no frills.

### Step 3: Create the Abstract Decorator

```java
public abstract class CoffeeDecorator implements Coffee {
    
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    @Override
    public double cost() {
        return decoratedCoffee.cost();
    }
    
    @Override
    public String description() {
        return decoratedCoffee.description();
    }
}
```

This is the wrapper template. It implements `Coffee`, holds a reference to another `Coffee`, and delegates to it by default.

### Step 4: Create Concrete Decorators

```java
public class MilkDecorator extends CoffeeDecorator {
    
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double cost() {
        return decoratedCoffee.cost() + 0.5;  // Add milk cost
    }
    
    @Override
    public String description() {
        return decoratedCoffee.description() + ", Milk";
    }
}
```

```java
public class SugarDecorator extends CoffeeDecorator {
    
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double cost() {
        return decoratedCoffee.cost() + 0.2;
    }
    
    @Override
    public String description() {
        return decoratedCoffee.description() + ", Sugar";
    }
}
```

```java
public class WhippedCreamDecorator extends CoffeeDecorator {
    
    public WhippedCreamDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double cost() {
        return decoratedCoffee.cost() + 0.7;
    }
    
    @Override
    public String description() {
        return decoratedCoffee.description() + ", Whipped Cream";
    }
}
```

Each decorator adds its own cost and description to whatever it's wrapping.

### Step 5: Using the Decorators

```java
public class CoffeeShop {
    public static void main(String[] args) {
        
        // Plain coffee
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.description() + " costs $" + coffee.cost());
        // Output: Simple Coffee costs $2.0
        
        // Coffee with milk
        coffee = new MilkDecorator(new SimpleCoffee());
        System.out.println(coffee.description() + " costs $" + coffee.cost());
        // Output: Simple Coffee, Milk costs $2.5
        
        // Coffee with milk and sugar
        coffee = new SugarDecorator(new MilkDecorator(new SimpleCoffee()));
        System.out.println(coffee.description() + " costs $" + coffee.cost());
        // Output: Simple Coffee, Milk, Sugar costs $2.7
        
        // Coffee with everything
        coffee = new WhippedCreamDecorator(
                    new SugarDecorator(
                        new MilkDecorator(
                            new SimpleCoffee())));
        System.out.println(coffee.description() + " costs $" + coffee.cost());
        // Output: Simple Coffee, Milk, Sugar, Whipped Cream costs $3.4
    }
}
```

Look at what's happening:
- Start with `SimpleCoffee`
- Wrap it with `MilkDecorator`
- Wrap that with `SugarDecorator`
- Wrap that with `WhippedCreamDecorator`

Each wrapper adds functionality. The nesting might look weird at first, but it's powerful. You can create any combination without creating new classes.

## How the Chain Works

When you call `coffee.cost()` on the fully decorated coffee:

1. `WhippedCreamDecorator.cost()` is called
2. It calls `decoratedCoffee.cost()` (which is the `SugarDecorator`)
3. `SugarDecorator.cost()` calls its `decoratedCoffee.cost()` (the `MilkDecorator`)
4. `MilkDecorator.cost()` calls its `decoratedCoffee.cost()` (the `SimpleCoffee`)
5. `SimpleCoffee.cost()` returns 2.0
6. `MilkDecorator` adds 0.5, returns 2.5
7. `SugarDecorator` adds 0.2, returns 2.7
8. `WhippedCreamDecorator` adds 0.7, returns 3.4

It's a chain reaction, unwinding from the innermost object outward.

## Real-World Example: Text Formatting

You have a text component. You want to add formatting without creating classes for every combination.

```java
interface Text {
    String render();
}

class PlainText implements Text {
    private String content;
    
    public PlainText(String content) {
        this.content = content;
    }
    
    @Override
    public String render() {
        return content;
    }
}
```

Now the decorators:

```java
abstract class TextDecorator implements Text {
    protected Text decoratedText;
    
    public TextDecorator(Text text) {
        this.decoratedText = text;
    }
    
    @Override
    public String render() {
        return decoratedText.render();
    }
}

class BoldDecorator extends TextDecorator {
    public BoldDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String render() {
        return "<b>" + decoratedText.render() + "</b>";
    }
}

class ItalicDecorator extends TextDecorator {
    public ItalicDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String render() {
        return "<i>" + decoratedText.render() + "</i>";
    }
}

class UnderlineDecorator extends TextDecorator {
    public UnderlineDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String render() {
        return "<u>" + decoratedText.render() + "</u>";
    }
}
```

Usage:

```java
Text text = new PlainText("Hello World");

// Bold
text = new BoldDecorator(text);
System.out.println(text.render());  // <b>Hello World</b>

// Bold and italic
text = new ItalicDecorator(new BoldDecorator(new PlainText("Hello World")));
System.out.println(text.render());  // <i><b>Hello World</b></i>

// Bold, italic, and underlined
text = new UnderlineDecorator(
           new ItalicDecorator(
               new BoldDecorator(
                   new PlainText("Hello World"))));
System.out.println(text.render());  // <u><i><b>Hello World</b></i></u>
```

Three classes give you any combination of formatting. Add strikethrough? Create one class. Instantly compatible with all existing decorators.

## Java's Built-In Example: I/O Streams

You've been using the Decorator pattern without knowing it:

```java
// Plain file input
InputStream input = new FileInputStream("file.txt");

// Buffered (decorator)
input = new BufferedInputStream(new FileInputStream("file.txt"));

// Buffered and compressed (two decorators)
input = new GZIPInputStream(
            new BufferedInputStream(
                new FileInputStream("file.txt")));
```

`FileInputStream` is the base component. `BufferedInputStream` adds buffering. `GZIPInputStream` adds decompression. Each layer adds functionality.

## Decorator vs Inheritance

Why not just use inheritance?

### Inheritance:
```java
class MilkCoffee extends SimpleCoffee { }
class SugarMilkCoffee extends MilkCoffee { }
class WhippedCreamSugarMilkCoffee extends SugarMilkCoffee { }
```

Problems:
- Fixed at compile time
- Class explosion for combinations
- Rigid hierarchy
- Can't add features dynamically

### Decorator:
```java
Coffee coffee = new SimpleCoffee();
if (customerWantsMilk) coffee = new MilkDecorator(coffee);
if (customerWantsSugar) coffee = new SugarDecorator(coffee);
if (customerWantsWhippedCream) coffee = new WhippedCreamDecorator(coffee);
```

Benefits:
- Runtime composition
- Minimal classes
- Flexible combinations
- Easy to extend

## When to Use Decorator

Use Decorator when:
- You need to add responsibilities to objects dynamically
- You want to add features without subclassing
- Extension by subclassing is impractical
- You have many independent optional features

Real scenarios:
- GUI components (borders, scrollbars, etc.)
- I/O streams
- Middleware in web frameworks
- Logging wrappers
- Caching layers
- Authorization checks

## Common Pitfalls

### Pitfall 1: Too Many Small Objects

Each decorator is a separate object. Deep nesting creates many objects.

```java
// Creates 4 objects
Coffee coffee = new WhippedCreamDecorator(
                    new SugarDecorator(
                        new MilkDecorator(
                            new SimpleCoffee())));
```

Usually not a problem, but be aware of memory implications for massive scales.

### Pitfall 2: Order Matters

```java
Text text1 = new BoldDecorator(new ItalicDecorator(new PlainText("Hi")));
// <b><i>Hi</i></b>

Text text2 = new ItalicDecorator(new BoldDecorator(new PlainText("Hi")));
// <i><b>Hi</b></i>
```

Different order, different result. Make sure order independence is handled if needed.

### Pitfall 3: Identity Checking

```java
SimpleCoffee coffee = new SimpleCoffee();
Coffee decoratedCoffee = new MilkDecorator(coffee);

if (decoratedCoffee instanceof SimpleCoffee) {  // False!
    // This won't execute
}
```

Decorated objects aren't the same type as the base component. Be careful with type checking.

### Pitfall 4: Complex Decorator Chains

```java
// What is this?
Coffee coffee = new DecoratorA(new DecoratorB(new DecoratorC(
                new DecoratorD(new DecoratorE(new SimpleCoffee())))));
```

Too many layers become hard to understand. Consider using a builder for complex compositions.

## Decorator with Builder Pattern

Make complex decorator chains more readable:

```java
class CoffeeBuilder {
    private Coffee coffee;
    
    public CoffeeBuilder() {
        this.coffee = new SimpleCoffee();
    }
    
    public CoffeeBuilder addMilk() {
        coffee = new MilkDecorator(coffee);
        return this;
    }
    
    public CoffeeBuilder addSugar() {
        coffee = new SugarDecorator(coffee);
        return this;
    }
    
    public CoffeeBuilder addWhippedCream() {
        coffee = new WhippedCreamDecorator(coffee);
        return this;
    }
    
    public Coffee build() {
        return coffee;
    }
}
```

Usage:

```java
Coffee coffee = new CoffeeBuilder()
                    .addMilk()
                    .addSugar()
                    .addWhippedCream()
                    .build();
```

Much more readable than nested constructors.

## Decorator vs Other Patterns

### Decorator vs Adapter

**Decorator:** Same interface, adds functionality
**Adapter:** Different interface, enables compatibility

### Decorator vs Proxy

**Decorator:** Adds functionality, client knows it's wrapped
**Proxy:** Controls access, client might not know

### Decorator vs Composite

**Decorator:** Adds features to individual objects
**Composite:** Treats groups of objects as one

## Decorator and SOLID Principles

### Single Responsibility Principle

Each decorator has one job: add one specific feature.

### Open/Closed Principle

Open for extension (add new decorators), closed for modification (don't change existing code).

### Liskov Substitution Principle

Decorated objects can replace base objects anywhere.

### Dependency Inversion Principle

Code depends on the interface, not concrete decorators.

## The Mental Model

Think of Russian nesting dolls (matryoshka):
- Smallest doll in the center (base component)
- Each larger doll wraps the smaller one (decorators)
- Each doll is complete on its own
- Together they form a whole

Or think of gift wrapping:
- Gift in a box (base component)
- Wrapped in paper (decorator)
- Tied with ribbon (decorator)
- Attached card (decorator)

Each layer adds something without changing what's inside.

## Performance Considerations

Decorators add method call overhead. Each decorator in the chain is an extra method call.

Usually negligible, but consider:
- Deep nesting in hot paths
- Very performance-critical code
- Thousands of objects decorated deeply

Profile before optimizing. In most cases, the flexibility is worth the tiny overhead.

## Final Thoughts

The Decorator pattern is about composability. Instead of creating a new class for every feature combination, you compose features at runtime.

It's elegant. It's flexible. It follows SOLID principles. And it scales beautifully.

The key insight: you don't need a class for every possible combination of features. You need a class for each feature, and a way to compose them.

Remember:
- Keep decorators simple and focused
- One decorator, one additional feature
- Consider readability with deep nesting
- Use builders for complex compositions

Next time you find yourself creating classes like `UserWithLoggingAndCachingAndValidation`, stop. Think decorator. Compose, don't explode.

Wrap it up.