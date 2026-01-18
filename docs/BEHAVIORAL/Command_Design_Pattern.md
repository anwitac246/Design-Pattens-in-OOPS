# The Command Pattern: Turning Requests Into Objects

Ever used undo in a text editor? Or set up macros in Excel? Or programmed a universal remote? Behind each of these features is the Command pattern, quietly turning actions into objects.

The insight is simple but powerful: instead of calling methods directly, wrap the request in an object. Suddenly, you can queue requests, log them, undo them, and do all sorts of things you couldn't do before.

## The Problem: Direct Method Calls

You're building a home automation system. You have various devices:

```java
class Light {
    public void on() { System.out.println("Light is ON"); }
    public void off() { System.out.println("Light is OFF"); }
}

class Fan {
    public void high() { System.out.println("Fan on HIGH"); }
    public void low() { System.out.println("Fan on LOW"); }
    public void off() { System.out.println("Fan is OFF"); }
}

class TV {
    public void on() { System.out.println("TV is ON"); }
    public void off() { System.out.println("TV is OFF"); }
    public void setChannel(int channel) { 
        System.out.println("Channel set to " + channel); 
    }
}
```

Now you want a universal remote control with programmable buttons. The naive approach:

```java
class RemoteControl {
    private Light light;
    private Fan fan;
    private TV tv;
    
    public void button1Pressed() {
        light.on();
    }
    
    public void button2Pressed() {
        fan.high();
    }
    
    public void button3Pressed() {
        tv.on();
        tv.setChannel(5);
    }
}
```

Problems:

**Tightly coupled:** Remote knows about every device and every method.

**Not flexible:** Want to reprogram button 1? Modify the remote. Want to add a new device? Modify the remote.

**No undo:** How do you undo button presses? You'd need to track state for every device.

**No macros:** Want button 3 to turn on lights AND TV? More hardcoding.

**Hard to test:** Can't test button logic without actual devices.

## The Command Solution

Instead of the remote calling methods directly, it executes command objects:

```java
// Command interface
interface Command {
    void execute();
    void undo();
}

// Concrete commands
class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    public void execute() {
        light.on();
    }
    
    public void undo() {
        light.off();
    }
}

class LightOffCommand implements Command {
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    public void execute() {
        light.off();
    }
    
    public void undo() {
        light.on();
    }
}

class FanHighCommand implements Command {
    private Fan fan;
    private String previousSpeed;
    
    public FanHighCommand(Fan fan) {
        this.fan = fan;
    }
    
    public void execute() {
        previousSpeed = "OFF";  // Simplified, would track actual state
        fan.high();
    }
    
    public void undo() {
        if (previousSpeed.equals("OFF")) {
            fan.off();
        } else if (previousSpeed.equals("LOW")) {
            fan.low();
        }
    }
}

// Remote control (invoker)
class RemoteControl {
    private Command[] onCommands;
    private Command[] offCommands;
    private Command lastCommand;
    
    public RemoteControl() {
        onCommands = new Command[7];  // 7 buttons
        offCommands = new Command[7];
        
        // Default command (null object pattern)
        Command noCommand = new NoCommand();
        for (int i = 0; i < 7; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
    }
    
    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }
    
    public void onButtonPressed(int slot) {
        onCommands[slot].execute();
        lastCommand = onCommands[slot];
    }
    
    public void offButtonPressed(int slot) {
        offCommands[slot].execute();
        lastCommand = offCommands[slot];
    }
    
    public void undoButtonPressed() {
        if (lastCommand != null) {
            lastCommand.undo();
        }
    }
}

// Null command (prevents null checks)
class NoCommand implements Command {
    public void execute() { }
    public void undo() { }
}
```

Usage:

```java
// Create devices
Light livingRoomLight = new Light();
Fan ceilingFan = new Fan();
TV television = new TV();

// Create commands
Command lightOn = new LightOnCommand(livingRoomLight);
Command lightOff = new LightOffCommand(livingRoomLight);
Command fanHigh = new FanHighCommand(ceilingFan);

// Create remote
RemoteControl remote = new RemoteControl();

// Program buttons
remote.setCommand(0, lightOn, lightOff);
remote.setCommand(1, fanHigh, new NoCommand());

// Use remote
remote.onButtonPressed(0);   // Light is ON
remote.undoButtonPressed();  // Light is OFF

remote.onButtonPressed(1);   // Fan on HIGH
remote.undoButtonPressed();  // Fan is OFF
```

The remote doesn't know about lights or fans. It just executes commands. Want to reprogram a button? Assign a different command. Want undo? Already built in.

## The Components

### 1. Command Interface

Defines the contract:

```java
interface Command {
    void execute();
    void undo();
}
```

### 2. Concrete Commands

Encapsulate a request:

```java
class LightOnCommand implements Command {
    private Light light;  // Receiver
    
    public void execute() {
        light.on();
    }
    
    public void undo() {
        light.off();
    }
}
```

### 3. Receiver

The object that performs the actual work:

```java
class Light {
    public void on() { /* actual work */ }
    public void off() { /* actual work */ }
}
```

### 4. Invoker

Executes commands:

```java
class RemoteControl {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
    }
}
```

### 5. Client

Creates and configures commands:

```java
Light light = new Light();
Command command = new LightOnCommand(light);
RemoteControl remote = new RemoteControl();
remote.setCommand(command);
```

## Real-World Example: Text Editor with Undo

```java
// Receiver
class TextEditor {
    private StringBuilder content;
    
    public TextEditor() {
        content = new StringBuilder();
    }
    
    public void insertText(String text, int position) {
        content.insert(position, text);
    }
    
    public void deleteText(int start, int end) {
        content.delete(start, end);
    }
    
    public String getContent() {
        return content.toString();
    }
}

// Commands
class InsertTextCommand implements Command {
    private TextEditor editor;
    private String text;
    private int position;
    
    public InsertTextCommand(TextEditor editor, String text, int position) {
        this.editor = editor;
        this.text = text;
        this.position = position;
    }
    
    public void execute() {
        editor.insertText(text, position);
    }
    
    public void undo() {
        editor.deleteText(position, position + text.length());
    }
}

class DeleteTextCommand implements Command {
    private TextEditor editor;
    private String deletedText;
    private int start;
    private int end;
    
    public DeleteTextCommand(TextEditor editor, int start, int end) {
        this.editor = editor;
        this.start = start;
        this.end = end;
        // Save text for undo
        this.deletedText = editor.getContent().substring(start, end);
    }
    
    public void execute() {
        editor.deleteText(start, end);
    }
    
    public void undo() {
        editor.insertText(deletedText, start);
    }
}

// Invoker with command history
class EditorInvoker {
    private Stack<Command> history;
    
    public EditorInvoker() {
        history = new Stack<>();
    }
    
    public void executeCommand(Command command) {
        command.execute();
        history.push(command);
    }
    
    public void undo() {
        if (!history.isEmpty()) {
            Command command = history.pop();
            command.undo();
        }
    }
}
```

Usage:

```java
TextEditor editor = new TextEditor();
EditorInvoker invoker = new EditorInvoker();

// Type some text
invoker.executeCommand(new InsertTextCommand(editor, "Hello", 0));
System.out.println(editor.getContent());  // Hello

invoker.executeCommand(new InsertTextCommand(editor, " World", 5));
System.out.println(editor.getContent());  // Hello World

// Undo
invoker.undo();
System.out.println(editor.getContent());  // Hello

invoker.undo();
System.out.println(editor.getContent());  // (empty)
```

## Macro Commands (Composite Commands)

Execute multiple commands as one:

```java
class MacroCommand implements Command {
    private Command[] commands;
    
    public MacroCommand(Command[] commands) {
        this.commands = commands;
    }
    
    public void execute() {
        for (Command command : commands) {
            command.execute();
        }
    }
    
    public void undo() {
        // Undo in reverse order
        for (int i = commands.length - 1; i >= 0; i--) {
            commands[i].undo();
        }
    }
}
```

Usage:

```java
// "Party mode" - turn on lights, TV, and music
Command[] partyCommands = {
    new LightOnCommand(light),
    new TVOnCommand(tv),
    new StereoOnCommand(stereo)
};

Command partyMode = new MacroCommand(partyCommands);

// One button press does everything
remote.setCommand(0, partyMode, new NoCommand());
remote.onButtonPressed(0);  // Lights, TV, and stereo all turn on

remote.undoButtonPressed();  // Everything turns off in reverse order
```

## Command Queue Example

```java
class CommandQueue {
    private Queue<Command> queue;
    
    public CommandQueue() {
        queue = new LinkedList<>();
    }
    
    public void addCommand(Command command) {
        queue.offer(command);
    }
    
    public void processCommands() {
        while (!queue.isEmpty()) {
            Command command = queue.poll();
            command.execute();
            
            // Could add delays, logging, etc.
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

Use cases:
- Job queues
- Task scheduling
- Request buffering
- Load balancing

## Command Pattern in Real Frameworks

### Java Runnable/Callable

```java
// Runnable is essentially a command
Runnable command = () -> System.out.println("Executing task");
new Thread(command).start();

