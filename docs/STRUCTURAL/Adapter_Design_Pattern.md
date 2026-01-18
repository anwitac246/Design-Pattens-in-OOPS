# The Adapter Pattern: Making Incompatible Things Work Together

Ever traveled abroad and realized your phone charger doesn't fit the wall outlet? You didn't buy a new phone. You bought an adapter.

That's exactly what the Adapter pattern does for code. It lets incompatible interfaces work together without changing either one.

## The Problem: Interface Incompatibility

You're building an app that displays weather data. You wrote a nice interface:

```java
interface WeatherService {
    Temperature getTemperature(String city);
    Humidity getHumidity(String city);
}
```

Your code uses this interface everywhere. Then your company buys a license for a third-party weather API that looks like this:

```java
class ThirdPartyWeatherAPI {
    public WeatherData fetchWeatherData(String location, String apiKey) {
        // Returns complex object with temp in Fahrenheit
    }
}
```

Problems:
- Different method names
- Different parameters
- Different return types
- Temperature in Fahrenheit vs your system using Celsius

You can't change the third-party library. You don't want to rewrite all your code. What do you do?

## Enter the Adapter Pattern

The Adapter pattern wraps an incompatible interface and makes it compatible with what your code expects.

Think of it as a translator at a business meeting. Two people speak different languages, but the translator makes communication possible without either person learning a new language.

## Structure of the Adapter Pattern

There are four key players:

1. **Target Interface:** What your code expects
2. **Adaptee:** The incompatible class you need to use
3. **Adapter:** The translator that makes them work together
4. **Client:** Your code that uses the target interface

## Building a Weather API Adapter

### Step 1: Your Target Interface

This is what your existing code expects:

```java
public interface WeatherService {
    double getTemperatureInCelsius(String city);
    int getHumidityPercentage(String city);
}
```

### Step 2: The Incompatible Third-Party Class (Adaptee)

This is what you're stuck with:

```java
public class ThirdPartyWeatherAPI {
    
    public WeatherData fetchData(String location, String apiKey) {
        // Imagine this calls an external API
        System.out.println("Fetching data from third-party API for " + location);
        return new WeatherData(75.5, 65);  // Temp in Fahrenheit, humidity
    }
}

class WeatherData {
    private double temperatureFahrenheit;
    private int humidity;
    
    public WeatherData(double tempF, int humidity) {
        this.temperatureFahrenheit = tempF;
        this.humidity = humidity;
    }
    
    public double getTemperatureFahrenheit() {
        return temperatureFahrenheit;
    }
    
    public int getHumidity() {
        return humidity;
    }
}
```

Completely different structure. Different names. Different units.

### Step 3: The Adapter (The Magic)

```java
public class WeatherAPIAdapter implements WeatherService {
    
    private ThirdPartyWeatherAPI thirdPartyAPI;
    private String apiKey;
    
    public WeatherAPIAdapter(ThirdPartyWeatherAPI api, String apiKey) {
        this.thirdPartyAPI = api;
        this.apiKey = apiKey;
    }
    
    @Override
    public double getTemperatureInCelsius(String city) {
        // Fetch data using the third-party API
        WeatherData data = thirdPartyAPI.fetchData(city, apiKey);
        
        // Convert Fahrenheit to Celsius
        double fahrenheit = data.getTemperatureFahrenheit();
        return (fahrenheit - 32) * 5.0 / 9.0;
    }
    
    @Override
    public int getHumidityPercentage(String city) {
        WeatherData data = thirdPartyAPI.fetchData(city, apiKey);
        return data.getHumidity();
    }
}
```

Look what the adapter does:
- Implements your target interface
- Holds a reference to the adaptee
- Translates method calls
- Converts data formats (Fahrenheit to Celsius)
- Handles different parameter requirements

### Step 4: Using It

