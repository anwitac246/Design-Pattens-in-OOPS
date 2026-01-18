# The Flyweight Pattern: Sharing Is Caring (And Memory-Efficient)

Imagine you're building a text editor. Your document has 100,000 characters. Do you create 100,000 separate character objects? That would use gigabytes of memory for a simple document.

The Flyweight pattern says: most of those characters share common data. Store it once, share it everywhere.

## The Problem: Memory Waste Through Duplication

Let's say you're building a forest rendering system for a game:

```java
class Tree {
    private String type;          // "Oak", "Pine", etc.
    private Color color;          // Tree color
    private Texture texture;      // Bark texture
    private Texture leafTexture;  // Leaf texture
    
    // Position varies per tree
    private int x;
    private int y;
    
    public Tree(String type, Color color, Texture texture, 
                Texture leafTexture, int x, int y) {
        this.type = type;
        this.color = color;
        this.texture = texture;
        this.leafTexture = leafTexture;
        this.x = x;
        this.y = y;
    }
    
    public void display() {
        System.out.println("Drawing " + type + " tree at (" + x + "," + y + ")");
    }
}
```

Now create a forest:

```java
List<Tree> forest = new ArrayList<>();

for (int i = 0; i < 100000; i++) {
    forest.add(new Tree("Oak", oakColor, oakTexture, oakLeafTexture, 
                        random.nextInt(), random.nextInt()));
}
```

You just created 100,000 Tree objects. Each stores type, color, texture, and leaf texture. But for an oak forest, 99% of trees are identical except for position.

Problem: You're duplicating the same type, color, and textures 100,000 times. That's massive memory waste.

## The Flyweight Solution

Separate intrinsic state (shared) from extrinsic state (unique):

```java
// Flyweight: Shared data
class TreeType {
    private String name;
    private Color color;
    private Texture texture;
    private Texture leafTexture;
    
    public TreeType(String name, Color color, Texture texture, Texture leafTexture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
        this.leafTexture = leafTexture;
    }
    
    public void display(int x, int y) {
        System.out.println("Drawing " + name + " tree at (" + x + "," + y + ")");
        // Use color, texture, leafTexture to render
    }
}

// Context: Unique data
class Tree {
    private int x;
    private int y;
    private TreeType type;  // Reference to shared flyweight
    
    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }
    
    public void display() {
        type.display(x, y);
    }
}

// Flyweight Factory: Manages flyweight instances
class TreeFactory {
    private static Map<String, TreeType> treeTypes = new HashMap<>();
    
    public static TreeType getTreeType(String name, Color color, 
                                       Texture texture, Texture leafTexture) {
        String key = name;  // Use name as key (simplified)
        
        TreeType type = treeTypes.get(key);
        if (type == null) {
            type = new TreeType(name, color, texture, leafTexture);
            treeTypes.put(key, type);
            System.out.println("Creating new TreeType: " + name);
        }
        return type;
    }
}
```

Usage:

```java
List<Tree> forest = new ArrayList<>();

// Get shared tree types
TreeType oakType = TreeFactory.getTreeType("Oak", oakColor, oakTexture, oakLeafTexture);
TreeType pineType = TreeFactory.getTreeType("Pine", pineColor, pineTexture, pineLeafTexture);

// Create 100,000 trees
for (int i = 0; i < 100000; i++) {
    TreeType type = (i % 2 == 0) ? oakType : pineType;
    forest.add(new Tree(random.nextInt(), random.nextInt(), type));
}
```

Now you have:
- 2 TreeType objects (shared)
- 100,000 Tree objects (each storing only position and a reference)

Memory saved: instead of storing type, color, and textures 100,000 times, you store them twice and reference them 100,000 times. Huge savings.

## The Components

### 1. Flyweight

Contains intrinsic (shared) state:

```java
class TreeType {
    private String name;      // Shared
    private Color color;      // Shared
    private Texture texture;  // Shared
    
    public void display(int x, int y) {  // Extrinsic state passed in
        // Render using shared data and position
    }
}
```

### 2. Context

Contains extrinsic (unique) state and references flyweight:

```java
class Tree {
    private int x, y;          // Unique
    private TreeType type;     // Reference to flyweight
}
```

### 3. Flyweight Factory

Creates and manages flyweight instances:

```java
class TreeFactory {
    private static Map<String, TreeType> cache = new HashMap<>();
    
    public static TreeType getTreeType(String name, ...) {
        // Return existing or create new
    }
}
```

## Real-World Example: Text Editor Characters

