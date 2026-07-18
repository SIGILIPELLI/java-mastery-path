# 01 · OOP Deep Dive

Level 1 introduced classes and objects. Level 2 goes deeper into the four
pillars that make object-oriented Java expressive: inheritance, polymorphism,
abstraction, and encapsulation (the last of which you already met in
[Level 1](../level-1/07-classes-objects.md)).

## Inheritance with `extends`

A subclass inherits the fields and methods of a superclass, and can add its
own or override existing behavior.

```java
public class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public String makeSound() {
        return "...";
    }

    public String describe() {
        return name + " says " + makeSound();
    }
}

public class Dog extends Animal {
    public Dog(String name) {
        super(name);   // must call the superclass constructor first
    }

    @Override
    public String makeSound() {
        return "Woof!";
    }
}

public class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }

    @Override
    public String makeSound() {
        return "Meow!";
    }
}
```

```java
Animal dog = new Dog("Rex");
Animal cat = new Cat("Whiskers");
System.out.println(dog.describe());   // Rex says Woof!
System.out.println(cat.describe());   // Whiskers says Meow!
```

## Overriding vs. overloading

These two words sound similar but mean very different things:

- **Overriding** — a subclass provides a new implementation of a method that
  already exists in its superclass, with the *same signature*. Marked with
  `@Override` (optional but strongly recommended — the compiler catches typos).
- **Overloading** — multiple methods in the *same* class share a name but
  differ in parameter list. This is what `Rectangle`'s two constructors did in
  [Level 1](../level-1/07-classes-objects.md).

```java
public class Calculator {
    // Overloaded methods -- same name, different parameters
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

## `super` — calling the parent explicitly

`super` refers to the superclass. It is used to call a superclass constructor
(must be the first statement in a subclass constructor) or to call a
superclass method that has been overridden.

```java
public class Employee {
    protected double baseSalary;

    public Employee(double baseSalary) {
        this.baseSalary = baseSalary;
    }

    public double calculatePay() {
        return baseSalary;
    }
}

public class Manager extends Employee {
    private double bonus;

    public Manager(double baseSalary, double bonus) {
        super(baseSalary);
        this.bonus = bonus;
    }

    @Override
    public double calculatePay() {
        return super.calculatePay() + bonus;   // reuse the parent's logic
    }
}

Manager m = new Manager(60000, 5000);
System.out.println(m.calculatePay());   // 65000.0
```

## Polymorphism — upcasting and dynamic dispatch

A subclass reference can always be stored in a variable of its superclass
type ("upcasting"). At runtime, Java calls the *actual* object's overridden
method, not the one belonging to the declared type — this is **dynamic
dispatch**, and it's what let the `dog.describe()` call above pick `Dog`'s
`makeSound()` even though `describe()` is written in `Animal`.

```java
List<Animal> zoo = List.of(new Dog("Rex"), new Cat("Whiskers"), new Dog("Fido"));
for (Animal a : zoo) {
    System.out.println(a.describe());   // each calls its own makeSound()
}
// Output:
// Rex says Woof!
// Whiskers says Meow!
// Fido says Woof!
```

This is the core benefit of polymorphism: code written against the
superclass type (`Animal`) automatically works for any current or future
subclass, with no `if`/`else` chains checking types.

## Abstract classes

An `abstract` class cannot be instantiated directly — it exists to be
extended. It can mix concrete (implemented) methods with `abstract` methods
that subclasses are *forced* to implement.

```java
public abstract class Shape {
    abstract double area();   // no body -- subclasses must provide one

    // concrete method -- shared by all subclasses
    void printArea() {
        System.out.printf("Area: %.2f%n", area());
    }
}

public class Circle extends Shape {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

Shape s = new Circle(3);
s.printArea();   // Area: 28.27
// Shape shape = new Shape();   // won't compile -- abstract classes can't be instantiated
```

## Interfaces

An interface defines a contract — a set of methods a class promises to
implement — without dictating how objects are constructed or what state they
hold. A class can implement **multiple** interfaces, unlike single-inheritance
classes.

```java
public interface Payable {
    double calculatePay();
}

public interface Reportable {
    String reportLine();
}

public class Contractor implements Payable, Reportable {
    private String name;
    private double hourlyRate;
    private int hours;

    public Contractor(String name, double hourlyRate, int hours) {
        this.name = name;
        this.hourlyRate = hourlyRate;
        this.hours = hours;
    }

    @Override
    public double calculatePay() {
        return hourlyRate * hours;
    }

    @Override
    public String reportLine() {
        return name + ": $" + calculatePay();
    }
}
```

## Default and static methods on interfaces

Since Java 8, interfaces can provide `default` methods (a body that
implementing classes inherit unless they override it) and `static` methods
(utility methods called on the interface itself).

```java
public interface Greeter {
    String name();

    // default method -- concrete, inherited automatically
    default String greet() {
        return "Hello, " + name() + "!";
    }

    // static method -- called as Greeter.formalGreeting(...), not on an instance
    static String formalGreeting(String name) {
        return "Good day, " + name + ".";
    }
}

public class Friend implements Greeter {
    private String name;
    public Friend(String name) { this.name = name; }

    @Override
    public String name() { return name; }
}

Friend f = new Friend("Sam");
System.out.println(f.greet());                       // Hello, Sam!
System.out.println(Greeter.formalGreeting("Sam"));    // Good day, Sam.
```

## Abstract class vs. interface

| Aspect | Abstract class | Interface |
|--------|-----------------|-----------|
| Instantiable? | No | No |
| Fields | Any (instance state allowed) | Only `static final` constants |
| Multiple inheritance | No (`extends` one class only) | Yes (`implements` many) |
| Method bodies | Concrete + abstract methods | `default`/`static` bodies + abstract signatures |
| Use when | Sharing state and code among closely related classes | Defining a capability unrelated classes can opt into |

## `instanceof` with pattern matching

Traditional `instanceof` checks a type and then requires a separate cast.
Modern Java (16+) lets you bind the cast result to a variable directly in the
condition.

```java
Object obj = new Dog("Rex");

// Old style
if (obj instanceof Animal) {
    Animal a = (Animal) obj;
    System.out.println(a.describe());
}

// Pattern matching for instanceof -- no separate cast needed
if (obj instanceof Animal a) {
    System.out.println(a.describe());   // "a" is already Animal here
}

if (obj instanceof Dog d && d.name.equals("Rex")) {
    System.out.println("Found Rex the dog!");
}
```

## Exercise

Design an interface `Discountable` with one method `double discountedPrice()`.
Create an abstract class `Product` with fields `name` and `price`, a
constructor, and a concrete method `describe()` that prints the name and
price. Create two subclasses of `Product` that also implement `Discountable`
— `ClearanceItem` (50% off) and `MemberItem` (10% off) — each providing its
own `discountedPrice()`. In `main`, put several products of both kinds in a
`List<Product>` and loop over it, printing each one's `describe()` output
followed by its discounted price (you'll need an `instanceof` pattern-matching
check, since `discountedPrice()` lives on `Discountable`, not `Product`).