```java
public class WeatherApp {
    
    private WeatherService weatherService;
    
    public WeatherApp(WeatherService service) {
        this.weatherService = service;
    }
    
    public void displayWeather(String city) {
        double temp = weatherService.getTemperatureInCelsius(city);
        int humidity = weatherService.getHumidityPercentage(city);
        
        System.out.println("Weather in " + city + ":");
        System.out.println("Temperature: " + temp + "°C");
        System.out.println("Humidity: " + humidity + "%");
    }
    
    public static void main(String[] args) {
        // Create the third-party API instance
        ThirdPartyWeatherAPI thirdPartyAPI = new ThirdPartyWeatherAPI();
        
        // Wrap it with an adapter
        WeatherService service = new WeatherAPIAdapter(thirdPartyAPI, "api-key-123");
        
        // Use it like any other WeatherService
        WeatherApp app = new WeatherApp(service);
        app.displayWeather("London");
    }
}
```

Your `WeatherApp` doesn't know anything changed. It still uses the `WeatherService` interface. The adapter handles all the messy translation work behind the scenes.

## Two Types of Adapters

### Object Adapter (Composition)

This is what we just built. The adapter contains the adaptee:

```java
class Adapter implements Target {
    private Adaptee adaptee;  // Composition
    
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
}
```

**Pros:**
- More flexible
- Can adapt the class and its subclasses
- Follows composition over inheritance principle

### Class Adapter (Inheritance)

The adapter inherits from the adaptee:

```java
class Adapter extends Adaptee implements Target {
    // Adapter IS-A Adaptee and IS-A Target
}
```

**Pros:**
- Simpler (no extra object)
- Can override adaptee behavior

**Cons:**
- Only works in languages supporting multiple inheritance
- Less flexible
- Tighter coupling

In Java, you usually use the object adapter because Java doesn't allow multiple inheritance of classes.

## Real-World Example: Payment Gateway Adapter

You're building an e-commerce site. You start with Stripe:

```java
interface PaymentProcessor {
    boolean processPayment(double amount, String currency, CreditCard card);
    String getTransactionId();
}
```

Later, you want to support PayPal too:

```java
class PayPalSDK {
    public PayPalResponse sendPayment(String email, double dollars) {
        // PayPal's API
    }
}
```

Different interface, different parameters. Build an adapter:

```java
class PayPalAdapter implements PaymentProcessor {
    
    private PayPalSDK paypal;
    private String lastTransactionId;
    
    public PayPalAdapter(PayPalSDK paypal) {
        this.paypal = paypal;
    }
    
    @Override
    public boolean processPayment(double amount, String currency, CreditCard card) {
        // Convert currency if needed
        if (!currency.equals("USD")) {
            amount = convertToUSD(amount, currency);
        }
        
        // Extract email from card (simplified)
        String email = card.getEmail();
        
        // Call PayPal's API
        PayPalResponse response = paypal.sendPayment(email, amount);
        
        // Store transaction ID
        this.lastTransactionId = response.getTransactionId();
        
        return response.isSuccessful();
    }
    
    @Override
    public String getTransactionId() {
        return lastTransactionId;
    }
    
    private double convertToUSD(double amount, String currency) {
        // Currency conversion logic
        return amount;
    }
}
```

Now your checkout code works with both:

```java
class Checkout {
    private PaymentProcessor processor;
    
    public void checkout(double amount, CreditCard card) {
        boolean success = processor.processPayment(amount, "USD", card);
        if (success) {
            System.out.println("Payment successful! ID: " + processor.getTransactionId());
        }
    }
}

// Use Stripe
Checkout checkout1 = new Checkout(new StripeProcessor());

// Switch to PayPal with no code changes
Checkout checkout2 = new Checkout(new PayPalAdapter(new PayPalSDK()));
```

## Another Example: Legacy Code Integration

You're maintaining an old system with a weird interface:

```java
class LegacyRectangle {
    public void draw(int x1, int y1, int x2, int y2) {
        System.out.println("Drawing rectangle from (" + x1 + "," + y1 + 
                          ") to (" + x2 + "," + y2 + ")");
    }
}
```

But your new system uses this interface:

```java
interface Shape {
    void draw(int x, int y, int width, int height);
}
```

Adapter to the rescue:

