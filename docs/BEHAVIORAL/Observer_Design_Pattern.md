# The Observer Pattern: The Pub-Sub You've Been Using All Along

Ever subscribed to a YouTube channel? You hit subscribe, and boom, you get notified when they upload new content. You didn't have to check their channel every hour. They notify you automatically.

That's the Observer pattern. One object (the subject) maintains a list of dependents (observers) and notifies them automatically when its state changes.

## The Problem: Tight Coupling and Polling

Imagine you're building a weather station. Multiple displays need to show the weather data: current conditions, statistics, and forecasts.

The naive approach:

```java
class WeatherStation {
    private float temperature;
    private float humidity;
    private float pressure;
    
    public void measurementsChanged() {
        float temp = getTemperature();
        float humidity = getHumidity();
        float pressure = getPressure();
        
        // Directly update all displays
        currentConditionsDisplay.update(temp, humidity, pressure);
        statisticsDisplay.update(temp, humidity, pressure);
        forecastDisplay.update(temp, humidity, pressure);
    }
}
```

Problems:

**Tightly coupled:** WeatherStation knows about every display. Add a new display? Modify WeatherStation. Remove a display? Modify WeatherStation.

**Violates Open/Closed:** Can't add displays without changing existing code.

**Not flexible:** Can't add/remove displays at runtime.

**Hard to test:** Can't test displays independently.

What if displays could just "subscribe" to the weather station and get notified automatically when data changes?

## The Observer Solution

The Observer pattern creates a subscription mechanism:

```java
// Observer interface
interface Observer {
    void update(float temperature, float humidity, float pressure);
}

// Subject interface
interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}

// Concrete Subject
class WeatherStation implements Subject {
    private List<Observer> observers;
    private float temperature;
    private float humidity;
    private float pressure;
    
    public WeatherStation() {
        observers = new ArrayList<>();
    }
    
    public void registerObserver(Observer o) {
        observers.add(o);
    }
    
    public void removeObserver(Observer o) {
        observers.remove(o);
    }
    
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }
    
    public void measurementsChanged() {
        notifyObservers();
    }
    
    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
}

// Concrete Observers
class CurrentConditionsDisplay implements Observer {
    private float temperature;
    private float humidity;
    
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }
    
    public void display() {
        System.out.println("Current conditions: " + temperature + 
                          "F degrees and " + humidity + "% humidity");
    }
}

class StatisticsDisplay implements Observer {
    private List<Float> temperatureHistory = new ArrayList<>();
    
    public void update(float temperature, float humidity, float pressure) {
        temperatureHistory.add(temperature);
        display();
    }
    
    public void display() {
        float avg = (float) temperatureHistory.stream()
                                             .mapToDouble(Float::doubleValue)
                                             .average()
                                             .orElse(0.0);
        System.out.println("Avg temperature: " + avg + "F");
    }
}
```

Usage:

```java
WeatherStation weatherStation = new WeatherStation();

CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay();
StatisticsDisplay statsDisplay = new StatisticsDisplay();

// Subscribe to weather updates
weatherStation.registerObserver(currentDisplay);
weatherStation.registerObserver(statsDisplay);

// Update weather
weatherStation.setMeasurements(80, 65, 30.4f);
// Both displays automatically update

weatherStation.setMeasurements(82, 70, 29.2f);
// Both displays automatically update again

// Unsubscribe
weatherStation.removeObserver(statsDisplay);

weatherStation.setMeasurements(78, 90, 29.2f);
// Only current display updates
```

The weather station doesn't know about specific display types. It just notifies all registered observers. Clean separation. Easy to extend.

## The Components

### 1. Subject (Observable)

The object being observed. Maintains a list of observers and notifies them of changes:

```java
interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}
```

### 2. Observer

Objects that want to be notified of changes:

```java
interface Observer {
    void update(/* state data */);
}
```

### 3. Concrete Subject

Implements the subject interface and holds the state:

```java
class WeatherStation implements Subject {
    private List<Observer> observers;
    private float temperature;
    // ...
}
```

### 4. Concrete Observers

Implement the observer interface and react to updates:

```java
class CurrentConditionsDisplay implements Observer {
    public void update(float temp, float humidity, float pressure) {
        // Update display
    }
}
```

## Real-World Example: Stock Market Ticker

Multiple investors want to track stock prices:

```java
interface StockObserver {
    void update(String stock, double price);
}

interface StockMarket {
    void registerObserver(StockObserver observer, String stock);
    void removeObserver(StockObserver observer, String stock);
    void notifyObservers(String stock);
}

class StockExchange implements StockMarket {
    // Map of stock symbols to their observers
    private Map<String, List<StockObserver>> observers;
    private Map<String, Double> stockPrices;
    
    public StockExchange() {
        observers = new HashMap<>();
        stockPrices = new HashMap<>();
    }
    
    public void registerObserver(StockObserver observer, String stock) {
        observers.computeIfAbsent(stock, k -> new ArrayList<>()).add(observer);
    }
    
    public void removeObserver(StockObserver observer, String stock) {
        List<StockObserver> stockObservers = observers.get(stock);
        if (stockObservers != null) {
            stockObservers.remove(observer);
        }
    }
    
    public void notifyObservers(String stock) {
        List<StockObserver> stockObservers = observers.get(stock);
        if (stockObservers != null) {
            double price = stockPrices.get(stock);
            for (StockObserver observer : stockObservers) {
                observer.update(stock, price);
            }
        }
    }
    
    public void setStockPrice(String stock, double price) {
        stockPrices.put(stock, price);
        notifyObservers(stock);
    }
}

class Investor implements StockObserver {
    private String name;
    
    public Investor(String name) {
        this.name = name;
    }
    
    public void update(String stock, double price) {
        System.out.println(name + " notified: " + stock + " is now $" + price);
        
        // Investment logic
        if (price < 50) {
            System.out.println(name + " buying " + stock);
        }
    }
}

class PortfolioTracker implements StockObserver {
    private Map<String, Double> portfolio = new HashMap<>();
    
    public void update(String stock, double price) {
        portfolio.put(stock, price);
        System.out.println("Portfolio updated: " + stock + " = $" + price);
        calculateTotal();
    }
    
    private void calculateTotal() {
        double total = portfolio.values().stream()
                                .mapToDouble(Double::doubleValue)
                                .sum();
        System.out.println("Total portfolio value: $" + total);
    }
}
```

Usage:

```java
StockExchange exchange = new StockExchange();

Investor alice = new Investor("Alice");
Investor bob = new Investor("Bob");
PortfolioTracker tracker = new PortfolioTracker();

// Alice watches AAPL and GOOGL
exchange.registerObserver(alice, "AAPL");
exchange.registerObserver(alice, "GOOGL");

// Bob watches AAPL
exchange.registerObserver(bob, "AAPL");

// Tracker watches everything
exchange.registerObserver(tracker, "AAPL");
exchange.registerObserver(tracker, "GOOGL");

// Update stock prices
exchange.setStockPrice("AAPL", 150.0);
// Alice, Bob, and tracker all notified

exchange.setStockPrice("GOOGL", 2800.0);
// Only Alice and tracker notified
```

## Push vs Pull Model

### Push Model (What We've Been Using)

The subject pushes all data to observers:

```java
interface Observer {
    void update(float temperature, float humidity, float pressure);
}

// Subject pushes all data
notifyObservers() {
    for (Observer o : observers) {
        o.update(temperature, humidity, pressure);
    }
}
```

**Pros:** Observers get all data without asking
**Cons:** Observers might not need all data, creates coupling

### Pull Model

Observers pull data they need:

```java
interface Observer {
    void update(Subject subject);
}

class CurrentConditionsDisplay implements Observer {
    public void update(Subject subject) {
        WeatherStation station = (WeatherStation) subject;
        // Pull only what we need
        float temp = station.getTemperature();
        float humidity = station.getHumidity();
        display();
    }
}
```

