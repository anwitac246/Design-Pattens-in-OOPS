# The Proxy Pattern: The Middleman That's Actually Useful

Ever dealt with a celebrity through their agent instead of directly? Or accessed a website through a VPN? Or loaded a preview image before the full-resolution version? That's the Proxy pattern in real life.

A proxy is a surrogate or placeholder for another object, controlling access to it.

## The Problem: Direct Access Isn't Always Best

Imagine you're building an image viewer. Images are large and stored on a remote server:

```java
class RealImage {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();  // Expensive operation
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + filename);
        // Simulate expensive load operation
        try {
            Thread.sleep(2000);  // Takes 2 seconds
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}
```

Usage:

```java
RealImage image1 = new RealImage("photo1.jpg");  // Loads immediately
RealImage image2 = new RealImage("photo2.jpg");  // Loads immediately
RealImage image3 = new RealImage("photo3.jpg");  // Loads immediately

// What if user only views photo1?
// We loaded all three for nothing!
```

Problems:

**Wasteful loading:** Loading images that might never be displayed.

**Slow startup:** User waits for all images to load.

**Memory waste:** All images in memory at once.

## The Proxy Solution

Create a proxy that loads the real image only when needed:

```java
// Subject interface
interface Image {
    void display();
}

// Real subject
class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + filename);
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

// Proxy
class ImageProxy implements Image {
    private String filename;
    private RealImage realImage;  // Lazy initialization
    
    public ImageProxy(String filename) {
        this.filename = filename;
        // Don't load yet!
    }
    
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);  // Load on first access
        }
        realImage.display();
    }
}
```

Usage:

```java
Image image1 = new ImageProxy("photo1.jpg");  // Instant, no loading
Image image2 = new ImageProxy("photo2.jpg");  // Instant, no loading
Image image3 = new ImageProxy("photo3.jpg");  // Instant, no loading

System.out.println("Proxies created. Images not loaded yet.");

image1.display();  // NOW photo1 loads (first access)
image1.display();  // Uses already-loaded image (fast)

// photo2 and photo3 never load if never displayed
```

The proxy delays expensive operations until they're actually needed. Lazy loading.

## Types of Proxies

There are several flavors of proxy, each serving a different purpose:

### 1. Virtual Proxy (Lazy Initialization)

What we just saw. Delays expensive object creation:

```java
class DatabaseConnectionProxy implements Connection {
    private Connection realConnection;
    
    public void query(String sql) {
        if (realConnection == null) {
            realConnection = new ExpensiveDatabaseConnection();
        }
        realConnection.query(sql);
    }
}
```

### 2. Protection Proxy (Access Control)

Controls access based on permissions:

```java
class Document {
    private String content;
    
    public void read() {
        System.out.println("Reading: " + content);
    }
    
    public void write(String text) {
        this.content = text;
        System.out.println("Writing: " + text);
    }
}

class ProtectedDocumentProxy {
    private Document document;
    private String userRole;
    
    public ProtectedDocumentProxy(Document doc, String role) {
        this.document = doc;
        this.userRole = role;
    }
    
    public void read() {
        // Anyone can read
        document.read();
    }
    
    public void write(String text) {
        if (userRole.equals("ADMIN") || userRole.equals("EDITOR")) {
            document.write(text);
        } else {
            System.out.println("Access denied: insufficient permissions");
        }
    }
}
```

Usage:

```java
Document doc = new Document();

ProtectedDocumentProxy adminProxy = new ProtectedDocumentProxy(doc, "ADMIN");
adminProxy.write("Admin content");  // Allowed

ProtectedDocumentProxy viewerProxy = new ProtectedDocumentProxy(doc, "VIEWER");
viewerProxy.write("Viewer content");  // Denied
viewerProxy.read();  // Allowed
```

### 3. Remote Proxy

Represents an object in a different address space (different server, process):

```java
// Client thinks it's calling local object
interface UserService {
    User getUser(int id);
}

class RemoteUserServiceProxy implements UserService {
    private String serverUrl;
    
    public RemoteUserServiceProxy(String url) {
        this.serverUrl = url;
    }
    
    public User getUser(int id) {
        System.out.println("Making HTTP request to " + serverUrl);
        // Make HTTP call to remote server
        // Parse response
        // Return User object
        return new User(id, "Remote User");
    }
}
```

Client code doesn't know it's calling a remote service. Looks like local object.

### 4. Caching Proxy

Caches results to avoid repeated expensive operations:

```java
class CachingDatabaseProxy implements Database {
    private Database realDatabase;
    private Map<String, Result> cache;
    
    public CachingDatabaseProxy(Database db) {
        this.realDatabase = db;
        this.cache = new HashMap<>();
    }
    
    public Result query(String sql) {
        if (cache.containsKey(sql)) {
            System.out.println("Returning cached result");
            return cache.get(sql);
        }
        
        System.out.println("Executing query on database");
        Result result = realDatabase.query(sql);
        cache.put(sql, result);
        return result;
    }
    
    public void clearCache() {
        cache.clear();
    }
}
```

