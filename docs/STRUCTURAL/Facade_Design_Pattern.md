# The Facade Pattern: Simplifying the Complex

Ever used a TV remote? You press one button and it turns on the TV, sets the volume, switches to your favorite channel, and adjusts the brightness. Behind that one button, there's a sequence of complex operations across multiple systems.

That's the Facade pattern. It provides a simple interface to a complex subsystem.

## The Problem: Too Many Moving Parts

Imagine you're building a home theater system. To watch a movie, you need to:

```java
// Turn on the projector
projector.on();
projector.setInput("DVD");

// Set up the screen
screen.down();

// Turn on the sound system
amplifier.on();
amplifier.setDVD(dvd);
amplifier.setSurroundSound();
amplifier.setVolume(11);

// Start the DVD
dvd.on();
dvd.play(movie);

// Dim the lights
lights.dim(10);
```

Seven different components. Multiple steps. Easy to forget something. Hard to remember the right order.

And when the movie ends, you have to reverse all of this manually. What a pain.

## The Facade Solution

Create one simple interface that hides all this complexity:

```java
homeTheater.watchMovie("The Matrix");
```

That's it. One method call. The facade handles all the coordination behind the scenes.

## Building a Home Theater Facade

### Step 1: The Complex Subsystem Components

These are the components we're trying to simplify:

```java
class Projector {
    public void on() {
        System.out.println("Projector on");
    }
    
    public void off() {
        System.out.println("Projector off");
    }
    
    public void setInput(String input) {
        System.out.println("Projector input set to " + input);
    }
}

class Screen {
    public void down() {
        System.out.println("Screen coming down");
    }
    
    public void up() {
        System.out.println("Screen going up");
    }
}

class Amplifier {
    public void on() {
        System.out.println("Amplifier on");
    }
    
    public void off() {
        System.out.println("Amplifier off");
    }
    
    public void setDVD(DVDPlayer dvd) {
        System.out.println("Amplifier set to DVD");
    }
    
    public void setSurroundSound() {
        System.out.println("Surround sound enabled");
    }
    
    public void setVolume(int level) {
        System.out.println("Volume set to " + level);
    }
}

class DVDPlayer {
    public void on() {
        System.out.println("DVD player on");
    }
    
    public void off() {
        System.out.println("DVD player off");
    }
    
    public void play(String movie) {
        System.out.println("Playing " + movie);
    }
    
    public void stop() {
        System.out.println("DVD stopped");
    }
}

class Lights {
    public void dim(int level) {
        System.out.println("Lights dimmed to " + level + "%");
    }
    
    public void on() {
        System.out.println("Lights on");
    }
}
```

Each component has its own interface. Each is complex in its own right.

### Step 2: The Facade

```java
public class HomeTheaterFacade {
    
    private Projector projector;
    private Screen screen;
    private Amplifier amplifier;
    private DVDPlayer dvd;
    private Lights lights;
    
    public HomeTheaterFacade(Projector projector, Screen screen,
                             Amplifier amplifier, DVDPlayer dvd,
                             Lights lights) {
        this.projector = projector;
        this.screen = screen;
        this.amplifier = amplifier;
        this.dvd = dvd;
        this.lights = lights;
    }
    
    public void watchMovie(String movie) {
        System.out.println("Get ready to watch a movie...\n");
        
        lights.dim(10);
        screen.down();
        projector.on();
        projector.setInput("DVD");
        amplifier.on();
        amplifier.setDVD(dvd);
        amplifier.setSurroundSound();
        amplifier.setVolume(11);
        dvd.on();
        dvd.play(movie);
        
        System.out.println("\nEnjoy your movie!");
    }
    
    public void endMovie() {
        System.out.println("Shutting down the home theater...\n");
        
        dvd.stop();
        dvd.off();
        amplifier.off();
        projector.off();
        screen.up();
        lights.on();
        
        System.out.println("\nHome theater is off");
    }
}
```

The facade coordinates all the subsystems. It knows the right order, the right settings, and handles all the complexity.

