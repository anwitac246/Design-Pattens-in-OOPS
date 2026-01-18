# The Iterator Pattern: Traversing Collections Without Exposing Guts

Ever used a for-each loop in Java? Ever called `next()` on a list? Congratulations, you've been using the Iterator pattern all along.

The idea: provide a way to access elements of a collection sequentially without exposing the collection's internal structure.

## The Problem: Exposing Internal Structure

You've built a custom book collection:

```java
class BookCollection {
    private Book[] books;
    private int index = 0;
    
    public BookCollection(int size) {
        books = new Book[size];
    }
    
    public void addBook(Book book) {
        books[index++] = book;
    }
    
    public Book[] getBooks() {
        return books;  // Exposing internal array!
    }
}
```

Client code:

```java
BookCollection collection = new BookCollection(10);
// Add books...

// Client needs to know it's an array
Book[] books = collection.getBooks();
for (int i = 0; i < books.length; i++) {
    if (books[i] != null) {  // Client handles nulls
        System.out.println(books[i].getTitle());
    }
}
```

Problems:

**Exposes internals:** Client knows the collection uses an array. What if you change to a LinkedList? All client code breaks.

**Tight coupling:** Client code depends on implementation details.

**Inconsistent iteration:** Different collections have different iteration logic. Arrays use indices, LinkedLists use nodes.

**No encapsulation:** Clients can modify the internal array.

What if collections could provide a standard way to iterate without revealing their structure?

## The Iterator Solution

Provide an iterator object that handles traversal:

```java
// Iterator interface
interface Iterator {
    boolean hasNext();
    Object next();
}

// Aggregate interface
interface Aggregate {
    Iterator createIterator();
}

// Concrete iterator
class BookIterator implements Iterator {
    private Book[] books;
    private int position = 0;
    
    public BookIterator(Book[] books) {
        this.books = books;
    }
    
    public boolean hasNext() {
        return position < books.length && books[position] != null;
    }
    
    public Object next() {
        Book book = books[position];
        position++;
        return book;
    }
}

// Concrete aggregate
class BookCollection implements Aggregate {
    private Book[] books;
    private int index = 0;
    
    public BookCollection(int size) {
        books = new Book[size];
    }
    
    public void addBook(Book book) {
        books[index++] = book;
    }
    
    public Iterator createIterator() {
        return new BookIterator(books);
    }
}
```

Usage:

```java
BookCollection collection = new BookCollection(10);
collection.addBook(new Book("1984"));
collection.addBook(new Book("Brave New World"));
collection.addBook(new Book("Fahrenheit 451"));

// Client doesn't know about internal structure
Iterator iterator = collection.createIterator();
while (iterator.hasNext()) {
    Book book = (Book) iterator.next();
    System.out.println(book.getTitle());
}
```

The client doesn't know or care whether it's an array, list, tree, or hash table. The iterator handles traversal.

## The Components

### 1. Iterator Interface

Defines traversal interface:

```java
interface Iterator<T> {
    boolean hasNext();
    T next();
    void remove();  // Optional
}
```

### 2. Concrete Iterator

Implements traversal for a specific collection:

```java
class BookIterator implements Iterator<Book> {
    private Book[] books;
    private int position;
    
    public boolean hasNext() { /* ... */ }
    public Book next() { /* ... */ }
}
```

### 3. Aggregate Interface

Defines method to create iterator:

```java
interface Aggregate<T> {
    Iterator<T> createIterator();
}
```

### 4. Concrete Aggregate

Creates and returns appropriate iterator:

```java
class BookCollection implements Aggregate<Book> {
    public Iterator<Book> createIterator() {
        return new BookIterator(books);
    }
}
```

## Java's Built-In Iterator

Java provides the Iterator interface:

```java
import java.util.Iterator;

class BookCollection implements Iterable<Book> {
    private List<Book> books = new ArrayList<>();
    
    public void addBook(Book book) {
        books.add(book);
    }
    
    @Override
    public Iterator<Book> iterator() {
        return books.iterator();  // Delegate to ArrayList's iterator
    }
}
```

Usage:

```java
BookCollection collection = new BookCollection();
collection.addBook(new Book("1984"));

// Enhanced for loop (syntactic sugar for iterator)
for (Book book : collection) {
    System.out.println(book.getTitle());
}

// Or explicitly
Iterator<Book> iterator = collection.iterator();
while (iterator.hasNext()) {
    Book book = iterator.next();
    System.out.println(book.getTitle());
}
```

Implementing `Iterable<T>` enables the enhanced for loop syntax.

## Real-World Example: Custom Binary Tree Iterator

