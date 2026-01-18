# The Mediator Pattern: The Middleman Who Actually Simplifies Things

Ever been in a group chat where everyone's talking to everyone? It gets messy fast. Now imagine a moderator who coordinates all communication. Much cleaner, right?

That's the Mediator pattern. Instead of objects communicating directly with each other, they communicate through a mediator that coordinates the interactions.

## The Problem: Objects Talking to Everyone

You're building a chat application. Users can send messages to each other:

```java
class User {
    private String name;
    private List<User> contacts;
    
    public User(String name) {
        this.name = name;
        this.contacts = new ArrayList<>();
    }
    
    public void addContact(User user) {
        contacts.add(user);
    }
    
    public void sendMessage(String message, User recipient) {
        System.out.println(name + " sends to " + recipient.name + ": " + message);
        recipient.receiveMessage(message, this);
    }
    
    public void receiveMessage(String message, User sender) {
        System.out.println(name + " received from " + sender.name + ": " + message);
    }
}
```

Usage:

```java
User alice = new User("Alice");
User bob = new User("Bob");
User charlie = new User("Charlie");

alice.addContact(bob);
alice.addContact(charlie);
bob.addContact(alice);
bob.addContact(charlie);
charlie.addContact(alice);
charlie.addContact(bob);

alice.sendMessage("Hi Bob", bob);
bob.sendMessage("Hi Alice", alice);
```

Problems:

**Tight coupling:** Every user knows about every other user. Add a user? Update everyone's contact list.

**Complex relationships:** With 10 users, you have 90 potential connections (n*(n-1)). With 100 users? 9,900 connections.

**Hard to maintain:** Want to add features like message history, filtering, or logging? Update every User class.

**Can't easily change communication logic:** Want to add spam filtering or rate limiting? Modify every send/receive interaction.

## The Mediator Solution

Create a mediator that coordinates all communication:

```java
// Mediator interface
interface ChatMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
}

// Concrete mediator
class ChatRoom implements ChatMediator {
    private List<User> users;
    
    public ChatRoom() {
        this.users = new ArrayList<>();
    }
    
    @Override
    public void addUser(User user) {
        this.users.add(user);
    }
    
    @Override
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            // Don't send to yourself
            if (user != sender) {
                user.receive(message, sender);
            }
        }
    }
}

// Colleague
class User {
    private String name;
    private ChatMediator mediator;
    
    public User(String name, ChatMediator mediator) {
        this.name = name;
        this.mediator = mediator;
    }
    
    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);
    }
    
    public void receive(String message, User sender) {
        System.out.println(name + " received from " + sender.name + ": " + message);
    }
    
    public String getName() {
        return name;
    }
}
```

Usage:

```java
ChatRoom chatRoom = new ChatRoom();

User alice = new User("Alice", chatRoom);
User bob = new User("Bob", chatRoom);
User charlie = new User("Charlie", chatRoom);

chatRoom.addUser(alice);
chatRoom.addUser(bob);
chatRoom.addUser(charlie);

alice.send("Hi everyone!");
// Bob and Charlie receive the message

bob.send("Hello Alice!");
// Alice and Charlie receive the message
```

Now users don't know about each other. They only know about the chat room. The chat room handles all coordination.

## The Components

### 1. Mediator Interface

Defines communication interface:

```java
interface Mediator {
    void notify(Component sender, String event);
}
```

### 2. Concrete Mediator

Implements coordination logic:

```java
class ConcreteMediator implements Mediator {
    private ComponentA a;
    private ComponentB b;
    
    public void notify(Component sender, String event) {
        if (event.equals("A")) {
            // Coordinate with B
            b.doSomething();
        }
    }
}
```

### 3. Colleague Classes

Components that communicate through mediator:

```java
class Component {
    protected Mediator mediator;
    
    public Component(Mediator mediator) {
        this.mediator = mediator;
    }
    
    public void doAction() {
        mediator.notify(this, "action");
    }
}
```

## Real-World Example: GUI Form Validation

Complex form with interdependent fields:

