# 07 · Classes & Objects Basics

A class is a blueprint; an object is an instance created from that blueprint.

## Fields and constructors

```java
public class Person {
    // fields (instance state)
    String name;
    int age;

    // constructor -- runs when you create a new Person
    public Person(String name, int age) {
        this.name = name;   // "this.name" is the field, "name" is the parameter
        this.age = age;
    }

    void introduce() {
        System.out.println("Hi, I'm " + this.name + ", age " + this.age);
    }
}

public class Main {
    public static void main(String[] args) {
        Person alice = new Person("Alice", 30);
        Person bob = new Person("Bob", 25);

        alice.introduce();   // Hi, I'm Alice, age 30
        bob.introduce();     // Hi, I'm Bob, age 25

        System.out.println(alice.name);   // Alice -- direct field access
    }
}
```

`this` refers to the current instance — it disambiguates a field from a
parameter of the same name, and lets a constructor or method refer to "this
particular object."

## Multiple constructors (overloading)

```java
public class Rectangle {
    double width;
    double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    // Overloaded constructor -- a square is a rectangle with equal sides
    public Rectangle(double side) {
        this(side, side);   // calls the other constructor
    }

    double area() {
        return width * height;
    }
}

Rectangle r = new Rectangle(4, 5);
Rectangle square = new Rectangle(3);
System.out.println(r.area());       // 20.0
System.out.println(square.area());   // 9.0
```

## Encapsulation — private fields with getters/setters

Direct field access (like `alice.name` above) is convenient but breaks
encapsulation — nothing stops invalid data from being assigned. Best practice
is to make fields `private` and expose controlled access:

```java
public class BankAccount {
    private double balance;   // private -- not accessible outside this class

    public BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount > balance) {
            throw new IllegalStateException("Insufficient funds");
        }
        balance -= amount;
    }
}

BankAccount account = new BankAccount(100.0);
account.deposit(50.0);
account.withdraw(30.0);
System.out.println(account.getBalance());   // 120.0
// account.balance = -500;   // won't compile -- balance is private
```

## Instance methods operate on "this" object's data

```java
public class Circle {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    double area() {
        return Math.PI * radius * radius;
    }

    double circumference() {
        return 2 * Math.PI * radius;
    }
}

Circle c1 = new Circle(2.0);
Circle c2 = new Circle(5.0);
System.out.printf("%.2f%n", c1.area());   // 12.57 -- each object has its own radius
System.out.printf("%.2f%n", c2.area());   // 78.54
```

| Concept | Meaning |
|---------|---------|
| Class | Blueprint / type definition |
| Object (instance) | A concrete value created with `new` |
| Field | A variable that lives on each instance |
| Constructor | Special method that initializes a new instance |
| `this` | Reference to the current instance |
| `private` | Restricts access to within the class (encapsulation) |

## How It Actually Works

`new MyClass(...)` is really three JVM steps: allocate raw memory sized
by the class's instance layout (object header — mark word + klass pointer
— plus all inherited and declared instance fields, padded for alignment),
run field defaults, then call the constructor via `invokespecial`, which
first chains to the superclass constructor before running your body — so
`Object`'s (trivial) constructor always runs before anything else, top of
the hierarchy down.

Instance method calls compile to `invokevirtual`, which is resolved
through the object's actual **vtable** (a per-class array of method
pointers built during class linking) rather than the compile-time
reference type — this single mechanism *is* runtime polymorphism:
calling `shape.area()` looks up index N in whatever concrete class's
vtable `shape` currently points to. `private`/`static`/constructor calls
skip this entirely and use `invokespecial`/`invokestatic`, which is
resolved once, directly, with no virtual dispatch — a real performance
and semantics difference, not just a style rule.

`this` is passed as a hidden first argument (local slot 0) to every
instance method — it isn't magic, it's an implicit parameter the compiler
inserts, which is also why static methods (no hidden `this`) can't
access instance fields.

## Exercise

Write a `Book` class with private fields `title`, `author`, and `pagesRead`
(starting at 0), a constructor taking `title` and `author`, a method
`readPages(int n)` that increases `pagesRead`, and a method `getProgress(int totalPages)`
that returns the percentage read as a `double`. Create two `Book` objects in
`main`, read some pages on each, and print their progress.