```java
class TreeNode {
    int value;
    TreeNode left;
    TreeNode right;
    
    public TreeNode(int value) {
        this.value = value;
    }
}

class BinaryTree implements Iterable<Integer> {
    private TreeNode root;
    
    public BinaryTree(TreeNode root) {
        this.root = root;
    }
    
    @Override
    public Iterator<Integer> iterator() {
        return new InOrderIterator(root);
    }
    
    // Inner class: In-order iterator
    private class InOrderIterator implements Iterator<Integer> {
        private Stack<TreeNode> stack = new Stack<>();
        
        public InOrderIterator(TreeNode root) {
            pushLeft(root);
        }
        
        private void pushLeft(TreeNode node) {
            while (node != null) {
                stack.push(node);
                node = node.left;
            }
        }
        
        @Override
        public boolean hasNext() {
            return !stack.isEmpty();
        }
        
        @Override
        public Integer next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            
            TreeNode node = stack.pop();
            pushLeft(node.right);
            return node.value;
        }
    }
}
```

Usage:

```java
//       5
//      / \
//     3   7
//    / \
//   2   4
TreeNode root = new TreeNode(5);
root.left = new TreeNode(3);
root.right = new TreeNode(7);
root.left.left = new TreeNode(2);
root.left.right = new TreeNode(4);

BinaryTree tree = new BinaryTree(root);

// In-order traversal: 2, 3, 4, 5, 7
for (int value : tree) {
    System.out.println(value);
}
```

The iterator hides the complexity of tree traversal. Client just calls `next()`.

## Multiple Iterator Types

One collection can have multiple iterators:

```java
class BinaryTree {
    private TreeNode root;
    
    public Iterator<Integer> inOrderIterator() {
        return new InOrderIterator(root);
    }
    
    public Iterator<Integer> preOrderIterator() {
        return new PreOrderIterator(root);
    }
    
    public Iterator<Integer> postOrderIterator() {
        return new PostOrderIterator(root);
    }
    
    public Iterator<Integer> levelOrderIterator() {
        return new LevelOrderIterator(root);
    }
}
```

Usage:

```java
BinaryTree tree = new BinaryTree(root);

Iterator<Integer> inOrder = tree.inOrderIterator();
Iterator<Integer> preOrder = tree.preOrderIterator();

// Different traversal orders from same tree
```

## Another Example: Range Iterator

```java
class Range implements Iterable<Integer> {
    private int start;
    private int end;
    private int step;
    
    public Range(int start, int end, int step) {
        this.start = start;
        this.end = end;
        this.step = step;
    }
    
    @Override
    public Iterator<Integer> iterator() {
        return new RangeIterator();
    }
    
    private class RangeIterator implements Iterator<Integer> {
        private int current = start;
        
        @Override
        public boolean hasNext() {
            return current < end;
        }
        
        @Override
        public Integer next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            int value = current;
            current += step;
            return value;
        }
    }
}
```

Usage:

```java
// Python-like range
Range range = new Range(0, 10, 2);  // 0, 2, 4, 6, 8

for (int i : range) {
    System.out.println(i);
}
```

## Internal vs External Iterators

### External Iterator (What we've been using)

Client controls iteration:

```java
Iterator<Book> iterator = collection.iterator();
while (iterator.hasNext()) {
    Book book = iterator.next();
    // Client controls when to call next()
}
```

**Pros:** More control, can stop/start/skip

**Cons:** More code, easier to misuse

### Internal Iterator

Collection controls iteration:

```java
interface Aggregate<T> {
    void forEach(Consumer<T> action);
}

class BookCollection implements Aggregate<Book> {
    private List<Book> books;
    
    public void forEach(Consumer<Book> action) {
        for (Book book : books) {
            action.accept(book);
        }
    }
}
```

Usage:

```java
collection.forEach(book -> System.out.println(book.getTitle()));
```

**Pros:** Simpler client code, harder to misuse

**Cons:** Less control, harder to break out early

Java's Stream API uses internal iteration:

```java
collection.stream()
    .filter(book -> book.getYear() > 2000)
    .forEach(book -> System.out.println(book.getTitle()));
```

## Concurrent Modification