```java
// Flyweight: Character format
class CharacterFormat {
    private String fontFamily;
    private int fontSize;
    private Color color;
    private boolean bold;
    private boolean italic;
    
    public CharacterFormat(String fontFamily, int fontSize, Color color, 
                          boolean bold, boolean italic) {
        this.fontFamily = fontFamily;
        this.fontSize = fontSize;
        this.color = color;
        this.bold = bold;
        this.italic = italic;
    }
    
    public void display(char c, int x, int y) {
        System.out.println("Drawing '" + c + "' at (" + x + "," + y + ") " +
                          "with " + fontFamily + " " + fontSize + "pt");
    }
}

// Context: Individual character
class Character {
    private char value;
    private int x, y;
    private CharacterFormat format;  // Shared
    
    public Character(char value, int x, int y, CharacterFormat format) {
        this.value = value;
        this.x = x;
        this.y = y;
        this.format = format;
    }
    
    public void display() {
        format.display(value, x, y);
    }
}

// Flyweight Factory
class FormatFactory {
    private static Map<String, CharacterFormat> formats = new HashMap<>();
    
    public static CharacterFormat getFormat(String fontFamily, int fontSize, 
                                           Color color, boolean bold, boolean italic) {
        String key = fontFamily + fontSize + color + bold + italic;
        
        CharacterFormat format = formats.get(key);
        if (format == null) {
            format = new CharacterFormat(fontFamily, fontSize, color, bold, italic);
            formats.put(key, format);
        }
        return format;
    }
}
```

Usage:

```java
// Create a document
List<Character> document = new ArrayList<>();

// Get shared formats
CharacterFormat defaultFormat = FormatFactory.getFormat("Arial", 12, Color.BLACK, false, false);
CharacterFormat boldFormat = FormatFactory.getFormat("Arial", 12, Color.BLACK, true, false);
CharacterFormat headerFormat = FormatFactory.getFormat("Arial", 18, Color.BLUE, true, false);

// Create characters (position varies, format is shared)
document.add(new Character('H', 0, 0, headerFormat));
document.add(new Character('e', 10, 0, headerFormat));
document.add(new Character('l', 20, 0, headerFormat));
// ... thousands more characters

// Most characters use defaultFormat (shared)
// Only headers, bold text use different formats (also shared)
// Memory efficient!
```

## Another Example: Particle System

```java
// Flyweight: Particle properties
class ParticleProperties {
    private Color color;
    private Texture texture;
    private float mass;
    
    public ParticleProperties(Color color, Texture texture, float mass) {
        this.color = color;
        this.texture = texture;
        this.mass = mass;
    }
    
    public void render(float x, float y, float velocityX, float velocityY) {
        System.out.println("Rendering particle at (" + x + "," + y + ")");
        // Use color, texture to draw
    }
}

// Context: Individual particle
class Particle {
    private float x, y;
    private float velocityX, velocityY;
    private ParticleProperties properties;  // Shared
    
    public Particle(float x, float y, float vx, float vy, ParticleProperties props) {
        this.x = x;
        this.y = y;
        this.velocityX = vx;
        this.velocityY = vy;
        this.properties = props;
    }
    
    public void update(float dt) {
        x += velocityX * dt;
        y += velocityY * dt;
    }
    
    public void render() {
        properties.render(x, y, velocityX, velocityY);
    }
}

// Factory
class ParticlePropertiesFactory {
    private static Map<String, ParticleProperties> cache = new HashMap<>();
    
    public static ParticleProperties get(String type, Color color, Texture texture, float mass) {
        String key = type;
        
        if (!cache.containsKey(key)) {
            cache.put(key, new ParticleProperties(color, texture, mass));
        }
        return cache.get(key);
    }
}
```

For an explosion with 10,000 particles of the same type, you create 1 ParticleProperties object and 10,000 lightweight Particle objects.

## Intrinsic vs Extrinsic State

The key to Flyweight is correctly identifying what's intrinsic and extrinsic:

**Intrinsic state:**
- Doesn't depend on context
- Can be shared
- Stored in the flyweight

Examples: tree type, character font, particle color

**Extrinsic state:**
- Depends on context
- Cannot be shared
- Passed to flyweight methods or stored in context objects

Examples: tree position, character position, particle velocity

Rule of thumb: if it varies for each instance, it's extrinsic. If it's the same across many instances, it's intrinsic.

## When to Use Flyweight

Use Flyweight when:
- Your application uses a large number of objects
- Storage costs are high due to object quantity
- Most object state can be made extrinsic
- Many objects can be replaced by relatively few shared objects
- The application doesn't depend on object identity

