# 04 · Methods & Parameters

A method is a named, reusable block of code. Methods live inside classes.

## Defining and calling a method

```java
public class MathUtils {

    // static method: belongs to the class itself, not an instance
    static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        int result = add(2, 3);
        System.out.println(result);   // 5
    }
}
```

A method signature has: an access modifier (`public`, `private`, ...), an
optional `static`, a return type (or `void`), a name, and a parameter list.

## Parameters and return values

```java
static double average(int[] numbers) {
    int sum = 0;
    for (int n : numbers) {
        sum += n;
    }
    return (double) sum / numbers.length;
}

static void printReport(String title) {
    System.out.println("Report: " + title);
    // no return statement needed -- return type is void
}
```

## Method overloading

Multiple methods can share a name if their **parameter lists** differ (in
count or type) — this is overloading, resolved at compile time.

```java
static int multiply(int a, int b) {
    return a * b;
}

static double multiply(double a, double b) {
    return a * b;
}

static int multiply(int a, int b, int c) {
    return a * b * c;
}

System.out.println(multiply(2, 3));        // 6 (int version)
System.out.println(multiply(2.5, 4.0));    // 10.0 (double version)
System.out.println(multiply(2, 3, 4));      // 24 (three-arg version)
```

## static vs. instance methods

A `static` method belongs to the class and can be called without an object. An
**instance method** operates on a specific object's data and requires one to
exist first.

```java
public class Counter {
    private int count = 0;   // instance field

    // instance method -- needs a Counter object to call it
    void increment() {
        count++;
    }

    int getCount() {
        return count;
    }

    // static method -- called via Counter.describe(), no instance needed
    static String describe() {
        return "A simple counter class";
    }
}

Counter c = new Counter();
c.increment();
c.increment();
System.out.println(c.getCount());        // 2
System.out.println(Counter.describe());   // A simple counter class
```

| Kind | Called via | Can access instance fields? |
|------|-----------|------------------------------|
| `static` method | `ClassName.method()` | No |
| instance method | `object.method()` | Yes |

## Variable scope

A variable declared inside a method (or block) only exists within that block.

```java
static void demo() {
    int local = 10;
    if (local > 5) {
        int inner = local * 2;
        System.out.println(inner);   // 20 -- visible here
    }
    // inner is NOT visible here -- out of scope
}
```

## How It Actually Works

Every method call pushes a new **stack frame** — its own local variable
array, operand stack, and reference to the constant pool — onto the
calling thread's call stack. Arguments are passed **by value**, always:
for a primitive that's a copy of the bits; for an object reference that's
a copy of the *pointer*, which is why mutating an object through a
parameter is visible to the caller (same object) but reassigning the
parameter itself is not (you only repointed your local copy).

Overload resolution (`invokevirtual` target selection when multiple
methods share a name) happens **entirely at compile time** based on the
static types of the arguments — the compiler picks the most specific
applicable overload and bakes that exact method descriptor into the
`invoke*` instruction's constant-pool entry. This is why passing `null`
to two same-named overloads that both accept reference types is
ambiguous at compile time even though it would be perfectly resolvable
at runtime.

Recursive calls consume stack frames linearly; the JVM does **not**
perform tail-call optimization (unlike some other JVM languages' own
compilers), so deep unbounded recursion in Java always risks
`StackOverflowError` — a real, fixed-size limit (`-Xss`, default around
512KB–1MB per thread) rather than a soft warning.

## Exercise

Write a class `Calculator` with overloaded static methods `add` for two `int`s,
two `double`s, and three `int`s. Add an instance method `remember(int value)`
that stores the last computed value in a field, and a `getLastValue()` method
to retrieve it. Write a `main` method that exercises all of them.
