# The Composite Pattern: Treating Trees Like Leaves

Ever noticed how you can select a single file in your file explorer, or select a folder (which contains more files and folders), and the "delete" button works the same way? That's the Composite pattern in action.

The pattern lets you treat individual objects and groups of objects uniformly. A single object is treated the same as a collection of objects.

## The Problem: Treating Leaves and Branches Differently

Imagine you're building a graphics editor. You have simple shapes like circles and rectangles. But you also want to group shapes together. A group can contain shapes and other groups.

The naive approach:

```java
class Circle {
    public void draw() {
        System.out.println("Drawing circle");
    }
}

class Rectangle {
    public void draw() {
        System.out.println("Drawing rectangle");
    }
}

class Group {
    private List<Object> children;
    
    public void draw() {
        for (Object child : children) {
            if (child instanceof Circle) {
                ((Circle) child).draw();
            } else if (child instanceof Rectangle) {
                ((Rectangle) child).draw();
            } else if (child instanceof Group) {
                ((Group) child).draw();
            }
            // What if we add Triangle? Modify this code!
        }
    }
}
```

Problems:

**Type checking everywhere:** Lots of `instanceof` checks. Ugly and error-prone.

**Not extensible:** Add a new shape type? Update every place that handles groups.

**Different interfaces:** Can't treat a single shape and a group uniformly.

**Violates Open/Closed:** Adding new types requires modifying existing code.

## The Composite Solution

Create a common interface for both individual objects and collections:

```java
// Component interface
interface Graphic {
    void draw();
    void move(int x, int y);
}

// Leaf: Individual shape
class Circle implements Graphic {
    private int x, y, radius;
    
    public Circle(int x, int y, int radius) {
        this.x = x;
        this.y = y;
        this.radius = radius;
    }
    
    public void draw() {
        System.out.println("Drawing circle at (" + x + "," + y + 
                          ") with radius " + radius);
    }
    
    public void move(int x, int y) {
        this.x += x;
        this.y += y;
        System.out.println("Moving circle");
    }
}

class Rectangle implements Graphic {
    private int x, y, width, height;
    
    public Rectangle(int x, int y, int width, int height) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    
    public void draw() {
        System.out.println("Drawing rectangle at (" + x + "," + y + 
                          ") with size " + width + "x" + height);
    }
    
    public void move(int x, int y) {
        this.x += x;
        this.y += y;
        System.out.println("Moving rectangle");
    }
}

// Composite: Group of shapes
class CompositeGraphic implements Graphic {
    private List<Graphic> children;
    
    public CompositeGraphic() {
        children = new ArrayList<>();
    }
    
    public void add(Graphic graphic) {
        children.add(graphic);
    }
    
    public void remove(Graphic graphic) {
        children.remove(graphic);
    }
    
    public void draw() {
        System.out.println("Drawing composite graphic:");
        for (Graphic child : children) {
            child.draw();  // Polymorphism handles the rest
        }
    }
    
    public void move(int x, int y) {
        System.out.println("Moving composite graphic:");
        for (Graphic child : children) {
            child.move(x, y);  // Move all children
        }
    }
}
```

Usage:

```java
// Create individual shapes
Graphic circle = new Circle(10, 10, 5);
Graphic rectangle = new Rectangle(20, 20, 30, 40);

// Create a group
CompositeGraphic group1 = new CompositeGraphic();
group1.add(circle);
group1.add(rectangle);

// Create more shapes
Graphic circle2 = new Circle(50, 50, 10);

// Create a larger group
CompositeGraphic group2 = new CompositeGraphic();
group2.add(group1);      // Group contains another group
group2.add(circle2);

// Treat everything uniformly
group2.draw();    // Draws everything in the hierarchy
group2.move(5, 5); // Moves everything

// Even a single shape works the same way
circle.draw();
```

No type checking. No special cases. Uniform treatment.

## The Components

### 1. Component Interface

Defines common operations:

```java
interface Graphic {
    void draw();
    void move(int x, int y);
}
```

### 2. Leaf

Represents individual objects with no children:

```java
class Circle implements Graphic {
    public void draw() {
        // Draw this circle
    }
}
```

### 3. Composite

Represents groups that can contain leaves and other composites:

```java
class CompositeGraphic implements Graphic {
    private List<Graphic> children;
    
    public void draw() {
        // Draw all children
        for (Graphic child : children) {
            child.draw();
        }
    }
}
```

### 4. Client

Treats all objects uniformly:

```java
Graphic graphic = getGraphic(); // Could be leaf or composite
graphic.draw(); // Don't care which it is
```

## Real-World Example: File System

Perfect example of Composite pattern:

