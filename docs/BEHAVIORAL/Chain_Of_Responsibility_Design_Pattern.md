# The Chain of Responsibility Pattern: Pass the Buck, Intelligently

Ever called customer support and been transferred from person to person until someone could actually help you? Annoying for you, but it's actually a smart system design. That's the Chain of Responsibility pattern.

The idea: multiple objects get a chance to handle a request. Each decides whether to handle it or pass it along the chain.

## The Problem: Hardcoded Request Handling

You're building a support ticket system. Different types of issues need different handlers:

```java
class SupportSystem {
    public void handleRequest(String issueType, String description) {
        if (issueType.equals("TECHNICAL")) {
            System.out.println("Technical team handling: " + description);
            // Technical handling logic
        } else if (issueType.equals("BILLING")) {
            System.out.println("Billing team handling: " + description);
            // Billing handling logic
        } else if (issueType.equals("GENERAL")) {
            System.out.println("General support handling: " + description);
            // General handling logic
        } else {
            System.out.println("No handler found");
        }
    }
}
```

Problems:

**Rigid coupling:** One class knows about all handlers.

**Hard to extend:** Add a new handler? Modify this class.

**No flexibility:** Can't dynamically configure the chain.

**Violates Open/Closed:** Every new issue type requires code changes.

What if handlers could be chained together, each deciding if they can handle the request?

## The Chain of Responsibility Solution

Create a chain of handler objects. Each tries to handle the request or passes it to the next:

```java
// Handler interface
abstract class SupportHandler {
    protected SupportHandler nextHandler;
    
    public void setNext(SupportHandler handler) {
        this.nextHandler = handler;
    }
    
    public void handleRequest(String issueType, String description) {
        if (canHandle(issueType)) {
            handle(description);
        } else if (nextHandler != null) {
            nextHandler.handleRequest(issueType, description);
        } else {
            System.out.println("No handler available for: " + issueType);
        }
    }
    
    protected abstract boolean canHandle(String issueType);
    protected abstract void handle(String description);
}

// Concrete handlers
class TechnicalSupportHandler extends SupportHandler {
    
    @Override
    protected boolean canHandle(String issueType) {
        return issueType.equals("TECHNICAL");
    }
    
    @Override
    protected void handle(String description) {
        System.out.println("Technical Support: Handling " + description);
    }
}

class BillingSupportHandler extends SupportHandler {
    
    @Override
    protected boolean canHandle(String issueType) {
        return issueType.equals("BILLING");
    }
    
    @Override
    protected void handle(String description) {
        System.out.println("Billing Support: Handling " + description);
    }
}

class GeneralSupportHandler extends SupportHandler {
    
    @Override
    protected boolean canHandle(String issueType) {
        return issueType.equals("GENERAL");
    }
    
    @Override
    protected void handle(String description) {
        System.out.println("General Support: Handling " + description);
    }
}
```

Usage:

```java
// Build the chain
SupportHandler technical = new TechnicalSupportHandler();
SupportHandler billing = new BillingSupportHandler();
SupportHandler general = new GeneralSupportHandler();

technical.setNext(billing);
billing.setNext(general);

// Send requests through the chain
technical.handleRequest("TECHNICAL", "App crashes on startup");
// Technical Support handles it

technical.handleRequest("BILLING", "Wrong charge on invoice");
// Passes to Billing, Billing handles it

technical.handleRequest("GENERAL", "How do I reset password?");
// Passes through Technical and Billing, General handles it

technical.handleRequest("UNKNOWN", "Something weird");
// Passes through entire chain, no handler found
```

Each handler checks if it can handle the request. If not, it passes to the next in the chain.

## The Components

### 1. Handler Interface

Defines request handling interface and maintains reference to next handler:

```java
abstract class Handler {
    protected Handler nextHandler;
    
    public void setNext(Handler handler) {
        this.nextHandler = handler;
    }
    
    public abstract void handleRequest(Request request);
}
```

### 2. Concrete Handlers

Implement actual handling logic:

```java
class ConcreteHandler extends Handler {
    public void handleRequest(Request request) {
        if (canHandle(request)) {
            // Handle it
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }
}
```

### 3. Client

Initiates the request:

```java
Handler chain = buildChain();
chain.handleRequest(new Request());
```

## Real-World Example: Expense Approval System

Different approval levels based on amount:

```java
abstract class ExpenseApprover {
    protected ExpenseApprover nextApprover;
    
    public void setNext(ExpenseApprover approver) {
        this.nextApprover = approver;
    }
    
    public void approveExpense(double amount, String purpose) {
        if (canApprove(amount)) {
            System.out.println(getClass().getSimpleName() + " approved $" + 
                             amount + " for " + purpose);
        } else if (nextApprover != null) {
            System.out.println(getClass().getSimpleName() + 
                             " can't approve. Passing to next level...");
            nextApprover.approveExpense(amount, purpose);
        } else {
            System.out.println("Expense requires board approval");
        }
    }
    
    protected abstract boolean canApprove(double amount);
}

class TeamLead extends ExpenseApprover {
    private static final double LIMIT = 1000;
    
    @Override
    protected boolean canApprove(double amount) {
        return amount <= LIMIT;
    }
}

class Manager extends ExpenseApprover {
    private static final double LIMIT = 5000;
    
    @Override
    protected boolean canApprove(double amount) {
        return amount <= LIMIT;
    }
}

class Director extends ExpenseApprover {
    private static final double LIMIT = 20000;
    
    @Override
    protected boolean canApprove(double amount) {
        return amount <= LIMIT;
    }
}

class CEO extends ExpenseApprover {
    private static final double LIMIT = 100000;
    
    @Override
    protected boolean canApprove(double amount) {
        return amount <= LIMIT;
    }
}
```

Usage:

```java
// Build approval chain
ExpenseApprover teamLead = new TeamLead();
ExpenseApprover manager = new Manager();
ExpenseApprover director = new Director();
ExpenseApprover ceo = new CEO();

teamLead.setNext(manager);
manager.setNext(director);
director.setNext(ceo);

// Submit expenses
teamLead.approveExpense(500, "Office supplies");
// TeamLead approved

teamLead.approveExpense(3000, "New laptops");
// TeamLead passes to Manager, Manager approved

teamLead.approveExpense(50000, "Server infrastructure");
// Passes through TeamLead, Manager, Director, CEO approved

teamLead.approveExpense(150000, "Office renovation");
// Passes through entire chain, requires board approval
```

## Another Example: Logging with Severity Levels

```java
abstract class Logger {
    public static final int INFO = 1;
    public static final int DEBUG = 2;
    public static final int WARNING = 3;
    public static final int ERROR = 4;
    
    protected int level;
    protected Logger nextLogger;
    
    public void setNext(Logger nextLogger) {
        this.nextLogger = nextLogger;
    }
    
    public void logMessage(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.logMessage(level, message);
        }
    }
    
    protected abstract void write(String message);
}

class ConsoleLogger extends Logger {
    public ConsoleLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("Console: " + message);
    }
}

class FileLogger extends Logger {
    public FileLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("File: " + message);
    }
}

class EmailLogger extends Logger {
    public EmailLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("Email: " + message);
    }
}
```

Usage:

```java
// Build logger chain
Logger consoleLogger = new ConsoleLogger(Logger.INFO);
Logger fileLogger = new FileLogger(Logger.DEBUG);
Logger emailLogger = new EmailLogger(Logger.ERROR);

consoleLogger.setNext(fileLogger);
fileLogger.setNext(emailLogger);

// Log messages
consoleLogger.logMessage(Logger.INFO, "This is an info message");
// Console logs it

consoleLogger.logMessage(Logger.DEBUG, "This is a debug message");
// Console and File log it

consoleLogger.logMessage(Logger.ERROR, "This is an error message");
// All three log it
```

Note: this logger doesn't stop after the first handler. Multiple handlers can process the same request.

## Two Variations

### 1. Stop After First Handler (Classic)

Request handled by first capable handler:

```java
public void handle(Request request) {
    if (canHandle(request)) {
        process(request);
        // Stop here
    } else if (nextHandler != null) {
        nextHandler.handle(request);
    }
}
```

Use when: Only one handler should process the request (expense approval, support tickets).

### 2. All Capable Handlers Process (Pipeline)

Request processed by all matching handlers:

```java
public void handle(Request request) {
    if (canHandle(request)) {
        process(request);
    }
    if (nextHandler != null) {
        nextHandler.handle(request);  // Continue regardless
    }
}
```

Use when: Multiple handlers might need to process the same request (logging, event handlers, middleware).

## Chain of Responsibility in Web Frameworks

This pattern is everywhere in web development:

### Servlet Filters (Java)

```java
public class AuthenticationFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) {
        if (isAuthenticated(request)) {
            chain.doFilter(request, response);  // Pass to next filter
        } else {
            response.sendError(401);  // Stop chain
        }
    }
}

public class LoggingFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) {
        logRequest(request);
        chain.doFilter(request, response);  // Always pass along
        logResponse(response);
    }
}
```

### Express.js Middleware

```javascript
app.use(authMiddleware);  // Check authentication
app.use(loggingMiddleware);  // Log requests
app.use(errorMiddleware);  // Handle errors
```

Each middleware can process the request, modify it, and pass it along or stop the chain.

## Building the Chain

### Method 1: Manual Setup

```java
Handler h1 = new Handler1();
Handler h2 = new Handler2();
Handler h3 = new Handler3();

h1.setNext(h2);
h2.setNext(h3);
```

### Method 2: Builder Pattern

```java
class ChainBuilder {
    private Handler first;
    private Handler last;
    
    public ChainBuilder add(Handler handler) {
        if (first == null) {
            first = last = handler;
        } else {
            last.setNext(handler);
            last = handler;
        }
        return this;
    }
    
    public Handler build() {
        return first;
    }
}

// Usage
Handler chain = new ChainBuilder()
    .add(new Handler1())
    .add(new Handler2())
    .add(new Handler3())
    .build();
```