### 5. Logging Proxy

Logs method calls:

```java
class LoggingServiceProxy implements Service {
    private Service realService;
    
    public LoggingServiceProxy(Service service) {
        this.realService = service;
    }
    
    public void execute(String command) {
        System.out.println("LOG: Executing command: " + command);
        long start = System.currentTimeMillis();
        
        realService.execute(command);
        
        long duration = System.currentTimeMillis() - start;
        System.out.println("LOG: Command completed in " + duration + "ms");
    }
}
```

### 6. Smart Proxy

Adds additional behavior:

```java
class SmartReferenceProxy implements Resource {
    private Resource realResource;
    private int referenceCount = 0;
    
    public void use() {
        if (realResource == null) {
            realResource = new ExpensiveResource();
        }
        referenceCount++;
        realResource.use();
    }
    
    public void release() {
        referenceCount--;
        if (referenceCount == 0) {
            // Clean up resource when no longer needed
            realResource.dispose();
            realResource = null;
        }
    }
}
```

## Real-World Example: Credit Card Proxy

```java
// Subject interface
interface PaymentMethod {
    void pay(double amount);
}

// Real subject
class CreditCard implements PaymentMethod {
    private String cardNumber;
    private double balance;
    
    public CreditCard(String cardNumber, double balance) {
        this.cardNumber = cardNumber;
        this.balance = balance;
    }
    
    public void pay(double amount) {
        if (balance >= amount) {
            balance -= amount;
            System.out.println("Paid $" + amount + " with card " + maskCard());
        } else {
            System.out.println("Insufficient funds");
        }
    }
    
    private String maskCard() {
        return "****" + cardNumber.substring(cardNumber.length() - 4);
    }
}

// Protection proxy with fraud detection
class SecureCreditCardProxy implements PaymentMethod {
    private CreditCard realCard;
    private int failedAttempts = 0;
    private static final int MAX_FAILED_ATTEMPTS = 3;
    private static final double MAX_TRANSACTION = 10000;
    
    public SecureCreditCardProxy(CreditCard card) {
        this.realCard = card;
    }
    
    public void pay(double amount) {
        if (failedAttempts >= MAX_FAILED_ATTEMPTS) {
            System.out.println("Card blocked due to suspicious activity");
            return;
        }
        
        if (amount > MAX_TRANSACTION) {
            System.out.println("Transaction exceeds limit. Requires verification.");
            failedAttempts++;
            return;
        }
        
        // Additional fraud checks here
        if (detectFraud(amount)) {
            System.out.println("Fraudulent transaction detected");
            failedAttempts++;
            return;
        }
        
        realCard.pay(amount);
        failedAttempts = 0;  // Reset on successful payment
    }
    
    private boolean detectFraud(double amount) {
        // Simplified fraud detection
        return amount > 5000;
    }
}
```

Usage:

```java
CreditCard card = new CreditCard("1234567812345678", 10000);
PaymentMethod secureCard = new SecureCreditCardProxy(card);

secureCard.pay(100);    // Success
secureCard.pay(6000);   // Fraud detected
secureCard.pay(15000);  // Exceeds limit
```

## Dynamic Proxy (Java Reflection)

Java provides built-in support for creating proxies dynamically:

```java
interface Calculator {
    int add(int a, int b);
    int multiply(int a, int b);
}

class CalculatorImpl implements Calculator {
    public int add(int a, int b) {
        return a + b;
    }
    
    public int multiply(int a, int b) {
        return a * b;
    }
}

// Create dynamic proxy with logging
Calculator calculator = new CalculatorImpl();

Calculator proxy = (Calculator) Proxy.newProxyInstance(
    Calculator.class.getClassLoader(),
    new Class[] { Calculator.class },
    (proxy, method, args) -> {
        System.out.println("Calling method: " + method.getName());
        Object result = method.invoke(calculator, args);
        System.out.println("Result: " + result);
        return result;
    }
);

proxy.add(5, 3);       // Logs method call and result
proxy.multiply(4, 7);  // Logs method call and result
```

Dynamic proxies are powerful for cross-cutting concerns like logging, security, transactions.

## When to Use Proxy

Use Proxy when:
- You need lazy initialization (virtual proxy)
- You need access control (protection proxy)
- You're communicating with remote objects (remote proxy)
- You want to cache results (caching proxy)
- You want to log/monitor operations (logging proxy)
- You need to manage object lifecycle (smart proxy)

Real scenarios:
- Image loading in applications
- Database connection pooling
- Remote method invocation (RMI)
- Security and authentication
- Caching and memoization
- Logging and monitoring
- Lazy loading of expensive resources

## Proxy vs Adapter vs Decorator

