# 05 · Design Patterns in Java

Design patterns are named, reusable solutions to recurring design problems.
They're not libraries to import — they're shapes of code you'll recognize
(and reach for) once you've seen them. This module covers four of the most
common: Factory, Singleton, Observer, and Strategy.

## Factory — centralizing object creation

A factory hides the details of *which* concrete class to instantiate behind a
single creation method, so callers depend on an interface, not a constructor.

```java
public interface Shape {
    double area();
}

public class Circle implements Shape {
    private final double radius;
    public Circle(double radius) { this.radius = radius; }
    @Override public double area() { return Math.PI * radius * radius; }
}

public class Square implements Shape {
    private final double side;
    public Square(double side) { this.side = side; }
    @Override public double area() { return side * side; }
}

public class ShapeFactory {
    public static Shape create(String type, double size) {
        return switch (type.toLowerCase()) {
            case "circle" -> new Circle(size);
            case "square" -> new Square(size);
            default -> throw new IllegalArgumentException("Unknown shape: " + type);
        };
    }
}

public class FactoryDemo {
    public static void main(String[] args) {
        Shape s1 = ShapeFactory.create("circle", 3);
        Shape s2 = ShapeFactory.create("square", 4);

        System.out.printf("Circle area: %.2f%n", s1.area());   // Circle area: 28.27
        System.out.printf("Square area: %.2f%n", s2.area());   // Square area: 16.00
    }
}
```

Callers never write `new Circle(...)` directly — if a new shape type is added
later, only `ShapeFactory` needs to change.

## Singleton — exactly one instance

A singleton guarantees a class has only one instance, globally accessible.
The classic approach uses a private constructor and a static instance:

```java
public class ConfigManager {
    private static ConfigManager instance;
    private final java.util.Map<String, String> settings = new java.util.HashMap<>();

    private ConfigManager() { }   // private -- no one else can call "new ConfigManager()"

    public static synchronized ConfigManager getInstance() {
        if (instance == null) {
            instance = new ConfigManager();
        }
        return instance;
    }

    public void set(String key, String value) { settings.put(key, value); }
    public String get(String key) { return settings.get(key); }
}

ConfigManager.getInstance().set("env", "production");
System.out.println(ConfigManager.getInstance().get("env"));   // production
```

The modern, recommended approach in Java is an **enum singleton** — it's
thread-safe by construction, serialization-safe, and immune to reflection
attacks that can otherwise break the classic pattern:

```java
public enum AppConfig {
    INSTANCE;

    private final java.util.Map<String, String> settings = new java.util.HashMap<>();

    public void set(String key, String value) { settings.put(key, value); }
    public String get(String key) { return settings.get(key); }
}

AppConfig.INSTANCE.set("env", "production");
System.out.println(AppConfig.INSTANCE.get("env"));   // production
```

## Observer — reacting to events

The observer pattern lets a subject notify a list of interested listeners
whenever something happens, without the subject knowing anything about them
beyond a common interface.

```java
import java.util.ArrayList;
import java.util.List;

public interface OrderListener {
    void onOrderPlaced(String item);
}

public class OrderService {
    private final List<OrderListener> listeners = new ArrayList<>();

    public void addListener(OrderListener listener) {
        listeners.add(listener);
    }

    public void placeOrder(String item) {
        System.out.println("Order placed: " + item);
        for (OrderListener listener : listeners) {
            listener.onOrderPlaced(item);   // notify every registered observer
        }
    }
}

public class ObserverDemo {
    public static void main(String[] args) {
        OrderService orders = new OrderService();

        orders.addListener(item -> System.out.println("Email: your order for " + item + " is confirmed"));
        orders.addListener(item -> System.out.println("Inventory: reserving stock for " + item));

        orders.placeOrder("Java Mastery Path Book");
        // Order placed: Java Mastery Path Book
        // Email: your order for Java Mastery Path Book is confirmed
        // Inventory: reserving stock for Java Mastery Path Book
    }
}
```

## Strategy — swappable algorithms

The strategy pattern extracts an algorithm behind an interface so it can be
swapped at runtime, avoiding long `if`/`switch` chains scattered through the
codebase.

```java
public interface DiscountStrategy {
    double apply(double price);
}

public class NoDiscount implements DiscountStrategy {
    @Override public double apply(double price) { return price; }
}

public class PercentageDiscount implements DiscountStrategy {
    private final double percent;
    public PercentageDiscount(double percent) { this.percent = percent; }
    @Override public double apply(double price) { return price * (1 - percent / 100); }
}

public class FlatDiscount implements DiscountStrategy {
    private final double amount;
    public FlatDiscount(double amount) { this.amount = amount; }
    @Override public double apply(double price) { return Math.max(0, price - amount); }
}

public class Checkout {
    private final DiscountStrategy strategy;

    public Checkout(DiscountStrategy strategy) {
        this.strategy = strategy;
    }

    public double total(double price) {
        return strategy.apply(price);
    }
}

public class StrategyDemo {
    public static void main(String[] args) {
        Checkout regular = new Checkout(new NoDiscount());
        Checkout memberDay = new Checkout(new PercentageDiscount(20));
        Checkout coupon = new Checkout(new FlatDiscount(5));

        System.out.printf("Regular: $%.2f%n", regular.total(50));      // Regular: $50.00
        System.out.printf("Member day: $%.2f%n", memberDay.total(50)); // Member day: $40.00
        System.out.printf("Coupon: $%.2f%n", coupon.total(50));        // Coupon: $45.00
    }
}
```

The `Checkout` class never changes when a new discount type is added — it
just depends on the `DiscountStrategy` interface.

| Pattern | Problem it solves | Key idea |
|---------|--------------------|----------|
| Factory | Which concrete class to instantiate | Centralize creation behind one method/interface |
| Singleton | Ensure exactly one shared instance | Private constructor + controlled global access point |
| Observer | Notify many interested parties of an event | Subject holds a list of listener interfaces |
| Strategy | Swappable algorithms without conditionals | Extract behavior behind an interface, inject the implementation |

## Exercise

Implement the Strategy pattern for a `SortingStrategy` interface with a
single method `int[] sort(int[] data)`. Provide two implementations —
`BubbleSortStrategy` and `BuiltInSortStrategy` (using `Arrays.sort`) — and a
`Sorter` class that takes a `SortingStrategy` in its constructor and exposes a
`sort` method delegating to it. Demonstrate swapping strategies at runtime on
the same unsorted array.