What happens if the collection changes during iteration?

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");

Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    list.remove(item);  // ConcurrentModificationException!
}
```

Java's iterators are **fail-fast**: they throw `ConcurrentModificationException` if the collection is modified during iteration (except through the iterator's own `remove()` method).

Safe way:

```java
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    iterator.remove();  // Safe
}
```

Or use `CopyOnWriteArrayList` for concurrent access.

## When to Use Iterator

Use Iterator when:
- You need to traverse a collection without exposing its structure
- You want to provide multiple traversal methods
- You want a uniform interface for different collection types
- You need to traverse the same collection in different ways

Real scenarios:
- Collections (List, Set, Map)
- Tree traversals (in-order, pre-order, post-order)
- Graph traversals (BFS, DFS)
- Custom data structures
- File system navigation
- Database result sets

## Iterator vs Visitor

**Iterator:** Focuses on traversal. Client processes each element.

**Visitor:** Focuses on operations. Visitor performs operations on elements.

**Iterator:**
```java
for (Element e : collection) {
    // Client code processes element
}
```

**Visitor:**
```java
collection.accept(visitor);
// Visitor processes each element
```

## Common Pitfalls

### Pitfall 1: Concurrent Modification

```java
// BAD
for (Book book : collection) {
    if (someCondition) {
        collection.remove(book);  // Exception!
    }
}

// GOOD
Iterator<Book> iterator = collection.iterator();
while (iterator.hasNext()) {
    Book book = iterator.next();
    if (someCondition) {
        iterator.remove();  // Safe
    }
}
```

### Pitfall 2: Multiple Active Iterators

```java
// Careful with multiple iterators on same mutable collection
Iterator<Book> it1 = collection.iterator();
Iterator<Book> it2 = collection.iterator();

it1.next();
collection.add(new Book());  // Invalidates both iterators
it2.next();  // May throw exception
```

### Pitfall 3: Forgetting hasNext()

```java
// BAD
Iterator<Book> iterator = collection.iterator();
Book book = iterator.next();  // What if collection is empty?

// GOOD
if (iterator.hasNext()) {
    Book book = iterator.next();
}
```

## Iterator and SOLID Principles

### Single Responsibility Principle

Collection manages data. Iterator manages traversal. Separate responsibilities.

### Open/Closed Principle

Add new iterator types without modifying collection.

### Liskov Substitution Principle

Any iterator can replace another anywhere Iterator interface is expected.

### Dependency Inversion Principle

Client depends on Iterator interface, not concrete iterators.

## The Mental Model

Think of Iterator like:

**A tour guide:** You're visiting a museum. The guide (iterator) leads you from exhibit to exhibit. You don't need to know the museum's layout. Just follow the guide and call `next()`.

**Reading a book:** You read page by page. `hasNext()` checks if there are more pages. `next()` turns the page. You don't care how pages are physically stored.

**Netflix autoplay:** Watching episodes. `hasNext()` checks if there's another episode. `next()` loads it. You don't know how episodes are stored on servers.

## Performance Considerations

Iterators typically have:
- O(1) space overhead (just tracking position)
- O(1) time per `next()` call for simple structures
- O(log n) or O(n) for complex structures like trees

The overhead is usually negligible compared to the benefits.

## Testing with Iterator

```java
@Test
public void testIteratorTraversesAllElements() {
    BookCollection collection = new BookCollection();
    collection.addBook(new Book("A"));
    collection.addBook(new Book("B"));
    
    Iterator<Book> iterator = collection.iterator();
    
    assertTrue(iterator.hasNext());
    assertEquals("A", iterator.next().getTitle());
    assertTrue(iterator.hasNext());
    assertEquals("B", iterator.next().getTitle());
    assertFalse(iterator.hasNext());
}

@Test
public void testIteratorRemove() {
    List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
    Iterator<String> iterator = list.iterator();
    
    iterator.next();
    iterator.remove();
    
    assertEquals(2, list.size());
    assertEquals("B", list.get(0));
}
```

## Real-World Usage

Iterator is everywhere in Java:

```java
// Collections
List<String> list = new ArrayList<>();
Iterator<String> it = list.iterator();

// Enhanced for loop (uses iterator internally)
for (String s : list) { }

// Streams (internal iteration)
list.stream().forEach(System.out::println);

// Database results
ResultSet rs = statement.executeQuery("SELECT * FROM users");
while (rs.next()) {  // Iterator pattern
    String name = rs.getString("name");
}
```

## Final Thoughts

The Iterator pattern is about separation of concerns. Collections handle data storage. Iterators handle traversal. Neither needs to know about the other's internals.

It's not about complexity. It's about:
- Hiding implementation details
- Providing uniform traversal interface
- Supporting multiple traversal types
- Enabling the enhanced for loop

The key insight: clients shouldn't know or care how collections store data. They just want to visit each element.

Remember:
- Iterator handles traversal logic
- Collection creates appropriate iterator
- Client uses standard interface
- Multiple iterator types possible

Next time you're exposing a collection's internals just to let clients traverse it, stop. Think Iterator. Provide a standard traversal interface.

Keep iterating.