```java
class RectangleAdapter implements Shape {
    
    private LegacyRectangle legacyRectangle;
    
    public RectangleAdapter(LegacyRectangle legacy) {
        this.legacyRectangle = legacy;
    }
    
    @Override
    public void draw(int x, int y, int width, int height) {
        // Convert from (x, y, width, height) to (x1, y1, x2, y2)
        int x1 = x;
        int y1 = y;
        int x2 = x + width;
        int y2 = y + height;
        
        legacyRectangle.draw(x1, y1, x2, y2);
    }
}
```

## When to Use the Adapter Pattern

Use Adapter when:
- You need to use an existing class with an incompatible interface
- You want to create a reusable class that works with unrelated classes
- You need to use several subclasses but don't want to adapt each one individually
- You're integrating third-party libraries
- You're working with legacy code

Common scenarios:
- Wrapping third-party APIs
- Database adapters (MySQL adapter, PostgreSQL adapter)
- UI framework adapters
- File format converters
- Protocol translators

## Adapter vs Other Patterns

### Adapter vs Facade

**Adapter:** Makes one interface compatible with another. Focuses on interface conversion.

**Facade:** Simplifies a complex system. Focuses on simplification, not compatibility.

### Adapter vs Decorator

**Adapter:** Changes the interface without adding functionality.

**Decorator:** Keeps the interface but adds functionality.

### Adapter vs Proxy

**Adapter:** Provides a different interface to an object.

**Proxy:** Provides the same interface but controls access.

## Common Mistakes

### Mistake 1: Adapter Does Too Much

```java
// BAD - Adapter adding business logic
class WeatherAdapter implements WeatherService {
    public double getTemperatureInCelsius(String city) {
        double temp = adaptee.getTemp(city);
        
        // This doesn't belong in an adapter!
        if (temp > 30) {
            sendHeatAlert();
        }
        
        return convert(temp);
    }
}
```

Adapters should only translate interfaces, not add business logic.

### Mistake 2: Two-Way Adapter

```java
// BAD - Trying to adapt in both directions
class TwoWayAdapter implements InterfaceA, InterfaceB {
    // Too complex, violates single responsibility
}
```

Keep adapters simple. One direction. One purpose.

### Mistake 3: Not Handling Errors

```java
// BAD - No error handling
class Adapter implements Target {
    public Result doSomething() {
        return adaptee.differentMethod();  // What if this throws?
    }
}
```

The adaptee might throw different exceptions. Handle them appropriately.

## Adapter and SOLID Principles

### Single Responsibility Principle

The adapter's only job is interface translation. Nothing else.

### Open/Closed Principle

You can integrate new adaptees without modifying existing code. Just create new adapters.

### Liskov Substitution Principle

The adapter can be used wherever the target interface is expected.

### Dependency Inversion Principle

Your code depends on the target interface, not concrete implementations or adaptees.

## The Mental Model

Think of the Adapter pattern as:

**Physical adapter:** Your laptop has USB-C. The monitor has HDMI. You buy a USB-C to HDMI adapter. The laptop and monitor don't change. The adapter makes them compatible.

**Human translator:** English speaker meets Japanese speaker. Translator sits between them. Neither learns the other's language. Translator makes communication possible.

**Currency exchanger:** You have dollars. Store accepts euros. Currency exchange booth converts between them. Neither changes their currency system.

## Performance Considerations

Adapters add a layer of indirection. Usually negligible, but consider:

- Extra method calls (usually JVM optimizes this away)
- Data conversion overhead (Fahrenheit to Celsius is cheap; complex object transformation might not be)
- Memory overhead (adapter object and reference to adaptee)

In 99% of cases, the performance cost is worth the flexibility and maintainability.

## Final Thoughts

The Adapter pattern is one of the most practical design patterns. You'll use it constantly in real-world development because you're always integrating with external systems, legacy code, and third-party libraries.

The key insight: don't force incompatible things to fit. Don't rewrite everything. Just add a thin translation layer that makes them work together.

It's not glamorous. It's not clever. It's practical problem-solving. And that's what makes it valuable.

Remember:
- Keep adapters simple
- Only translate interfaces
- Don't add business logic
- One adapter, one purpose

Next time you're wrestling with incompatible interfaces, remember the humble travel adapter. Sometimes the best solution is the simplest one: a translator that makes different things work together.

Adapt and overcome.