// Or with ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(command);
```

### Spring Framework

```java
// JdbcTemplate uses command pattern
jdbcTemplate.execute("CREATE TABLE users (id INT, name VARCHAR(50))");
```

### Android Development

```java
// Event handlers are commands
button.setOnClickListener(new OnClickListener() {
    public void onClick(View v) {
        // Command execution
    }
});
```

## When to Use Command

Use Command when:
- You need to parameterize objects with operations
- You need to queue operations
- You need to implement undo/redo
- You need to log operations
- You need to support transactions

Real scenarios:
- GUI buttons and menu items
- Transaction management
- Macro recording
- Job/task queues
- Undo/redo functionality
- Database migrations
- Network request handling
- Game input systems

## Command vs Strategy

Both encapsulate algorithms, but:

**Strategy:** Focuses on how to do something. Different algorithms for the same task.

**Command:** Focuses on what to do and when. Encapsulates requests for later execution.

Strategy is about interchangeable algorithms. Command is about delaying and managing execution.

## Command vs Memento

**Command:** Stores operations (behavior). Enables undo by reversing operations.

**Memento:** Stores state (data). Enables undo by restoring previous state.

Commands are about actions. Mementos are about snapshots.

## Common Pitfalls

### Pitfall 1: Complex Undo Logic

```java
// BAD: Undo requires recomputing everything
class ComplexCommand implements Command {
    public void undo() {
        // Recalculate entire state from scratch
        // This defeats the purpose!
    }
}
```

Store enough information in the command to efficiently undo.

### Pitfall 2: Forgetting to Save State

```java
// BAD: Can't undo because we don't know previous state
class SetVolumeCommand implements Command {
    private Stereo stereo;
    private int newVolume;
    // Missing: previous volume!
    
    public void undo() {
        // What was the previous volume??
    }
}
```

Save state before executing:

```java
class SetVolumeCommand implements Command {
    private Stereo stereo;
    private int newVolume;
    private int previousVolume;
    
    public void execute() {
        previousVolume = stereo.getVolume();
        stereo.setVolume(newVolume);
    }
    
    public void undo() {
        stereo.setVolume(previousVolume);
    }
}
```

### Pitfall 3: Leaking Command History

```java
// BAD: Unlimited undo history
class Invoker {
    private Stack<Command> history;  // Grows forever!
}
```

Limit history size:

```java
class Invoker {
    private static final int MAX_HISTORY = 100;
    private LinkedList<Command> history;
    
    public void executeCommand(Command command) {
        command.execute();
        history.addFirst(command);
        if (history.size() > MAX_HISTORY) {
            history.removeLast();
        }
    }
}
```

## Command and SOLID Principles

### Single Responsibility Principle

Each command has one job: encapsulate one action.

### Open/Closed Principle

Add new commands without modifying existing code.

### Liskov Substitution Principle

Any command can replace another anywhere `Command` interface is expected.

### Dependency Inversion Principle

Invoker depends on `Command` interface, not concrete commands.

## The Mental Model

Think of Command like:

**Restaurant order:** Waiter (invoker) doesn't cook. Takes your order (command), gives it to kitchen (receiver). Order slip contains all info needed. Can be queued, prioritized, or canceled.

**Queue at coffee shop:** Each order (command) is independent. Barista (invoker) processes them one by one. Orders can be undone (remake), logged (receipt), or batched (group orders).

**TV remote:** Each button is a command object. Remote doesn't know what TV does. Just executes the command. Commands can be reprogrammed without changing the remote.

## Performance Considerations

Commands add overhead:
- Extra objects created
- Possibly storing undo state
- Command history memory

Usually negligible. The flexibility gained is worth it.

For performance-critical code with millions of commands per second, consider:
- Object pooling
- Flyweight pattern for commands
- Limiting undo history

## Testing with Command

Command pattern makes testing easier:

```java
@Test
public void testLightOnCommand() {
    Light light = mock(Light.class);
    Command command = new LightOnCommand(light);
    
    command.execute();
    verify(light).on();
    
    command.undo();
    verify(light).off();
}

@Test
public void testRemoteControl() {
    Command mockCommand = mock(Command.class);
    RemoteControl remote = new RemoteControl();
    remote.setCommand(0, mockCommand, new NoCommand());
    
    remote.onButtonPressed(0);
    verify(mockCommand).execute();
}
```

Each component is testable in isolation.

## Final Thoughts

The Command pattern is about turning requests into first-class objects. Once you do that, a world of possibilities opens up: queuing, logging, undo, macros, delayed execution.

It's not about being clever. It's about:
- Decoupling sender from receiver
- Making requests reusable
- Adding functionality without changing existing code
- Following SOLID principles

The key insight: requests are just data. Treat them as such. Wrap them in objects, and you can do anything with them.

Remember:
- Command encapsulates a request
- Invoker doesn't know about receivers
- Receivers don't know about commands
- Undo is built-in if you design for it

Next time you're directly calling methods and wishing you could undo, queue, or log them, stop. Think Command. Wrap those calls in objects.

Execute wisely.