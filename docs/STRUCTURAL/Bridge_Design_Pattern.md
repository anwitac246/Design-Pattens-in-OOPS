# The Bridge Pattern: Decoupling Abstraction from Implementation

Here's a problem you've probably faced: you're building a shape drawing application. You have circles and squares. You need to draw them in red and blue. Do you create RedCircle, BlueCircle, RedSquare, and BlueSquare classes?

What happens when you add triangles? And green? Suddenly you need nine classes. Add one more shape and one more color, and you're at sixteen classes. This is called the "combinatorial explosion" problem.

The Bridge pattern says: stop multiplying classes. Instead, separate the things that vary independently.

## The Problem: Class Explosion

Let's say you're building a remote control system. You have different types of remotes (basic, advanced) and different devices (TV, radio):

```java
// Inheritance approach - BAD
class BasicRemoteForTV extends TV {
    public void togglePower() { /* TV specific */ }
}

class BasicRemoteForRadio extends Radio {
    public void togglePower() { /* Radio specific */ }
}

class AdvancedRemoteForTV extends TV {
    public void togglePower() { /* TV specific */ }
    public void mute() { /* TV specific */ }
}

class AdvancedRemoteForRadio extends Radio {
    public void togglePower() { /* Radio specific */ }
    public void mute() { /* Radio specific */ }
}
```

Two remote types, two device types = four classes. Add streaming devices? Six classes. Add a universal remote? Nine classes. This doesn't scale.

The problem: you have two dimensions of variation (remote type and device type) forced into one hierarchy.

## The Bridge Solution

The Bridge pattern separates abstraction (remote controls) from implementation (devices). Each can vary independently.

```java
// Implementation interface
interface Device {
    void turnOn();
    void turnOff();
    void setVolume(int volume);
    int getVolume();
}

// Concrete implementations
class TV implements Device {
    private boolean on = false;
    private int volume = 50;
    
    public void turnOn() {
        on = true;
        System.out.println("TV is now ON");
    }
    
    public void turnOff() {
        on = false;
        System.out.println("TV is now OFF");
    }
    
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("TV volume set to " + volume);
    }
    
    public int getVolume() {
        return volume;
    }
}

class Radio implements Device {
    private boolean on = false;
    private int volume = 30;
    
    public void turnOn() {
        on = true;
        System.out.println("Radio is now ON");
    }
    
    public void turnOff() {
        on = false;
        System.out.println("Radio is now OFF");
    }
    
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("Radio volume set to " + volume);
    }
    
    public int getVolume() {
        return volume;
    }
}

// Abstraction
abstract class RemoteControl {
    protected Device device;
    
    public RemoteControl(Device device) {
        this.device = device;
    }
    
    public void togglePower() {
        // Implementation delegated to device
        System.out.println("Toggling power...");
    }
    
    public void volumeUp() {
        device.setVolume(device.getVolume() + 10);
    }
    
    public void volumeDown() {
        device.setVolume(device.getVolume() - 10);
    }
}

// Refined abstractions
class BasicRemote extends RemoteControl {
    public BasicRemote(Device device) {
        super(device);
    }
    
    public void togglePower() {
        System.out.println("BasicRemote: Toggling power");
        // Simple toggle logic
    }
}

class AdvancedRemote extends RemoteControl {
    public AdvancedRemote(Device device) {
        super(device);
    }
    
    public void togglePower() {
        System.out.println("AdvancedRemote: Toggling power with fancy animation");
    }
    
    public void mute() {
        System.out.println("AdvancedRemote: Muting device");
        device.setVolume(0);
    }
}
```

Usage:

```java
// Create devices
Device tv = new TV();
Device radio = new Radio();

// Create remotes
RemoteControl basicRemoteForTV = new BasicRemote(tv);
RemoteControl advancedRemoteForRadio = new AdvancedRemote(radio);

// Use them
basicRemoteForTV.togglePower();
basicRemoteForTV.volumeUp();

advancedRemoteForRadio.togglePower();
advancedRemoteForRadio.mute();

// Easy to switch devices
RemoteControl basicRemoteForRadio = new BasicRemote(radio);
```

Now we have two remote classes and two device classes. Add a new device? One class. Add a new remote type? One class. No explosion.

## The Components

### 1. Abstraction

Defines the high-level control interface and holds a reference to the implementation:

```java
abstract class RemoteControl {
    protected Device device;  // Bridge to implementation
    
    public RemoteControl(Device device) {
        this.device = device;
    }
}
```

### 2. Refined Abstraction

Extends the abstraction with additional features:

```java
class AdvancedRemote extends RemoteControl {
    public void mute() {
        device.setVolume(0);
    }
}
```

### 3. Implementation Interface

Defines the interface for implementation classes:

```java
interface Device {
    void turnOn();
    void turnOff();
    void setVolume(int volume);
}
```

### 4. Concrete Implementations

Implement the implementation interface:

```java
class TV implements Device {
    // TV-specific implementation
}

class Radio implements Device {
    // Radio-specific implementation
}
```

