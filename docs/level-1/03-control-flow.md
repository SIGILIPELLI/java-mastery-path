# 03 · Control Flow

## if / else

```java
int age = 20;

if (age < 13) {
    System.out.println("Child");
} else if (age < 20) {
    System.out.println("Teenager");
} else {
    System.out.println("Adult");
}
```

## switch statement

Classic `switch` falls through unless you `break`:

```java
int day = 3;
String name;

switch (day) {
    case 1:
        name = "Monday";
        break;
    case 2:
        name = "Tuesday";
        break;
    case 3:
        name = "Wednesday";
        break;
    default:
        name = "Unknown";
}

System.out.println(name);   // Wednesday
```

## Modern switch expression

Java 14+ supports `switch` as an **expression** with arrow syntax — no
fall-through, and it returns a value directly:

```java
int day = 3;

String name = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4, 5 -> "Almost the weekend";
    default -> "Unknown";
};

System.out.println(name);   // Wednesday
```

## while and do-while loops

```java
int count = 0;
while (count < 3) {
    System.out.println("count = " + count);
    count++;
}
// count = 0, count = 1, count = 2

int n = 0;
do {
    System.out.println("runs at least once: " + n);
    n++;
} while (n < 0);   // condition is false immediately, but body already ran once
```

## for loops

```java
// Classic indexed for loop
for (int i = 0; i < 5; i++) {
    System.out.println("i = " + i);
}

// Enhanced for-each loop (arrays, collections)
int[] scores = {90, 85, 77};
for (int score : scores) {
    System.out.println("score: " + score);
}
```

## break and continue

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;       // exit the loop entirely
    }
    if (i % 2 == 0) {
        continue;    // skip to the next iteration
    }
    System.out.println(i);   // prints 1, 3
}
```

| Construct | Use when |
|-----------|----------|
| `if / else` | branching on a boolean condition |
| `switch` | branching on one variable against many discrete values |
| `while` | repeat while a condition holds, unknown iteration count |
| `do-while` | same, but body always runs at least once |
| `for` | repeat a known number of times, or with an index |
| for-each | iterate every element of an array/collection |

## How It Actually Works

Every `if`/`else`, `while`, and `for` you write compiles down to
**conditional jump bytecodes** operating on the operand stack —
`if_icmpge`, `ifeq`, `goto` — there is no structured-control-flow concept
inside the class file at all; it's all comparisons and jumps to byte
offsets, exactly like assembly. The compiler's job is to turn nested
braces into a flat, jump-target-labeled instruction stream.

The JIT compiler treats loop bodies specially: methods with hot loops can
get **on-stack replacement (OSR)** — the JVM compiles just that loop to
native code and transplants the interpreter's live locals into the
compiled frame *while the loop is still running*, without waiting for the
method to be called again. This is why a single long-running loop in
`main` can still benefit from JIT optimization even though `main` itself
is called only once.

`switch` on a `String` is not a special bytecode — the compiler
desugars it into two nested switches: the first switches on
`String.hashCode()` (a fast `tableswitch`/`lookupswitch` over ints), and
inside each hash bucket a chain of `.equals()` calls disambiguates
collisions before jumping to the matching case body. That's why
`hashCode()` consistency matters even though you never call it directly
in a `switch` statement.

## Exercise

Write a program that loops from 1 to 100 and, for each number: prints "Fizz"
if divisible by 3, "Buzz" if divisible by 5, "FizzBuzz" if divisible by both,
and the number itself otherwise (classic FizzBuzz). Implement it once with a
classic `for` loop and `if/else`, then rewrite the divisibility check using a
switch expression.