```java
// Component
interface FileSystemItem {
    void display(int indent);
    int getSize();
}

// Leaf: File
class File implements FileSystemItem {
    private String name;
    private int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    public void display(int indent) {
        System.out.println(" ".repeat(indent) + "- " + name + " (" + size + " KB)");
    }
    
    public int getSize() {
        return size;
    }
}

// Composite: Directory
class Directory implements FileSystemItem {
    private String name;
    private List<FileSystemItem> items;
    
    public Directory(String name) {
        this.name = name;
        this.items = new ArrayList<>();
    }
    
    public void add(FileSystemItem item) {
        items.add(item);
    }
    
    public void remove(FileSystemItem item) {
        items.remove(item);
    }
    
    public void display(int indent) {
        System.out.println(" ".repeat(indent) + "+ " + name + "/");
        for (FileSystemItem item : items) {
            item.display(indent + 2);
        }
    }
    
    public int getSize() {
        int totalSize = 0;
        for (FileSystemItem item : items) {
            totalSize += item.getSize();
        }
        return totalSize;
    }
}
```

Usage:

```java
// Create files
File file1 = new File("document.txt", 10);
File file2 = new File("image.jpg", 250);
File file3 = new File("video.mp4", 5000);

// Create directories
Directory root = new Directory("root");
Directory docs = new Directory("documents");
Directory media = new Directory("media");

// Build hierarchy
docs.add(file1);
media.add(file2);
media.add(file3);
root.add(docs);
root.add(media);

// Use uniformly
root.display(0);
System.out.println("Total size: " + root.getSize() + " KB");

// Output:
// + root/
//   + documents/
//     - document.txt (10 KB)
//   + media/
//     - image.jpg (250 KB)
//     - video.mp4 (5000 KB)
// Total size: 5260 KB
```

Notice: calling `getSize()` on a directory recursively calculates the size of all children. Calling it on a file just returns the file size. Same interface, different behavior.

## Another Example: Organization Hierarchy

```java
// Component
interface Employee {
    void showDetails();
    double getSalary();
}

// Leaf: Individual contributor
class Developer implements Employee {
    private String name;
    private double salary;
    
    public Developer(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public void showDetails() {
        System.out.println("Developer: " + name + " ($" + salary + ")");
    }
    
    public double getSalary() {
        return salary;
    }
}

class Designer implements Employee {
    private String name;
    private double salary;
    
    public Designer(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public void showDetails() {
        System.out.println("Designer: " + name + " ($" + salary + ")");
    }
    
    public double getSalary() {
        return salary;
    }
}

// Composite: Manager with team
class Manager implements Employee {
    private String name;
    private double salary;
    private List<Employee> team;
    
    public Manager(String name, double salary) {
        this.name = name;
        this.salary = salary;
        this.team = new ArrayList<>();
    }
    
    public void addTeamMember(Employee employee) {
        team.add(employee);
    }
    
    public void showDetails() {
        System.out.println("Manager: " + name + " ($" + salary + ")");
        System.out.println("  Team:");
        for (Employee employee : team) {
            employee.showDetails();
        }
    }
    
    public double getSalary() {
        double totalSalary = salary;
        for (Employee employee : team) {
            totalSalary += employee.getSalary();
        }
        return totalSalary;
    }
}
```

Usage:

```java
// Individual contributors
Employee dev1 = new Developer("Alice", 80000);
Employee dev2 = new Developer("Bob", 75000);
Employee designer = new Designer("Charlie", 70000);

// Team lead
Manager teamLead = new Manager("Dave", 100000);
teamLead.addTeamMember(dev1);
teamLead.addTeamMember(dev2);

// Department head
Manager deptHead = new Manager("Eve", 150000);
deptHead.addTeamMember(teamLead);
deptHead.addTeamMember(designer);

// Show hierarchy
deptHead.showDetails();

// Calculate total cost
System.out.println("Total salary cost: $" + deptHead.getSalary());
```

## Tree Traversal Strategies

Composite creates tree structures. You can traverse them different ways:

### Depth-First (Pre-order)

```java
class CompositeGraphic implements Graphic {
    public void draw() {
        System.out.println("Drawing group");  // Process node first
        for (Graphic child : children) {
            child.draw();  // Then children
        }
    }
}
```

### Depth-First (Post-order)

```java
class CompositeGraphic implements Graphic {
    public void draw() {
        for (Graphic child : children) {
            child.draw();  // Process children first
        }
        System.out.println("Drawing group");  // Then node
    }
}
```

### Breadth-First

```java
class CompositeGraphic implements Graphic {
    public void draw() {
        Queue<Graphic> queue = new LinkedList<>();
        queue.add(this);
        
        while (!queue.isEmpty()) {
            Graphic current = queue.poll();
            
            if (current instanceof CompositeGraphic) {
                CompositeGraphic composite = (CompositeGraphic) current;
                queue.addAll(composite.children);
            }
            
            current.draw();
        }
    }
}
```

Choose based on your needs. Most composites use depth-first.

## When to Use Composite

Use Composite when:
- You have part-whole hierarchies
- You want clients to treat individual objects and compositions uniformly
- You're building tree structures
- You want to ignore the difference between leaves and branches