### Method 3: Configuration-Based

```java
// Read from config file or database
List<String> handlerTypes = config.getHandlers();
Handler chain = null;
Handler last = null;

for (String type : handlerTypes) {
    Handler handler = HandlerFactory.create(type);
    if (chain == null) {
        chain = last = handler;
    } else {
        last.setNext(handler);
        last = handler;
    }
}
```

## When to Use Chain of Responsibility

Use Chain of Responsibility when:
- Multiple objects might handle a request
- You don't know which object should handle it in advance
- The set of handlers should be specified dynamically
- You want to avoid coupling sender to receiver

Real scenarios:
- Event handling systems
- Middleware in web frameworks
- Approval workflows
- Logging systems
- Error handling
- Request filtering
- Validation pipelines
- Permission checking

## Common Pitfalls

### Pitfall 1: Broken Chain

```java
// BAD: Forgot to set next handler
Handler h1 = new Handler1();
Handler h2 = new Handler2();
// Missing: h1.setNext(h2);

h1.handle(request);  // h2 never gets a chance!
```

Always ensure the chain is properly connected.

### Pitfall 2: No Handler Available

```java
// BAD: No fallback when chain ends
public void handle(Request request) {
    if (canHandle(request)) {
        process(request);
    } else if (nextHandler != null) {
        nextHandler.handle(request);
    }
    // What if nextHandler is null and we can't handle it?
    // Request silently ignored!
}
```

Add a default handler at the end:

```java
class DefaultHandler extends Handler {
    public void handle(Request request) {
        System.out.println("No specific handler found, using default");
    }
}
```

### Pitfall 3: Infinite Loops

```java
// BAD: Circular reference
Handler h1 = new Handler1();
Handler h2 = new Handler2();

h1.setNext(h2);
h2.setNext(h1);  // Circular!

h1.handle(request);  // Infinite loop!
```

Be careful when dynamically building chains.

## Chain of Responsibility vs Command

**Chain of Responsibility:** Multiple handlers, one might handle the request. Focus on who handles it.

**Command:** Single handler per command. Focus on encapsulating requests as objects.

**CoR:** Request travels through chain looking for handler.

**Command:** Request goes directly to appropriate handler (command object).

## Chain of Responsibility and SOLID

### Single Responsibility Principle

Each handler has one job: decide if it can handle a specific type of request.

### Open/Closed Principle

Add new handlers without modifying existing ones. Just add to the chain.

### Liskov Substitution Principle

All handlers implement the same interface. Interchangeable.

### Dependency Inversion Principle

Client depends on handler interface, not concrete handlers.

## The Mental Model

Think of Chain of Responsibility like:

**Customer support escalation:** Start with level 1 support. If they can't help, escalate to level 2. If they can't help, escalate to level 3. Someone eventually helps or you hit the CEO.

**Security checkpoint:** Bag scan → metal detector → ID check → boarding. Each station checks specific things. Pass all checks to board.

**Mail routing:** Local post office → regional center → national hub → destination regional center → local office → delivery. Each step routes the mail closer to destination.

## Performance Considerations

Chain adds overhead:
- Must traverse handlers until one accepts
- Longer chains = more method calls
- Could be slow if chain is deep

Optimizations:
- Keep chains short
- Put most likely handlers first
- Cache handler decisions if applicable
- Consider using a registry/map for direct lookup instead of chaining

## Testing with Chain of Responsibility

```java
@Test
public void testHandlerHandlesRequest() {
    Handler handler = new ConcreteHandler();
    Request request = new Request("type1");
    
    handler.handle(request);
    
    // Verify request was handled
}

@Test
public void testChainPassesRequest() {
    Handler h1 = new Handler1();  // Can't handle
    Handler h2 = new Handler2();  // Can handle
    h1.setNext(h2);
    
    h1.handle(new Request());
    
    // Verify h2 was called
}

@Test
public void testNoHandlerAvailable() {
    Handler h1 = new Handler1();
    
    h1.handle(new Request("unknown"));
    
    // Verify appropriate error handling
}
```

## Final Thoughts

The Chain of Responsibility pattern is about decoupling senders from receivers. Instead of the sender knowing exactly who should handle a request, it sends it into a chain and lets the handlers figure it out.

It's not about complexity. It's about:
- Flexibility in request handling
- Easy addition of new handlers
- Dynamic handler configuration
- Loose coupling between sender and receiver

The key insight: you don't always know who should handle a request. Let potential handlers decide for themselves.

Remember:
- Build the chain properly
- Decide if handlers should stop or continue
- Provide a fallback/default handler
- Keep chains reasonable length

Next time you're writing a long if-else chain to route requests to different handlers, stop. Think Chain of Responsibility. Extract handlers, chain them together, and let them self-select.

Pass it on.