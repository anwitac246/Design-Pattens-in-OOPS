# The Memento Pattern: Time Travel for Objects

Ever hit undo in a text editor? Ever restored from a saved game? Ever rolled back a database transaction? You've experienced the Memento pattern.

The idea: capture an object's internal state without exposing its structure, so you can restore it later.

## The Problem: Saving State Breaks Encapsulation

You're building a text editor with undo functionality:

```java
class TextEditor {
    private StringBuilder content;
    
    public TextEditor() {
        content = new StringBuilder();
    }
    
    public void write(String text) {
        content.append(text);
    }
    
    public String getContent() {
        return content.toString();
    }
    
    // How do we implement undo?
    // Exposing StringBuilder breaks encapsulation
    public StringBuilder getState() {
        return content;  // BAD: exposes internal structure
    }
    
    public void setState(StringBuilder state) {
        this.content = state;  // BAD: accepts internal structure
    }
}
```

Problems:

**Breaks encapsulation:** Exposing internal `StringBuilder` to save state.

**Tight coupling:** Client knows about internal structure. Change from `StringBuilder` to `String[]`? All client code breaks.

**No protection:** Client can modify the state directly.

What if we could save state without exposing internals?

## The Memento Solution

Create a memento object that holds state:

```java
// Originator: object whose state we want to save
class TextEditor {
    private StringBuilder content;
    
    public TextEditor() {
        content = new StringBuilder();
    }
    
    public void write(String text) {
        content.append(text);
    }
    
    public String getContent() {
        return content.toString();
    }
    
    // Create memento
    public Memento save() {
        return new Memento(content.toString());
    }
    
    // Restore from memento
    public void restore(Memento memento) {
        this.content = new StringBuilder(memento.getState());
    }
    
    // Memento: snapshot of state
    public static class Memento {
        private final String state;
        
        private Memento(String state) {
            this.state = state;
        }
        
        private String getState() {
            return state;
        }
    }
}

// Caretaker: manages mementos
class History {
    private Stack<TextEditor.Memento> history;
    
    public History() {
        history = new Stack<>();
    }
    
    public void save(TextEditor editor) {
        history.push(editor.save());
    }
    
    public void undo(TextEditor editor) {
        if (!history.isEmpty()) {
            editor.restore(history.pop());
        }
    }
}
```

Usage:

```java
TextEditor editor = new TextEditor();
History history = new History();

editor.write("Hello");
history.save(editor);  // Save state

editor.write(" World");
history.save(editor);  // Save state

editor.write("!");
System.out.println(editor.getContent());  // "Hello World!"

history.undo(editor);
System.out.println(editor.getContent());  // "Hello World"

history.undo(editor);
System.out.println(editor.getContent());  // "Hello"
```

The memento holds state without exposing `TextEditor`'s internal structure.

## The Components

### 1. Originator

Object whose state needs to be saved:

```java
class Originator {
    private String state;
    
    public Memento save() {
        return new Memento(state);
    }
    
    public void restore(Memento memento) {
        this.state = memento.getState();
    }
}
```

### 2. Memento

Stores originator's state:

```java
class Memento {
    private final String state;
    
    private Memento(String state) {
        this.state = state;
    }
    
    private String getState() {
        return state;
    }
}
```

Note: Memento's constructor and getter are private. Only accessible to originator (if it's an inner class).

### 3. Caretaker

Manages mementos but doesn't examine or modify them:

```java
class Caretaker {
    private List<Memento> history = new ArrayList<>();
    
    public void save(Originator originator) {
        history.add(originator.save());
    }
    
    public void undo(Originator originator) {
        if (!history.isEmpty()) {
            Memento memento = history.remove(history.size() - 1);
            originator.restore(memento);
        }
    }
}
```

## Real-World Example: Game Save System

```java
class GameState {
    private int level;
    private int health;
    private int score;
    private List<String> inventory;
    
    public GameState() {
        this.level = 1;
        this.health = 100;
        this.score = 0;
        this.inventory = new ArrayList<>();
    }
    
    public void play(String action) {
        if (action.equals("fight")) {
            health -= 20;
            score += 100;
            System.out.println("Fought enemy! Health: " + health + ", Score: " + score);
        } else if (action.equals("collect")) {
            inventory.add("sword");
            System.out.println("Collected sword!");
        } else if (action.equals("level_up")) {
            level++;
            health = 100;
            System.out.println("Level up! Now at level " + level);
        }
    }
    
    public SavePoint createSavePoint() {
        return new SavePoint(level, health, score, new ArrayList<>(inventory));
    }
    
    public void loadSavePoint(SavePoint savePoint) {
        this.level = savePoint.level;
        this.health = savePoint.health;
        this.score = savePoint.score;
        this.inventory = new ArrayList<>(savePoint.inventory);
        System.out.println("Game loaded: Level " + level + ", Health " + health);
    }
    
    // Memento
    public static class SavePoint {
        private final int level;
        private final int health;
        private final int score;
        private final List<String> inventory;
        
        private SavePoint(int level, int health, int score, List<String> inventory) {
            this.level = level;
            this.health = health;
            this.score = score;
            this.inventory = inventory;
        }
    }
}

// Caretaker
class GameSaveManager {
    private Map<String, GameState.SavePoint> saves;
    
    public GameSaveManager() {
        saves = new HashMap<>();
    }
    
    public void save(String name, GameState game) {
        saves.put(name, game.createSavePoint());
        System.out.println("Game saved: " + name);
    }
    
    public void load(String name, GameState game) {
        GameState.SavePoint savePoint = saves.get(name);
        if (savePoint != null) {
            game.loadSavePoint(savePoint);
        } else {
            System.out.println("Save not found: " + name);
        }
    }
    
    public List<String> listSaves() {
        return new ArrayList<>(saves.keySet());
    }
}
```

