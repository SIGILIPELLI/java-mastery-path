# 02 · Variables, Data Types & Operators

Java is **statically typed**: every variable's type is fixed at declaration and
checked by the compiler, before the program ever runs.

## Declaring variables

```java
int age = 30;              // declare and initialize in one line
String name;                // declare now, assign later
name = "Ada";

final double PI_APPROX = 3.14159;  // final = cannot be reassigned
```

## Primitive types

Primitives store raw values directly (not objects) and are the fastest,
lowest-overhead types in Java.

| Type | Size | Example | Notes |
|------|------|---------|-------|
| `byte` | 8-bit | `byte b = 100;` | small integers |
| `short` | 16-bit | `short s = 30000;` | rarely used directly |
| `int` | 32-bit | `int x = 42;` | default integer type |
| `long` | 64-bit | `long big = 10_000_000_000L;` | note the `L` suffix |
| `float` | 32-bit | `float f = 3.14f;` | note the `f` suffix |
| `double` | 64-bit | `double d = 3.14159;` | default floating-point type |
| `char` | 16-bit | `char c = 'A';` | single UTF-16 character |
| `boolean` | 1 bit (JVM-dependent) | `boolean ok = true;` | `true`/`false` only |

```java
int x = 42;
long big = 10_000_000_000L;   // underscores as digit separators
double price = 19.99;
char grade = 'A';
boolean passed = true;
```

## Reference (object) types

Everything that isn't a primitive is a **reference type** — the variable holds
a reference (pointer) to an object on the heap. `String`, arrays, and every
class you write are reference types.

```java
String greeting = "Hello";     // reference to a String object
int[] numbers = {1, 2, 3};      // reference to an array object
Integer boxed = 42;             // "boxed" wrapper around a primitive int
```

Every primitive has a corresponding **wrapper class** (`int` → `Integer`,
`double` → `Double`, `boolean` → `Boolean`, ...) used when you need an object —
for example, storing numbers in a `List`, which only holds objects.

```java
import java.util.List;
import java.util.ArrayList;

List<Integer> scores = new ArrayList<>();
scores.add(95);        // int 95 is auto-boxed into Integer
int first = scores.get(0);  // Integer is auto-unboxed into int
```

## Casting

**Widening** (safe, implicit): smaller type to a bigger one.

```java
int i = 100;
long l = i;        // int -> long, automatic
double d = l;       // long -> double, automatic
```

**Narrowing** (unsafe, must be explicit): bigger type to a smaller one, can
lose data.

```java
double price = 19.99;
int wholeDollars = (int) price;   // 19 -- truncates, doesn't round
long big = 300L;
byte small = (byte) big;           // may overflow/wrap silently
```

## Operators

```java
int a = 10, b = 3;

System.out.println(a + b);   // 13
System.out.println(a - b);   // 7
System.out.println(a * b);   // 30
System.out.println(a / b);   // 3  -- integer division truncates
System.out.println(a % b);   // 1  -- remainder
System.out.println((double) a / b);  // 3.3333333333333335 -- cast first

// Compound assignment
int count = 0;
count += 5;   // count = count + 5
count++;      // increment by 1
count--;      // decrement by 1

// Comparison and logical operators
boolean result = (a > b) && (b > 0);   // AND
boolean either = (a > b) || (b < 0);   // OR
boolean not = !result;                  // NOT
```

| Category | Operators |
|----------|-----------|
| Arithmetic | `+ - * / %` |
| Assignment | `= += -= *= /= %=` |
| Comparison | `== != > < >= <=` |
| Logical | `&& \|\| !` |
| Increment/Decrement | `++ --` |

## Exercise

Write a program that declares an `int` for a number of items, a `double` for
a price per item, computes the total as a `double`, and prints it formatted to
two decimal places using `System.out.printf("%.2f%n", total)`. Then add a
`boolean` for whether the total exceeds `100.0` and print that too.