```java
// Mediator
interface FormMediator {
    void componentChanged(Component component);
}

// Colleague base class
abstract class Component {
    protected FormMediator mediator;
    protected String value;
    
    public Component(FormMediator mediator) {
        this.mediator = mediator;
    }
    
    public void setValue(String value) {
        this.value = value;
        mediator.componentChanged(this);
    }
    
    public String getValue() {
        return value;
    }
}

// Concrete colleagues
class TextBox extends Component {
    private String label;
    
    public TextBox(FormMediator mediator, String label) {
        super(mediator);
        this.label = label;
    }
    
    public String getLabel() {
        return label;
    }
}

class CheckBox extends Component {
    private String label;
    
    public CheckBox(FormMediator mediator, String label) {
        super(mediator);
        this.label = label;
    }
    
    public boolean isChecked() {
        return "true".equals(value);
    }
}

class Button extends Component {
    private boolean enabled = true;
    
    public Button(FormMediator mediator) {
        super(mediator);
    }
    
    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }
    
    public boolean isEnabled() {
        return enabled;
    }
}

// Concrete mediator with validation logic
class RegistrationFormMediator implements FormMediator {
    private TextBox username;
    private TextBox password;
    private TextBox confirmPassword;
    private CheckBox termsCheckbox;
    private Button submitButton;
    
    public RegistrationFormMediator() {
        username = new TextBox(this, "Username");
        password = new TextBox(this, "Password");
        confirmPassword = new TextBox(this, "Confirm Password");
        termsCheckbox = new CheckBox(this, "I agree to terms");
        submitButton = new Button(this);
    }
    
    @Override
    public void componentChanged(Component component) {
        // Validate and update form state
        validateForm();
    }
    
    private void validateForm() {
        boolean usernameValid = username.getValue() != null && 
                               username.getValue().length() >= 3;
        boolean passwordValid = password.getValue() != null && 
                               password.getValue().length() >= 8;
        boolean passwordsMatch = password.getValue() != null && 
                                password.getValue().equals(confirmPassword.getValue());
        boolean termsAccepted = termsCheckbox.isChecked();
        
        // Enable submit button only if all conditions met
        boolean formValid = usernameValid && passwordValid && 
                           passwordsMatch && termsAccepted;
        submitButton.setEnabled(formValid);
        
        System.out.println("Form valid: " + formValid);
    }
    
    // Getters for form components
    public TextBox getUsername() { return username; }
    public TextBox getPassword() { return password; }
    public TextBox getConfirmPassword() { return confirmPassword; }
    public CheckBox getTermsCheckbox() { return termsCheckbox; }
    public Button getSubmitButton() { return submitButton; }
}
```

Usage:

```java
RegistrationFormMediator form = new RegistrationFormMediator();

// Initially, submit button is disabled
System.out.println("Submit enabled: " + form.getSubmitButton().isEnabled());

// Fill in form
form.getUsername().setValue("alice123");
form.getPassword().setValue("securepass123");
form.getConfirmPassword().setValue("securepass123");
form.getTermsCheckbox().setValue("true");

// Now submit button is enabled
System.out.println("Submit enabled: " + form.getSubmitButton().isEnabled());
```

The mediator coordinates validation logic. Components don't need to know about each other.

## Another Example: Air Traffic Control

```java
// Mediator
interface AirTrafficControl {
    void requestLanding(Airplane airplane);
    void requestTakeoff(Airplane airplane);
    void notifyPositionChange(Airplane airplane);
}

// Colleague
class Airplane {
    private String id;
    private AirTrafficControl atc;
    private String status;
    
    public Airplane(String id, AirTrafficControl atc) {
        this.id = id;
        this.atc = atc;
        this.status = "in air";
    }
    
    public void requestLanding() {
        System.out.println(id + " requesting landing");
        atc.requestLanding(this);
    }
    
    public void requestTakeoff() {
        System.out.println(id + " requesting takeoff");
        atc.requestTakeoff(this);
    }
    
    public void land() {
        status = "landed";
        System.out.println(id + " has landed");
    }
    
    public void takeoff() {
        status = "in air";
        System.out.println(id + " has taken off");
        atc.notifyPositionChange(this);
    }
    
    public String getId() { return id; }
    public String getStatus() { return status; }
}

// Concrete mediator
class Tower implements AirTrafficControl {
    private List<Airplane> airplanes;
    private boolean runwayFree = true;
    
    public Tower() {
        this.airplanes = new ArrayList<>();
    }
    
    public void registerAirplane(Airplane airplane) {
        airplanes.add(airplane);
    }
    
    @Override
    public void requestLanding(Airplane airplane) {
        if (runwayFree) {
            System.out.println("Tower: Cleared for landing, " + airplane.getId());
            runwayFree = false;
            airplane.land();
            runwayFree = true;
        } else {
            System.out.println("Tower: Hold position, " + airplane.getId() + 
                             ", runway occupied");
        }
    }
    
    @Override
    public void requestTakeoff(Airplane airplane) {
        if (runwayFree) {
            System.out.println("Tower: Cleared for takeoff, " + airplane.getId());
            runwayFree = false;
            airplane.takeoff();
            runwayFree = true;
        } else {
            System.out.println("Tower: Hold position, " + airplane.getId() + 
                             ", runway occupied");
        }
    }
    
    @Override
    public void notifyPositionChange(Airplane airplane) {
        System.out.println("Tower: Tracking " + airplane.getId());
    }
}
```

Usage:

```java
Tower tower = new Tower();

Airplane flight1 = new Airplane("Flight-001", tower);
Airplane flight2 = new Airplane("Flight-002", tower);

tower.registerAirplane(flight1);
tower.registerAirplane(flight2);

flight1.requestLanding();  // Cleared
flight2.requestLanding();  // Must wait
```

The tower (mediator) coordinates all airplane interactions. Airplanes don't communicate directly with each other.

## Mediator in Real Frameworks