Usage:

```java
GameState game = new GameState();
GameSaveManager saveManager = new GameSaveManager();

// Play game
game.play("collect");
game.play("fight");
saveManager.save("checkpoint1", game);

game.play("level_up");
game.play("fight");
saveManager.save("checkpoint2", game);

game.play("fight");  // Health drops

// Load previous save
saveManager.load("checkpoint2", game);  // Restore to checkpoint2
```

## Another Example: Configuration Snapshots

```java
class DatabaseConfig {
    private String host;
    private int port;
    private String database;
    private Map<String, String> options;
    
    public DatabaseConfig(String host, int port, String database) {
        this.host = host;
        this.port = port;
        this.database = database;
        this.options = new HashMap<>();
    }
    
    public void setOption(String key, String value) {
        options.put(key, value);
    }
    
    public ConfigSnapshot snapshot() {
        return new ConfigSnapshot(host, port, database, new HashMap<>(options));
    }
    
    public void restore(ConfigSnapshot snapshot) {
        this.host = snapshot.host;
        this.port = snapshot.port;
        this.database = snapshot.database;
        this.options = new HashMap<>(snapshot.options);
    }
    
    public void displayConfig() {
        System.out.println("Host: " + host + ":" + port);
        System.out.println("Database: " + database);
        System.out.println("Options: " + options);
    }
    
    // Memento
    public static class ConfigSnapshot {
        private final String host;
        private final int port;
        private final String database;
        private final Map<String, String> options;
        private final long timestamp;
        
        private ConfigSnapshot(String host, int port, String database, 
                              Map<String, String> options) {
            this.host = host;
            this.port = port;
            this.database = database;
            this.options = options;
            this.timestamp = System.currentTimeMillis();
        }
        
        public long getTimestamp() {
            return timestamp;
        }
    }
}
```

## Multi-Level Undo/Redo

```java
class UndoRedoManager {
    private Stack<TextEditor.Memento> undoStack;
    private Stack<TextEditor.Memento> redoStack;
    
    public UndoRedoManager() {
        undoStack = new Stack<>();
        redoStack = new Stack<>();
    }
    
    public void save(TextEditor editor) {
        undoStack.push(editor.save());
        redoStack.clear();  // Clear redo stack on new action
    }
    
    public void undo(TextEditor editor) {
        if (!undoStack.isEmpty()) {
            // Save current state to redo stack
            redoStack.push(editor.save());
            // Restore previous state
            editor.restore(undoStack.pop());
        }
    }
    
    public void redo(TextEditor editor) {
        if (!redoStack.isEmpty()) {
            // Save current state to undo stack
            undoStack.push(editor.save());
            // Restore redo state
            editor.restore(redoStack.pop());
        }
    }
    
    public boolean canUndo() {
        return !undoStack.isEmpty();
    }
    
    public boolean canRedo() {
        return !redoStack.isEmpty();
    }
}
```

Usage:

```java
TextEditor editor = new TextEditor();
UndoRedoManager manager = new UndoRedoManager();

editor.write("A");
manager.save(editor);

editor.write("B");
manager.save(editor);

editor.write("C");
System.out.println(editor.getContent());  // "ABC"

manager.undo(editor);
System.out.println(editor.getContent());  // "AB"

manager.undo(editor);
System.out.println(editor.getContent());  // "A"

manager.redo(editor);
System.out.println(editor.getContent());  // "AB"
```

## When to Use Memento

Use Memento when:
- You need to save and restore object state
- You want undo/redo functionality
- Direct access to state would break encapsulation
- State needs to be externalized without exposing structure

Real scenarios:
- Text editors (undo/redo)
- Games (save/load)
- Database transactions (rollback)
- Configuration management (snapshots)
- Drawing applications (undo)
- Version control systems
- Form drafts

## Memento vs Command for Undo

**Memento:** Stores complete state. Undo by restoring previous state.