### Step 3: Using the Facade

```java
public class Client {
    public static void main(String[] args) {
        // Set up the subsystems
        Projector projector = new Projector();
        Screen screen = new Screen();
        Amplifier amplifier = new Amplifier();
        DVDPlayer dvd = new DVDPlayer();
        Lights lights = new Lights();
        
        // Create the facade
        HomeTheaterFacade homeTheater = new HomeTheaterFacade(
            projector, screen, amplifier, dvd, lights
        );
        
        // Simple interface to complex operations
        homeTheater.watchMovie("The Matrix");
        
        // Some time later...
        homeTheater.endMovie();
    }
}
```

Look how clean the client code is. No knowledge of the subsystems required. No worrying about order or settings.

## Real-World Example: Order Processing System

E-commerce order processing involves many subsystems:

```java
class InventorySystem {
    public boolean checkStock(String productId, int quantity) {
        System.out.println("Checking inventory for " + productId);
        return true;
    }
    
    public void reserveStock(String productId, int quantity) {
        System.out.println("Reserving " + quantity + " units of " + productId);
    }
}

class PaymentGateway {
    public boolean processPayment(String cardNumber, double amount) {
        System.out.println("Processing payment of $" + amount);
        return true;
    }
}

class ShippingService {
    public String scheduleShipping(String address, String productId) {
        System.out.println("Scheduling shipping to " + address);
        return "TRACK-123";
    }
}

class NotificationService {
    public void sendOrderConfirmation(String email, String orderId) {
        System.out.println("Sending confirmation email to " + email);
    }
    
    public void sendShippingUpdate(String email, String trackingNumber) {
        System.out.println("Sending tracking info to " + email);
    }
}

class InvoiceSystem {
    public void generateInvoice(String orderId, double amount) {
        System.out.println("Generating invoice for order " + orderId);
    }
}
```

Without a facade, the client needs to coordinate all of these:

```java
// BAD: Client handles all the complexity
InventorySystem inventory = new InventorySystem();
PaymentGateway payment = new PaymentGateway();
ShippingService shipping = new ShippingService();
NotificationService notification = new NotificationService();
InvoiceSystem invoice = new InvoiceSystem();

// Check inventory
if (!inventory.checkStock("PROD-123", 1)) {
    return "Out of stock";
}

// Reserve stock
inventory.reserveStock("PROD-123", 1);

// Process payment
if (!payment.processPayment("1234-5678", 99.99)) {
    // Roll back inventory
    return "Payment failed";
}

// Schedule shipping
String tracking = shipping.scheduleShipping("123 Main St", "PROD-123");

// Generate invoice
invoice.generateInvoice("ORD-456", 99.99);

// Send notifications
notification.sendOrderConfirmation("user@email.com", "ORD-456");
notification.sendShippingUpdate("user@email.com", tracking);
```

That's a lot of responsibility on the client. What if order processing logic changes? Every client needs to be updated.

Now with a facade:

```java
public class OrderFacade {
    
    private InventorySystem inventory;
    private PaymentGateway payment;
    private ShippingService shipping;
    private NotificationService notification;
    private InvoiceSystem invoice;
    
    public OrderFacade(InventorySystem inventory, PaymentGateway payment,
                       ShippingService shipping, NotificationService notification,
                       InvoiceSystem invoice) {
        this.inventory = inventory;
        this.payment = payment;
        this.shipping = shipping;
        this.notification = notification;
        this.invoice = invoice;
    }
    
    public String placeOrder(String productId, int quantity, String cardNumber,
                            double amount, String shippingAddress, String email) {
        
        // Check inventory
        if (!inventory.checkStock(productId, quantity)) {
            return "Order failed: Out of stock";
        }
        
        // Reserve stock
        inventory.reserveStock(productId, quantity);
        
        // Process payment
        if (!payment.processPayment(cardNumber, amount)) {
            // Could add rollback logic here
            return "Order failed: Payment declined";
        }
        
        // Generate order ID
        String orderId = generateOrderId();
        
        // Schedule shipping
        String trackingNumber = shipping.scheduleShipping(shippingAddress, productId);
        
        // Generate invoice
        invoice.generateInvoice(orderId, amount);
        
        // Send notifications
        notification.sendOrderConfirmation(email, orderId);
        notification.sendShippingUpdate(email, trackingNumber);
        
        return "Order placed successfully: " + orderId;
    }
    
    private String generateOrderId() {
        return "ORD-" + System.currentTimeMillis();
    }
}
```