Real scenarios:
- File systems (files and directories)
- GUI components (buttons, panels containing buttons)
- Graphics (shapes and groups of shapes)
- Organization charts (employees and managers)
- Menu systems (items and submenus)
- HTML DOM (elements and containers)
- Expression trees (operands and operators)

## Safety vs Transparency Trade-off

There are two approaches to the Composite interface:

### Transparent (What we've been using)

Component interface includes child management operations:

```java
interface Graphic {
    void draw();
    void add(Graphic g);     // In interface
    void remove(Graphic g);  // In interface
}
```

**Pros:** Uniform interface. Client treats everything the same.

**Cons:** Leaves must implement child operations (usually as no-ops or exceptions).

### Safe

Component interface doesn't include child management:

```java
interface Graphic {
    void draw();
    // No add/remove
}

class CompositeGraphic implements Graphic {
    public void add(Graphic g) { }     // Only in composite
    public void remove(Graphic g) { }  // Only in composite
}
```

**Pros:** Leaves don't need meaningless operations. Type-safe.

**Cons:** Client must know if something is a composite to add children.

Most implementations use the safe approach to avoid confusing leaf operations.

## Common Mistakes

### Mistake 1: Forgetting to Implement Component Methods in Composite

```java
// BAD
class CompositeGraphic implements Graphic {
    public void draw() {
        // Forgot to iterate over children!
        System.out.println("Drawing group");
    }
}
```

Composites must delegate to children.

### Mistake 2: Reference Loops

```java
// BAD: Creating cycles
CompositeGraphic group1 = new CompositeGraphic();
CompositeGraphic group2 = new CompositeGraphic();

group1.add(group2);
group2.add(group1);  // Cycle!

group1.draw();  // Infinite loop!
```

Prevent cycles or detect them during traversal.

### Mistake 3: Not Handling Parent References

```java
// If you need parent references, maintain them
class CompositeGraphic implements Graphic {
    private Graphic parent;
    
    public void add(Graphic g) {
        children.add(g);
        if (g instanceof CompositeGraphic) {
            ((CompositeGraphic) g).parent = this;
        }
    }
}
```

Useful for navigation but adds complexity.

## Composite and SOLID Principles

### Single Responsibility Principle

Leaves handle individual behavior. Composites handle collection behavior. Separate responsibilities.

### Open/Closed Principle

Add new component types without modifying existing code.

### Liskov Substitution Principle

Leaves and composites are both components. Can be substituted anywhere.

## The Mental Model

Think of Composite like:

**Russian nesting dolls:** Each doll can be treated as a single item. But larger dolls contain smaller dolls. The action "open the doll" works on any doll, whether it contains more dolls or not.

**Company organization:** An employee might be an individual or a manager with a team. "Calculate salary cost" works the same way - individual returns their salary, manager returns team total.

**Folders on your computer:** You can copy a file or copy a folder. The operation works the same. The folder just needs to recursively copy its contents.

## Performance Considerations

Composite operations are recursive. Be aware of:

- Deep hierarchies can cause stack overflow
- Operations on large trees can be slow
- Consider caching computed values (like total size)

Example with caching:

```java
class Directory implements FileSystemItem {
    private Integer cachedSize;  // Cache the size
    
    public void add(FileSystemItem item) {
        items.add(item);
        cachedSize = null;  // Invalidate cache
    }
    
    public int getSize() {
        if (cachedSize == null) {
            cachedSize = 0;
            for (FileSystemItem item : items) {
                cachedSize += item.getSize();
            }
        }
        return cachedSize;
    }
}
```

## Testing with Composite

Composite makes testing straightforward:

```java
@Test
public void testLeaf() {
    Graphic circle = new Circle(10, 10, 5);
    circle.draw();
    // Test individual behavior
}

@Test
public void testComposite() {
    CompositeGraphic group = new CompositeGraphic();
    group.add(new Circle(10, 10, 5));
    group.add(new Rectangle(20, 20, 30, 40));
    
    group.draw();
    // Verify all children were drawn
}

@Test
public void testNestedComposite() {
    CompositeGraphic outer = new CompositeGraphic();
    CompositeGraphic inner = new CompositeGraphic();
    
    inner.add(new Circle(10, 10, 5));
    outer.add(inner);
    
    outer.draw();
    // Verify nested structure works
}
```

## Final Thoughts

The Composite pattern is about treating individual objects and groups of objects uniformly. It's particularly useful for tree structures where you want operations to work recursively.

It's not about complexity. It's about:
- Eliminating type checks
- Creating uniform interfaces
- Building recursive structures
- Following SOLID principles

The key insight: if something can be either a single item or a collection of items, and you want to treat both the same way, use Composite.

Remember:
- Component defines the interface
- Leaf implements individual behavior
- Composite implements collection behavior
- Client treats both uniformly

Next time you're writing `if (x instanceof Y)` to handle individual objects differently from collections, stop. Think Composite. Create a common interface and let polymorphism handle the rest.

Think trees, not branches.