**Command:** Stores operations. Undo by reversing operations.

**Memento:**
```java
// Save entire state
Memento m = editor.save();  // State: "Hello World"
editor.write("!");
editor.restore(m);  // Back to "Hello World"
```

**Command:**
```java
// Store operation
Command cmd = new InsertTextCommand(editor, "!");
cmd.execute();
cmd.undo();  // Reverse the specific operation
```

Memento is simpler but uses more memory. Command is more complex but more efficient.

## Common Pitfalls

### Pitfall 1: Exposing Memento Contents

```java
// BAD: Public access to state
class Memento {
    public String state;  // Anyone can access/modify!
}

// GOOD: Private access
class Memento {
    private final String state;
    
    private Memento(String state) {
        this.state = state;
    }
    
    private String getState() {  // Only originator can access
        return state;
    }
}
```

### Pitfall 2: Memory Leaks

```java
// BAD: Unlimited history
class History {
    private List<Memento> history = new ArrayList<>();
    
    public void save(Editor editor) {
        history.add(editor.save());  // Grows forever!
    }
}

// GOOD: Limited history
class History {
    private static final int MAX_HISTORY = 50;
    private LinkedList<Memento> history = new LinkedList<>();
    
    public void save(Editor editor) {
        if (history.size() >= MAX_HISTORY) {
            history.removeFirst();  // Remove oldest
        }
        history.add(editor.save());
    }
}
```

### Pitfall 3: Shallow Copying

```java
// BAD: Shallow copy
class Memento {
    private List<String> data;
    
    public Memento(List<String> data) {
        this.data = data;  // Same reference!
    }
}

// GOOD: Deep copy
class Memento {
    private List<String> data;
    
    public Memento(List<String> data) {
        this.data = new ArrayList<>(data);  // New list
    }
}
```

## Serialization for Memento

Java serialization can simplify memento creation:

```java
class SerializableEditor implements Serializable {
    private StringBuilder content;
    
    public byte[] save() throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(this);
        return baos.toByteArray();
    }
    
    public static SerializableEditor restore(byte[] data) throws Exception {
        ByteArrayInputStream bais = new ByteArrayInputStream(data);
        ObjectInputStream ois = new ObjectInputStream(bais);
        return (SerializableEditor) ois.readObject();
    }
}
```

Pros: Easy to implement, handles complex objects

Cons: Slower, requires Serializable, whole object serialized

## Memento and SOLID Principles

### Single Responsibility Principle

Originator manages business logic. Memento stores state. Caretaker manages history.

### Open/Closed Principle

Add new state fields without changing caretaker.

## The Mental Model

Think of Memento like:

**Save games:** Press save, game creates a snapshot. Press load, game restores from snapshot. You don't see what's in the save file.

**Time machine:** Take a photo of current state. Later, restore everything to how it was in the photo.

**System restore point:** Windows creates restore points. If something breaks, restore from point. You don't see what data is stored, just "restore to this point."

## Performance Considerations

Memento can be expensive:
- Memory: Each snapshot stores full state
- Time: Creating deep copies takes time

Optimizations:
- Limit history size
- Use incremental snapshots (store only changes)
- Compress mementos
- Lazy copying (copy-on-write)

Example of incremental snapshots:

```java
class Editor {
    public Memento saveDelta(Memento previous) {
        // Store only changes since previous
        String previousContent = previous.getState();
        String currentContent = this.content.toString();
        String delta = computeDelta(previousContent, currentContent);
        return new DeltaMemento(previous, delta);
    }
}
```

## Testing with Memento

```java
@Test
public void testSaveAndRestore() {
    TextEditor editor = new TextEditor();
    editor.write("Hello");
    
    Memento memento = editor.save();
    
    editor.write(" World");
    assertEquals("Hello World", editor.getContent());
    
    editor.restore(memento);
    assertEquals("Hello", editor.getContent());
}

@Test
public void testMultipleUndos() {
    TextEditor editor = new TextEditor();
    History history = new History();
    
    editor.write("A");
    history.save(editor);
    
    editor.write("B");
    history.save(editor);
    
    history.undo(editor);
    assertEquals("A", editor.getContent());
    
    history.undo(editor);
    assertEquals("", editor.getContent());
}
```

## Final Thoughts

The Memento pattern is about capturing state without breaking encapsulation. It enables time travel for objects—save state, restore state, undo changes—all while keeping internals private.

It's not about complexity. It's about:
- Preserving encapsulation
- Enabling undo/redo
- Managing object history
- Saving and restoring state

The key insight: you can save an object's state without exposing its internal structure.

Remember:
- Originator creates and uses mementos
- Memento stores state opaquely
- Caretaker manages but doesn't examine mementos
- Watch memory usage

Next time you need undo functionality or state snapshots, think Memento. Capture state, preserve encapsulation, enable time travel.

Save your progress.