## Real-World Example: Drawing Shapes

You have shapes and rendering APIs. Both vary independently:

```java
// Implementation: Rendering API
interface Renderer {
    void renderCircle(float radius);
    void renderSquare(float side);
}

class VectorRenderer implements Renderer {
    public void renderCircle(float radius) {
        System.out.println("Drawing circle with radius " + radius + " using vectors");
    }
    
    public void renderSquare(float side) {
        System.out.println("Drawing square with side " + side + " using vectors");
    }
}

class RasterRenderer implements Renderer {
    public void renderCircle(float radius) {
        System.out.println("Drawing circle with radius " + radius + " as pixels");
    }
    
    public void renderSquare(float side) {
        System.out.println("Drawing square with side " + side + " as pixels");
    }
}

// Abstraction: Shape
abstract class Shape {
    protected Renderer renderer;
    
    public Shape(Renderer renderer) {
        this.renderer = renderer;
    }
    
    public abstract void draw();
    public abstract void resize(float factor);
}

// Refined abstractions
class Circle extends Shape {
    private float radius;
    
    public Circle(Renderer renderer, float radius) {
        super(renderer);
        this.radius = radius;
    }
    
    public void draw() {
        renderer.renderCircle(radius);
    }
    
    public void resize(float factor) {
        radius *= factor;
    }
}

class Square extends Shape {
    private float side;
    
    public Square(Renderer renderer, float side) {
        super(renderer);
        this.side = side;
    }
    
    public void draw() {
        renderer.renderSquare(side);
    }
    
    public void resize(float factor) {
        side *= factor;
    }
}
```

Usage:

```java
Renderer vector = new VectorRenderer();
Renderer raster = new RasterRenderer();

Shape circle = new Circle(vector, 5);
circle.draw();  // Drawing circle with radius 5.0 using vectors

Shape square = new Square(raster, 10);
square.draw();  // Drawing square with side 10.0 as pixels

// Easy to switch renderer
circle = new Circle(raster, 5);
circle.draw();  // Drawing circle with radius 5.0 as pixels
```

Want to add triangles? Create one `Triangle` class. Want to add OpenGL rendering? Create one `OpenGLRenderer` class. No combinatorial explosion.

## Another Example: Database Drivers

```java
// Implementation: Database driver
interface DatabaseDriver {
    void connect(String url);
    void executeQuery(String query);
    void disconnect();
}

class MySQLDriver implements DatabaseDriver {
    public void connect(String url) {
        System.out.println("Connecting to MySQL at " + url);
    }
    
    public void executeQuery(String query) {
        System.out.println("Executing MySQL query: " + query);
    }
    
    public void disconnect() {
        System.out.println("Disconnecting from MySQL");
    }
}

class PostgreSQLDriver implements DatabaseDriver {
    public void connect(String url) {
        System.out.println("Connecting to PostgreSQL at " + url);
    }
    
    public void executeQuery(String query) {
        System.out.println("Executing PostgreSQL query: " + query);
    }
    
    public void disconnect() {
        System.out.println("Disconnecting from PostgreSQL");
    }
}

// Abstraction: Database connection
abstract class DatabaseConnection {
    protected DatabaseDriver driver;
    
    public DatabaseConnection(DatabaseDriver driver) {
        this.driver = driver;
    }
    
    public abstract void open(String url);
    public abstract void close();
    public abstract void query(String sql);
}

// Refined abstraction: Connection with pooling
class PooledConnection extends DatabaseConnection {
    public PooledConnection(DatabaseDriver driver) {
        super(driver);
    }
    
    public void open(String url) {
        System.out.println("Opening pooled connection");
        driver.connect(url);
    }
    
    public void close() {
        System.out.println("Returning connection to pool");
        driver.disconnect();
    }
    
    public void query(String sql) {
        driver.executeQuery(sql);
    }
}

// Refined abstraction: Direct connection
class DirectConnection extends DatabaseConnection {
    public DirectConnection(DatabaseDriver driver) {
        super(driver);
    }
    
    public void open(String url) {
        System.out.println("Opening direct connection");
        driver.connect(url);
    }
    
    public void close() {
        System.out.println("Closing direct connection");
        driver.disconnect();
    }
    
    public void query(String sql) {
        driver.executeQuery(sql);
    }
}
```

## Bridge vs Adapter

These patterns look similar but serve different purposes:

**Adapter:** Makes incompatible interfaces work together. Applied after systems are designed.

**Bridge:** Prevents explosion of subclasses. Applied during design phase.

**Adapter:** Changes interface to match client expectations.

**Bridge:** Separates interface from implementation so both can vary independently.

Think of it this way: Adapter is reactive (fixing compatibility issues). Bridge is proactive (preventing design problems).

## Bridge vs Strategy

**Bridge:** About structure. Separates abstraction from implementation. Both sides are object hierarchies.

**Strategy:** About behavior. Encapsulates algorithms. Usually just one object with interchangeable strategies.