These patterns look similar but have different intents:

**Proxy:** Same interface as real object. Controls access. Lazy loading, security, caching.

**Adapter:** Different interface. Makes incompatible interfaces work together.

**Decorator:** Same interface. Adds new behavior. Focuses on enhancement, not control.

Example:
- **Proxy:** `ImageProxy` controls when `RealImage` loads
- **Adapter:** `ImageAdapter` makes `LegacyImageLibrary` work with new `Image` interface  
- **Decorator:** `BorderDecorator` adds a border to an image

## Common Mistakes

### Mistake 1: Proxy Doing Too Much

```java
// BAD: Proxy contains business logic
class ImageProxy implements Image {
    public void display() {
        // Load image
        // Resize image - business logic!
        // Apply filters - business logic!
        // Display image
    }
}
```

Proxies should control access, not add business logic. That's Decorator's job.

### Mistake 2: Breaking Liskov Substitution

```java
// BAD: Proxy has different behavior
class SlowImageProxy implements Image {
    public void display() {
        Thread.sleep(5000);  // Artificial delay
        realImage.display();
    }
}
```

Proxy should be substitutable for the real object. Don't change semantics.

### Mistake 3: Not Thread-Safe Lazy Initialization

```java
// BAD: Race condition
class ImageProxy {
    private RealImage image;
    
    public void display() {
        if (image == null) {  // Thread 1 and 2 both see null
            image = new RealImage();  // Both create images!
        }
        image.display();
    }
}
```

Use proper synchronization:

```java
class ImageProxy {
    private volatile RealImage image;
    
    public void display() {
        if (image == null) {
            synchronized(this) {
                if (image == null) {  // Double-checked locking
                    image = new RealImage();
                }
            }
        }
        image.display();
    }
}
```

## Proxy and SOLID Principles

### Single Responsibility Principle

Proxy handles access control. Real object handles business logic.

### Open/Closed Principle

Add new proxies (logging, caching) without modifying real object.

### Liskov Substitution Principle

Proxy can substitute real object anywhere.

### Dependency Inversion Principle

Client depends on interface, not concrete proxy or real object.

## The Mental Model

Think of Proxy like:

**A lawyer or agent:** You don't talk to the celebrity directly. You talk to their agent, who controls access and handles logistics.

**A receptionist:** You don't walk into the CEO's office. You go through the receptionist, who checks if you have an appointment and manages access.

**A bank ATM:** You don't go into the vault. You use an ATM (proxy) that controls access to your money with security checks.

## Performance Considerations

Proxies add overhead:
- Extra object
- Extra method call
- Possible synchronization cost

But they often improve performance through:
- Lazy loading (don't create until needed)
- Caching (avoid repeated expensive operations)
- Connection pooling (reuse expensive resources)

Profile to see if proxy helps or hurts in your specific case.

## Testing with Proxy

Proxies make testing easier:

```java
@Test
public void testVirtualProxy() {
    // Test that image isn't loaded until display
    ImageProxy proxy = new ImageProxy("test.jpg");
    // Verify no loading happened
    
    proxy.display();
    // Verify loading happened
}

@Test
public void testProtectionProxy() {
    Document doc = mock(Document.class);
    ProtectedDocumentProxy proxy = new ProtectedDocumentProxy(doc, "VIEWER");
    
    proxy.write("test");
    
    // Verify real document's write was never called
    verify(doc, never()).write(anyString());
}

@Test
public void testCachingProxy() {
    Database mockDb = mock(Database.class);
    when(mockDb.query(anyString())).thenReturn(new Result());
    
    CachingDatabaseProxy proxy = new CachingDatabaseProxy(mockDb);
    
    proxy.query("SELECT * FROM users");
    proxy.query("SELECT * FROM users");  // Same query
    
    // Verify database was only called once
    verify(mockDb, times(1)).query(anyString());
}
```

## Real-World Usage

Proxies are everywhere:

**Spring Framework:** Uses dynamic proxies for AOP (aspect-oriented programming), transactions, security.

**Hibernate:** Uses proxies for lazy loading of entities.

**Web Frameworks:** Proxy patterns for remote API calls.

**Security:** Protection proxies for access control.

## Final Thoughts

The Proxy pattern is about control. It sits between the client and the real object, managing access, adding security, enabling lazy loading, or providing caching.

It's not about complexity. It's about:
- Controlling when objects are created
- Adding security without changing core logic  
- Optimizing performance through caching
- Managing remote or expensive resources

The key insight: sometimes you don't want direct access to an object. A middleman (proxy) can provide value through control, optimization, or security.

Remember:
- Proxy has same interface as real object
- Controls access, doesn't change behavior
- Many types: virtual, protection, remote, caching
- Common in frameworks and libraries

Next time you're about to create expensive objects upfront, or you need to add access control, or you want to cache results, think Proxy. Add a layer of control.

Stand in the gap.