**Pros:** Observers get only what they need, less coupling
**Cons:** Observers need to know about subject type

Most real-world implementations use a hybrid approach or pure push for simplicity.

## Another Example: Event System

```java
interface EventListener {
    void onEvent(Event event);
}

class Event {
    private String type;
    private Object data;
    
    public Event(String type, Object data) {
        this.type = type;
        this.data = data;
    }
    
    public String getType() { return type; }
    public Object getData() { return data; }
}

class EventManager {
    private Map<String, List<EventListener>> listeners;
    
    public EventManager() {
        listeners = new HashMap<>();
    }
    
    public void subscribe(String eventType, EventListener listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }
    
    public void unsubscribe(String eventType, EventListener listener) {
        List<EventListener> eventListeners = listeners.get(eventType);
        if (eventListeners != null) {
            eventListeners.remove(listener);
        }
    }
    
    public void notify(Event event) {
        List<EventListener> eventListeners = listeners.get(event.getType());
        if (eventListeners != null) {
            for (EventListener listener : eventListeners) {
                listener.onEvent(event);
            }
        }
    }
}

// Usage in a game
class Game {
    private EventManager eventManager;
    
    public Game() {
        eventManager = new EventManager();
        setupListeners();
    }
    
    private void setupListeners() {
        // Achievement system listens to player events
        eventManager.subscribe("PLAYER_KILL", event -> {
            System.out.println("Achievement unlocked!");
        });
        
        // UI listens to score events
        eventManager.subscribe("SCORE_CHANGED", event -> {
            int score = (int) event.getData();
            System.out.println("Score: " + score);
        });
        
        // Audio system listens to game events
        eventManager.subscribe("PLAYER_KILL", event -> {
            System.out.println("Playing kill sound");
        });
    }
    
    public void playerKilledEnemy() {
        // Notify all listeners
        eventManager.notify(new Event("PLAYER_KILL", null));
        eventManager.notify(new Event("SCORE_CHANGED", 100));
    }
}
```

## Java's Built-In Observer (Legacy)

Java has `Observable` class and `Observer` interface (though deprecated as of Java 9):

```java
import java.util.Observable;
import java.util.Observer;

class WeatherData extends Observable {
    private float temperature;
    
    public void setTemperature(float temp) {
        this.temperature = temp;
        setChanged();  // Mark as changed
        notifyObservers();  // Notify observers
    }
    
    public float getTemperature() {
        return temperature;
    }
}

class Display implements Observer {
    public void update(Observable o, Object arg) {
        if (o instanceof WeatherData) {
            WeatherData data = (WeatherData) o;
            System.out.println("Temperature: " + data.getTemperature());
        }
    }
}
```

