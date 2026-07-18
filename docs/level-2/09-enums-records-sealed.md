# 09 · Enums, Records & Sealed Classes

This module covers three modern Java features that make data modeling more
concise and safer: `enum` for fixed sets of constants, `record` for
immutable data carriers, and `sealed` classes for restricted class
hierarchies.

## Enums with fields, constructors, and methods

An `enum` is more than a list of names — each constant can carry its own
data and every constant shares the enum's methods.

```java
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6),
    EARTH(5.976e+24, 6.37814e6);

    private final double mass;    // kilograms
    private final double radius;  // meters

    // enum constructors are implicitly private -- called once per constant
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }

    double surfaceGravity() {
        final double G = 6.67300E-11;
        return G * mass / (radius * radius);
    }
}

for (Planet p : Planet.values()) {
    System.out.printf("%-8s gravity: %.2f%n", p, p.surfaceGravity());
}
// Output:
// MERCURY  gravity: 3.70
// VENUS    gravity: 8.87
// EARTH    gravity: 9.80
```

`values()` returns every constant in declaration order, and every enum
automatically gets `name()`, `ordinal()` (its position, starting at 0), and a
`valueOf(String)` static lookup.

## Enums implementing an interface

Enum constants can override methods per-constant, which is a clean
alternative to a chain of `if`/`switch` statements — each constant supplies
its own behavior.

```java
public interface Operation {
    int apply(int a, int b);
}

public enum BasicOperation implements Operation {
    ADD {
        @Override public int apply(int a, int b) { return a + b; }
    },
    SUBTRACT {
        @Override public int apply(int a, int b) { return a - b; }
    },
    MULTIPLY {
        @Override public int apply(int a, int b) { return a * b; }
    };
}

for (BasicOperation op : BasicOperation.values()) {
    System.out.println(op + "(3, 4) = " + op.apply(3, 4));
}
// ADD(3, 4) = 7
// SUBTRACT(3, 4) = -1
// MULTIPLY(3, 4) = 12
```

## Records — concise immutable data carriers

A `record` declares its fields once in the header, and the compiler
generates a canonical constructor, private final fields, public accessor
methods (named after the field, not `getX()`), plus `equals`, `hashCode`, and
`toString` — all automatically.

```java
public record Point(int x, int y) {}

Point p1 = new Point(3, 4);
Point p2 = new Point(3, 4);

System.out.println(p1.x());          // 3 -- accessor, not getX()
System.out.println(p1);               // Point[x=3, y=4] -- generated toString
System.out.println(p1.equals(p2));    // true -- generated equals compares all fields
System.out.println(p1.hashCode() == p2.hashCode());   // true
```

Compare this to writing the equivalent class by hand with a constructor,
three accessors, `equals`, `hashCode`, and `toString` — records eliminate
that boilerplate entirely for simple data holders.

## Records with validation in a compact constructor

A **compact constructor** (no parameter list — it reuses the record's
declared components) lets you validate or normalize input before the fields
are assigned.

```java
public record Range(int min, int max) {
    // compact constructor -- runs before the fields are set
    public Range {
        if (min > max) {
            throw new IllegalArgumentException("min (" + min + ") cannot exceed max (" + max + ")");
        }
    }

    public int length() {
        return max - min;   // records can have extra methods too
    }
}

Range valid = new Range(1, 10);
System.out.println(valid.length());   // 9

Range invalid = new Range(10, 1);   // throws IllegalArgumentException
```

## Sealed classes and interfaces

A `sealed` type declares exactly which classes are allowed to extend or
implement it via `permits` — closing off the hierarchy so the compiler (and
readers) know the complete set of subtypes.

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}

public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
public record Triangle(double base, double height) implements Shape {}
```

Permitted subtypes must be `final`, `sealed`, or `non-sealed` themselves —
records are implicitly `final`, so they qualify automatically.

## Exhaustive `switch` pattern matching over a sealed hierarchy

Because the compiler knows every possible subtype of a sealed interface, a
`switch` expression over one can skip a `default` branch entirely once every
case is covered — and the compiler will error if a new subtype is added later
without updating the `switch`.

```java
public static double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t -> 0.5 * t.base() * t.height();
        // no default needed -- Circle, Rectangle, Triangle are the only permitted types
    };
}

List<Shape> shapes = List.of(new Circle(2), new Rectangle(3, 4), new Triangle(6, 2));
for (Shape s : shapes) {
    System.out.printf("%.2f%n", area(s));
}
// Output:
// 12.57
// 12.00
// 6.00
```

This combines pattern matching (from [Module 1](01-oop-deep-dive.md)'s
`instanceof` form) with `switch`, and is the modern, type-safe replacement
for long `instanceof`/cast chains.

## Cheat sheet

| Feature | Purpose | Key trait |
|---------|---------|-----------|
| `enum` | Fixed set of named constants | Can hold fields, methods, per-constant bodies |
| `record` | Immutable data carrier | Auto-generates constructor, accessors, `equals`/`hashCode`/`toString` |
| `sealed` | Restrict which types may extend/implement | Enables exhaustive `switch` with no `default` |

## Exercise

Define a sealed interface `PaymentMethod permits CreditCard, BankTransfer,
Cash`, where `CreditCard` and `BankTransfer` are records (with fields like
account/card number) and `Cash` is a record with no fields. Write a method
`processingFee(PaymentMethod method)` using an exhaustive `switch` expression
that returns a flat 2% fee for `CreditCard`, a flat $1.50 for
`BankTransfer`, and `0` for `Cash`. Then define an `enum Currency` with
constants `USD`, `EUR`, `GBP`, each carrying a `String symbol` field set via
an enum constructor, and a method `format(double amount)` that returns the
symbol concatenated with the amount.