Real scenarios:
- Text editors (character formatting)
- Game engines (trees, enemies, particles)
- GUI frameworks (icons, cursors)
- Caching systems
- String interning (Java does this automatically)

## Flyweight in Java

Java uses Flyweight internally:

### String Interning

```java
String s1 = "hello";
String s2 = "hello";

System.out.println(s1 == s2);  // true - same object!
```

Java reuses string literals. "hello" exists once in memory, referenced twice.

### Integer Caching

```java
Integer i1 = 127;
Integer i2 = 127;

System.out.println(i1 == i2);  // true - cached!

Integer i3 = 128;
Integer i4 = 128;

System.out.println(i3 == i4);  // false - not cached
```

Java caches Integer objects from -128 to 127. Beyond that, new objects are created.

## Thread Safety

If flyweights are shared across threads, ensure thread safety:

```java
class TreeFactory {
    private static final Map<String, TreeType> cache = 
        new ConcurrentHashMap<>();  // Thread-safe
    
    public static TreeType getTreeType(String name, ...) {
        return cache.computeIfAbsent(name, k -> 
            new TreeType(name, ...));  // Atomic operation
    }
}
```

Or use synchronization:

```java
class TreeFactory {
    private static Map<String, TreeType> cache = new HashMap<>();
    
    public static synchronized TreeType getTreeType(String name, ...) {
        // Synchronized method
    }
}
```

## Common Mistakes

### Mistake 1: Making Everything a Flyweight

```java
// BAD: Position is unique, shouldn't be in flyweight
class TreeType {
    private int x, y;  // Wrong! This varies per tree
}
```

Only share what's truly common. Unique state stays in context objects.

### Mistake 2: Not Using a Factory

```java
// BAD: Client creates flyweights directly
TreeType type = new TreeType("Oak", ...);  // Not reusing!
```

Always use a factory to ensure sharing.

### Mistake 3: Mutating Flyweights

```java
// BAD: Changing shared state
class TreeType {
    private Color color;
    
    public void setColor(Color c) {  // Dangerous!
        this.color = c;  // Affects all trees using this flyweight
    }
}
```

Flyweights should be immutable. Changing shared state affects all users.

## Flyweight vs Singleton

**Singleton:** One instance per application.

**Flyweight:** Multiple shared instances, one per distinct set of intrinsic values.

Singleton is about controlling instantiation. Flyweight is about sharing instances.

## Flyweight and SOLID Principles

### Single Responsibility Principle

Flyweight handles shared state. Context handles unique state. Factory handles instance management.

### Open/Closed Principle

Add new flyweight types without modifying existing code.

## The Mental Model

Think of Flyweight like:

**Fonts in a word processor:** The font data (Arial, 12pt, bold) is loaded once. Thousands of characters reference it. Don't store font data 10,000 times, store it once and reference it 10,000 times.

**Stamps in a factory:** The stamp pattern (flyweight) is shared. You press it at different positions (extrinsic state) to create many identical items efficiently.

**Color palette:** A painting uses a limited palette of colors (flyweights). Different parts of the canvas reference these colors at different positions.

## Performance Considerations

Flyweight trades computation for memory:

**Memory saved:** Dramatic reduction when many objects share state.

**CPU cost:** Looking up flyweights in factory (usually negligible).

**Trade-off:** Passing extrinsic state to methods vs storing it.

Profile your application. Use Flyweight when memory, not CPU, is the bottleneck.

## Measuring Impact

Before Flyweight:
```java
// 100,000 trees × (String + 3 Textures + position) = lots of memory
```

After Flyweight:
```java
// 2 TreeType objects + (100,000 × position) = much less memory
```

The difference can be gigabytes in large applications.

## Final Thoughts

The Flyweight pattern is about efficiency. When you have thousands or millions of similar objects, sharing their common state can dramatically reduce memory usage.

It's not about complexity. It's about:
- Identifying what can be shared
- Separating intrinsic from extrinsic state
- Using a factory to manage sharing
- Keeping flyweights immutable

The key insight: many objects are mostly identical. Store the shared parts once, reference them many times.

Remember:
- Intrinsic state is shared and stored in flyweight
- Extrinsic state is unique and passed to methods
- Factory ensures sharing
- Immutability prevents bugs

Next time you're creating thousands of similar objects, stop. Ask yourself: what's truly unique? What's shared? Extract the shared parts into flyweights.

Share the load, save the memory.