**Why deprecated?** `Observable` is a class, not an interface. Limits flexibility (can't extend another class). Modern Java uses listeners and event systems instead.

## Modern Alternatives

### Property Change Listeners (Java Beans)

```java
import java.beans.PropertyChangeListener;
import java.beans.PropertyChangeSupport;

class WeatherStation {
    private PropertyChangeSupport support;
    private float temperature;
    
    public WeatherStation() {
        support = new PropertyChangeSupport(this);
    }
    
    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }
    
    public void setTemperature(float temperature) {
        float oldTemp = this.temperature;
        this.temperature = temperature;
        support.firePropertyChange("temperature", oldTemp, temperature);
    }
}

// Usage
station.addPropertyChangeListener(evt -> {
    System.out.println("Temperature changed from " + evt.getOldValue() + 
                      " to " + evt.getNewValue());
});
```

### Reactive Programming (RxJava)

Modern approach using observables:

```java
// Using RxJava
Observable<Integer> observable = Observable.just(1, 2, 3, 4, 5);

observable.subscribe(
    item -> System.out.println("Received: " + item),
    error -> System.err.println("Error: " + error),
    () -> System.out.println("Complete")
);
```

## When to Use Observer

Use Observer when:
- Changes to one object require changes to others
- You don't know how many objects need to be updated
- Objects should be loosely coupled
- An abstraction has two aspects, one dependent on the other

Common scenarios:
- Event handling systems
- MVC architecture (Model notifies Views)
- Publish-subscribe messaging
- Stock tickers
- Social media feeds
- Notification systems
- Real-time dashboards

## Common Pitfalls

### Pitfall 1: Memory Leaks

```java
// BAD: Observer never unregisters
class Display implements Observer {
    public Display(WeatherStation station) {
        station.registerObserver(this);
        // If Display is destroyed but never unregisters,
        // station holds a reference, preventing garbage collection
    }
}
```

**Solution:** Always unregister when done:

```java
class Display implements Observer {
    private WeatherStation station;
    
    public Display(WeatherStation station) {
        this.station = station;
        station.registerObserver(this);
    }
    
    public void destroy() {
        station.removeObserver(this);
    }
}
```

### Pitfall 2: Update Order Dependency

```java
// BAD: Observer B depends on Observer A being updated first
class ObserverA implements Observer {
    public void update() {
        data.processFirst();
    }
}

class ObserverB implements Observer {
    public void update() {
        // Assumes A already ran!
        data.processSecond();
    }
}
```

Observers should be independent. If order matters, you probably need a different pattern.

### Pitfall 3: Notification Cascades

```java
// BAD: Observer triggers another notification
class ObserverA implements Observer {
    public void update() {
        subject.setState(newState);  // Triggers another notification!
    }
}
```

Can create infinite loops or performance issues. Be careful with observers modifying the subject.

## Observer and SOLID Principles

### Single Responsibility Principle

Subject manages observers. Observers handle updates. Separate responsibilities.

### Open/Closed Principle

Add new observers without modifying the subject. Open for extension, closed for modification.

### Dependency Inversion Principle

Subject depends on Observer interface, not concrete observers. Observers depend on Subject interface, not concrete subjects.

## The Mental Model

Think of Observer like:

**Newsletter subscription:** You subscribe to a newsletter. When new content is published, you get an email. You didn't check the website every hour. They notified you.

**YouTube notifications:** Subscribe to a channel. Get notified of new videos. The channel doesn't know who you are specifically, just that you're subscribed.

**Fire alarm:** Fire starts. Alarm goes off. Everyone (observers) hears it and reacts. The alarm doesn't know who's in the building.

## Performance Considerations

Observer has minimal overhead for small numbers of observers. With hundreds or thousands:

- Consider async notifications
- Batch updates instead of notifying for every change
- Use weak references to prevent memory leaks
- Consider thread safety (covered next)

## Thread Safety

If multiple threads modify the subject or observers:

```java
class ThreadSafeSubject {
    private final List<Observer> observers = 
        Collections.synchronizedList(new ArrayList<>());
    
    public void registerObserver(Observer o) {
        observers.add(o);
    }
    
    public void notifyObservers() {
        // Copy list to avoid ConcurrentModificationException
        List<Observer> observersCopy;
        synchronized(observers) {
            observersCopy = new ArrayList<>(observers);
        }
        
        for (Observer observer : observersCopy) {
            observer.update();
        }
    }
}
```

Or use `CopyOnWriteArrayList` for better read performance.

## Final Thoughts

The Observer pattern is everywhere. Every GUI framework, event system, and reactive library uses this concept.

It's about decoupling. The subject doesn't know about specific observers. Observers don't know about each other. Changes propagate automatically.

Key insights:
- One-to-many dependency
- Automatic notification
- Loose coupling
- Dynamic subscriptions

Remember:
- Register and unregister properly
- Keep observers independent
- Watch for memory leaks
- Consider threading

Next time you're tempted to directly call multiple objects when something changes, stop. Think Observer. Let them subscribe and get notified automatically.

Keep watching.