### Java Swing Event Handling

```java
JButton button = new JButton("Click me");
JTextField textField = new JTextField();

// ActionListener acts as mediator
button.addActionListener(e -> {
    textField.setText("Button clicked");
});
```

### Spring Framework

```java
@Component
class OrderMediator {
    @Autowired
    private InventoryService inventory;
    
    @Autowired
    private PaymentService payment;
    
    @Autowired
    private ShippingService shipping;
    
    public void placeOrder(Order order) {
        // Mediator coordinates multiple services
        inventory.reserve(order);
        payment.process(order);
        shipping.schedule(order);
    }
}
```

## When to Use Mediator

Use Mediator when:
- Objects communicate in complex but well-defined ways
- Reusing objects is difficult because they refer to many other objects
- Behavior distributed among several classes should be customizable
- You want to reduce coupling between communicating objects

Real scenarios:
- GUI components coordination
- Chat rooms and messaging systems
- Air traffic control systems
- Workflow engines
- Event dispatching systems
- Form validation
- Multi-component interactions

## Mediator vs Facade

**Mediator:** Bi-directional communication. Components know about mediator, mediator knows about components.

**Facade:** Uni-directional. Facade knows about subsystem, subsystem doesn't know about facade.

**Mediator:** Coordinates peer-to-peer interactions.

**Facade:** Simplifies access to a subsystem.

## Mediator vs Observer

**Mediator:** One-to-many relationships between specific components. Centralized control.

**Observer:** One-to-many dependencies. Decentralized, automatic notifications.

**Mediator:** "I need to coordinate these specific components."

**Observer:** "Notify everyone interested when state changes."

Can use both together: mediator uses observer pattern to notify components.

## Common Pitfalls

### Pitfall 1: God Object Mediator

```java
// BAD: Mediator doing everything
class GodMediator {
    // Business logic
    // Validation logic
    // Persistence logic
    // UI logic
    // Everything!
}
```

Mediator should only coordinate. Don't dump all logic into it.

### Pitfall 2: Components Bypassing Mediator

```java
// BAD: Direct communication
class Component1 {
    private Component2 component2;
    
    public void doSomething() {
        component2.doSomethingElse();  // Bypassing mediator!
    }
}
```

All communication should go through mediator.

### Pitfall 3: Circular Dependencies

```java
// BAD: Mediator and components tightly coupled
class Mediator {
    private Component1 c1 = new Component1(this);  // Creates component
}

class Component1 {
    public Component1(Mediator m) {
        // Also creates mediator somewhere?
    }
}
```

Use dependency injection to avoid circular dependencies.

## Mediator and SOLID Principles

### Single Responsibility Principle

Each component has its own responsibility. Mediator's responsibility is coordination.

### Open/Closed Principle

Add new components without modifying existing ones. Update mediator to coordinate new components.

### Dependency Inversion Principle

Components depend on mediator interface, not concrete mediator.

## The Mental Model

Think of Mediator like:

**Airport control tower:** Airplanes don't talk to each other. They all talk to the tower. Tower coordinates takeoffs, landings, and airspace.

**Orchestrator in music:** Musicians don't coordinate with each other directly. They all follow the conductor, who coordinates the performance.

**Slack or Teams:** People don't call each other directly. They send messages through the platform, which handles routing, notifications, and coordination.

## Performance Considerations

Mediator can become a bottleneck:
- All communication goes through one point
- Complex coordination logic in one place

Mitigations:
- Keep mediator logic simple
- Consider multiple mediators for different concerns
- Use asynchronous communication if appropriate

## Testing with Mediator

```java
@Test
public void testMediatorCoordinatesComponents() {
    ChatMediator mediator = new ChatRoom();
    User alice = new User("Alice", mediator);
    User bob = new User("Bob", mediator);
    
    mediator.addUser(alice);
    mediator.addUser(bob);
    
    alice.send("Hello");
    
    // Verify bob received message (through mediator)
}

@Test
public void testFormValidation() {
    RegistrationFormMediator form = new RegistrationFormMediator();
    
    assertFalse(form.getSubmitButton().isEnabled());
    
    form.getUsername().setValue("alice");
    form.getPassword().setValue("password123");
    form.getConfirmPassword().setValue("password123");
    form.getTermsCheckbox().setValue("true");
    
    assertTrue(form.getSubmitButton().isEnabled());
}
```

## Final Thoughts

The Mediator pattern is about reducing chaos. Instead of n objects all talking to each other (n² connections), they all talk to one mediator (n connections).

It's not about adding complexity. It's about:
- Reducing coupling between components
- Centralizing coordination logic
- Making systems easier to maintain
- Enabling component reuse

The key insight: objects shouldn't know about each other's existence. A mediator coordinates their interactions.

Remember:
- Mediator knows about all components
- Components only know about mediator
- Coordination logic lives in mediator
- Don't make mediator a god object

Next time you have objects all coupled to each other, stop. Think Mediator. Create a coordinator that handles their interactions.

Mediate wisely.