Client code becomes trivial:

```java
OrderFacade orderSystem = new OrderFacade(
    inventory, payment, shipping, notification, invoice
);

String result = orderSystem.placeOrder(
    "PROD-123", 1, "1234-5678", 99.99,
    "123 Main St", "user@email.com"
);

System.out.println(result);
```

One method call. All complexity hidden. If the order process changes, only the facade needs updating.

## Another Example: Database Operations

```java
class DatabaseConnection {
    public void connect(String url) {
        System.out.println("Connecting to " + url);
    }
    
    public void disconnect() {
        System.out.println("Disconnecting");
    }
}

class QueryBuilder {
    public String buildQuery(String table, String condition) {
        return "SELECT * FROM " + table + " WHERE " + condition;
    }
}

class ResultParser {
    public List<String> parseResults(String rawData) {
        System.out.println("Parsing results");
        return Arrays.asList("Result1", "Result2");
    }
}

class ConnectionPool {
    public void returnConnection(DatabaseConnection conn) {
        System.out.println("Returning connection to pool");
    }
}
```

Facade to simplify database access:

```java
public class DatabaseFacade {
    
    private DatabaseConnection connection;
    private QueryBuilder queryBuilder;
    private ResultParser parser;
    private ConnectionPool pool;
    
    public DatabaseFacade(String dbUrl) {
        this.connection = new DatabaseConnection();
        this.queryBuilder = new QueryBuilder();
        this.parser = new ResultParser();
        this.pool = new ConnectionPool();
        
        connection.connect(dbUrl);
    }
    
    public List<String> findData(String table, String condition) {
        // Build query
        String query = queryBuilder.buildQuery(table, condition);
        
        // Execute (simplified)
        System.out.println("Executing: " + query);
        String rawData = "raw database results";
        
        // Parse results
        List<String> results = parser.parseResults(rawData);
        
        return results;
    }
    
    public void close() {
        connection.disconnect();
        pool.returnConnection(connection);
    }
}
```

Usage:

```java
DatabaseFacade db = new DatabaseFacade("jdbc:mysql://localhost/mydb");
List<String> users = db.findData("users", "age > 18");
db.close();
```

Clean and simple.

## When to Use Facade

Use Facade when:
- You need a simple interface to a complex subsystem
- There are many dependencies between clients and implementation classes
- You want to layer your subsystems
- You want to decouple client code from subsystems

Common scenarios:
- Complex libraries or frameworks
- Legacy systems with complicated APIs
- Multi-step workflows
- Systems with many interdependent components

## Facade vs Other Patterns

### Facade vs Adapter

**Facade:** Simplifies a complex system, may involve multiple classes
**Adapter:** Makes one interface compatible with another, typically one class

### Facade vs Mediator

**Facade:** One-way simplification, subsystems don't know about facade
**Mediator:** Two-way coordination, colleagues know about mediator

### Facade vs Proxy

**Facade:** Provides simplified interface, different interface than subsystem
**Proxy:** Provides same interface, controls access to a single object

## The Key Principle: Least Knowledge

Facade embodies the Principle of Least Knowledge (Law of Demeter): "Only talk to your immediate friends."

Bad (violates principle):
```java
// Client knows too much about internal structure
amplifier.getPreprocessor().getNoiseReducer().setLevel(5);
```

Good (uses facade):
```java
// Client only knows about facade
homeTheater.optimizeSound();
```

The less a client knows about the subsystem, the better.