**Bridge:** Multiple dimensions of variation.

**Strategy:** One dimension (algorithm choice).

## When to Use Bridge

Use Bridge when:
- You want to avoid permanent binding between abstraction and implementation
- Both abstraction and implementation need to be extensible via subclassing
- Changes in implementation shouldn't affect clients
- You have proliferating classes due to multiple dimensions
- You want to share implementation among multiple objects

Real scenarios:
- GUI frameworks (abstraction: widgets, implementation: platform APIs)
- Database drivers (abstraction: connections, implementation: specific databases)
- Graphics rendering (abstraction: shapes, implementation: rendering APIs)
- Device drivers (abstraction: devices, implementation: platforms)
- Messaging systems (abstraction: message types, implementation: protocols)

## Common Mistakes

### Mistake 1: Using Bridge When Adapter Would Do

```java
// Don't use Bridge just to convert interfaces
// Use Adapter for that
```

Bridge is for preventing class explosion, not fixing incompatibilities.

### Mistake 2: Too Many Layers

```java
// BAD: Over-engineering with unnecessary abstraction layers
abstract class AbstractRemoteBase {
    abstract class RemoteControl {
        abstract class BasicRemote {
            // Too many layers!
        }
    }
}
```

Keep it simple. Two layers: abstraction and implementation.

### Mistake 3: Forgetting the Bridge

```java
// BAD: Abstraction not using implementation
class Circle extends Shape {
    public void draw() {
        // Drawing directly instead of using renderer
        System.out.println("Drawing circle");
    }
}
```

The abstraction must delegate to the implementation. That's the whole point.

## Bridge and SOLID Principles

### Single Responsibility Principle

Abstraction handles high-level logic. Implementation handles low-level details. Separate responsibilities.

### Open/Closed Principle

Add new abstractions or implementations without modifying existing code.

### Dependency Inversion Principle

Abstraction depends on implementation interface, not concrete implementations.

## The Mental Model

Think of Bridge like:

**TV and remote:** Remote (abstraction) works with any TV (implementation) that follows the interface. Remote doesn't care if it's a Sony or Samsung. TV doesn't care if it's a basic or universal remote.

**Computer and monitor:** Computer (abstraction) connects to any monitor (implementation) via a standard port. You can upgrade either independently.

**Driver and car:** Driver (abstraction) can drive different cars (implementations) as long as they have standard controls. Same driver, different car. Or different driver, same car.

The key: two things that vary independently, connected by a bridge (interface).

## Implementation Tips

### Tip 1: Start with Composition

```java
// Prefer this
class RemoteControl {
    private Device device;  // Composition
}

// Over this
class RemoteControl extends Device {  // Inheritance
}
```

Bridge uses composition, not inheritance, between abstraction and implementation.

### Tip 2: Keep Implementation Interface Focused

```java
// GOOD: Minimal interface
interface Device {
    void turnOn();
    void turnOff();
    void setVolume(int volume);
}

// BAD: Too many methods
interface Device {
    void turnOn();
    void turnOff();
    void setVolume(int volume);
    void setChannel(int channel);  // Not all devices have channels
    void setColor(Color color);     // Not all devices have color
}
```

Implementation interface should contain only operations common to all implementations.

### Tip 3: Make Abstraction Independent

```java
// Abstraction shouldn't know concrete implementation types
class RemoteControl {
    public RemoteControl(Device device) {  // Interface type
        this.device = device;
    }
    
    // NOT this:
    // public RemoteControl(TV tv) {  // Concrete type
}
```

## Performance Considerations

Bridge adds one level of indirection. Usually negligible overhead, but consider:

- One extra method call (JVM often inlines this)
- One extra object (the implementation)

The flexibility gained far outweighs the tiny performance cost.

## Testing with Bridge

Bridge makes testing easier:

```java
@Test
public void testRemoteControl() {
    // Mock the implementation
    Device mockDevice = mock(Device.class);
    RemoteControl remote = new BasicRemote(mockDevice);
    
    remote.volumeUp();
    
    // Verify behavior
    verify(mockDevice).setVolume(anyInt());
}

@Test
public void testDevice() {
    // Test device independently
    Device tv = new TV();
    tv.turnOn();
    // Verify TV-specific behavior
}
```

Abstraction and implementation are testable in isolation.

## Final Thoughts

The Bridge pattern is about separation. When you have two things that vary independently, don't mix them into one hierarchy. Separate them and connect them with a bridge.

It's not about being clever. It's about:
- Preventing class explosion
- Making code more maintainable
- Allowing independent variation
- Following SOLID principles

The key insight: inheritance is rigid. Composition is flexible. Bridge uses composition to connect two hierarchies, allowing each to evolve independently.

Remember:
- Identify dimensions that vary independently
- Separate abstraction from implementation
- Connect them via composition
- Both can now vary freely

Next time you're creating RedCircle, BlueCircle, RedSquare, and BlueSquare classes, stop. Think Bridge. Separate color from shape. Problem solved.

Build bridges, not towers.