## Common Mistakes

### Mistake 1: Facade Does Too Much

```java
// BAD: Facade has business logic
class OrderFacade {
    public String placeOrder(...) {
        // Complex business rules here
        // Discounts, loyalty points, promotions
        // This doesn't belong in a facade!
    }
}
```

Facades should coordinate, not contain business logic. Keep them thin.

### Mistake 2: Bypassing the Facade

```java
// BAD: Client still accesses subsystems directly
orderFacade.placeOrder(...);
inventory.reserveStock(...);  // Why have a facade if you bypass it?
```

If you have a facade, use it consistently. Don't mix facade calls with direct subsystem calls.

### Mistake 3: Facade Knows Implementation Details

```java
// BAD: Facade is tightly coupled
class HomeTheaterFacade {
    public void watchMovie(String movie) {
        // Facade shouldn't know projector uses HDMI 2.1
        projector.setHDMI21Mode();
        // Or that amplifier needs 5 seconds to warm up
        Thread.sleep(5000);
    }
}
```

Keep the facade at a higher level of abstraction.

## Additional Facades for Different Use Cases

A subsystem can have multiple facades for different clients:

```java
// Basic facade for home users
class SimpleHomeTheaterFacade {
    public void watchMovie(String movie) {
        // Simple setup
    }
}

// Advanced facade for power users
class AdvancedHomeTheaterFacade {
    public void watchMovie(String movie, AudioFormat format, 
                          Resolution resolution, boolean hdr) {
        // Full control
    }
}
```

Different abstractions for different needs.

## Facade and SOLID Principles

### Single Responsibility Principle

The facade's single responsibility is simplifying access to the subsystem.

### Open/Closed Principle

New features can be added to the facade without changing subsystem classes.

### Dependency Inversion Principle

Clients depend on the facade interface, not concrete subsystems.

### Interface Segregation Principle

The facade provides a client-specific interface instead of a general-purpose one.

## The Mental Model

Think of a facade like:

**A hotel concierge:** You don't book your own restaurant reservations, arrange transportation, and schedule tours separately. You tell the concierge what you want, and they coordinate everything.

**A car dashboard:** You don't manipulate the engine, transmission, and brakes directly. You use simple controls: steering wheel, pedals, gear shift. The car handles the complex orchestration.

**A light switch:** You flip a switch. You don't worry about the circuit breaker, wiring, or light fixture. Simple interface to a complex electrical system.

## Performance Considerations

Facades add minimal overhead. They're just method calls that delegate to subsystems.

The benefit of reduced coupling and simplified code far outweighs any tiny performance cost.

## Testing with Facades

Facades make testing easier:

```java
// Without facade: Mock 5 different objects
@Test
public void testOrderProcessing() {
    InventorySystem mockInventory = mock(InventorySystem.class);
    PaymentGateway mockPayment = mock(PaymentGateway.class);
    ShippingService mockShipping = mock(ShippingService.class);
    NotificationService mockNotification = mock(NotificationService.class);
    InvoiceSystem mockInvoice = mock(InvoiceSystem.class);
    // Complex setup...
}

// With facade: Mock one object
@Test
public void testOrderProcessing() {
    OrderFacade mockFacade = mock(OrderFacade.class);
    when(mockFacade.placeOrder(...)).thenReturn("Success");
    // Simple!
}
```

## Final Thoughts

The Facade pattern is one of the most practical patterns you'll use. Every time you create a simplified API over something complex, you're using Facade thinking.

It's not fancy. It's not clever. It's just good design: hide complexity, provide simplicity.

The key insights:
- Complex systems need simple interfaces
- Clients shouldn't know subsystem details
- Coordination logic belongs in one place
- Simplification doesn't mean limiting power

Remember: the goal isn't to restrict access to subsystems. Advanced users can still access them directly if needed. The facade is for everyone else who just wants to get stuff done.

Next time you're exposing a complex system to clients, ask yourself: "What's the simplest possible interface I can provide?" That's your facade.

Keep it simple.