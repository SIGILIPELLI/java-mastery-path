# 08 · Exception Handling Basics

Exceptions represent errors or unusual conditions detected at runtime. Java
lets you handle them explicitly instead of crashing the whole program.

## try / catch / finally

```java
public class Divide {
    public static void main(String[] args) {
        int a = 10;
        int b = 0;

        try {
            int result = a / b;   // throws ArithmeticException
            System.out.println(result);
        } catch (ArithmeticException e) {
            System.out.println("Cannot divide by zero: " + e.getMessage());
        } finally {
            System.out.println("This always runs, error or not.");
        }

        System.out.println("Program continues normally.");
    }
}
// Output:
// Cannot divide by zero: / by zero
// This always runs, error or not.
// Program continues normally.
```

`finally` runs whether or not an exception was thrown — commonly used for
cleanup (closing files, releasing resources).

## Catching multiple exception types

```java
public class ParseInput {
    public static void main(String[] args) {
        String[] inputs = {"42", "oops", null};

        for (String input : inputs) {
            try {
                int value = Integer.parseInt(input);
                System.out.println("Parsed: " + value);
            } catch (NumberFormatException e) {
                System.out.println("Not a number: " + input);
            } catch (NullPointerException e) {
                System.out.println("Input was null");
            }
        }
    }
}
// Parsed: 42
// Not a number: oops
// Input was null
```

You can also catch several types in one clause with `|`:

```java
try {
    riskyOperation();
} catch (NumberFormatException | NullPointerException e) {
    System.out.println("Bad input: " + e.getMessage());
}
```

## Checked vs. unchecked exceptions

Java has two families of exceptions:

- **Checked exceptions** (subclasses of `Exception` but not `RuntimeException`)
  — the compiler *forces* you to either catch them or declare `throws` on the
  method. Examples: `IOException`, `SQLException`.
- **Unchecked exceptions** (subclasses of `RuntimeException`) — the compiler
  does not require handling. Examples: `NullPointerException`,
  `ArithmeticException`, `IllegalArgumentException`, `IndexOutOfBoundsException`.

```java
import java.io.FileReader;
import java.io.IOException;

// Checked exception -- must declare "throws" or wrap in try/catch
public static void readFile(String path) throws IOException {
    FileReader reader = new FileReader(path);   // can throw IOException
    reader.close();
}

public static void main(String[] args) {
    try {
        readFile("nonexistent.txt");
    } catch (IOException e) {
        System.out.println("File problem: " + e.getMessage());
    }
}
```

| Kind | Extends | Must be declared/caught? | Examples |
|------|---------|---------------------------|----------|
| Checked | `Exception` | Yes | `IOException`, `SQLException` |
| Unchecked | `RuntimeException` | No | `NullPointerException`, `ArithmeticException` |
| Error | `Error` | No (don't catch these) | `OutOfMemoryError`, `StackOverflowError` |

## Throwing your own exceptions

```java
public class AgeValidator {
    static void validate(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Age out of range: " + age);
        }
    }

    public static void main(String[] args) {
        try {
            validate(-5);
        } catch (IllegalArgumentException e) {
            System.out.println("Validation failed: " + e.getMessage());
        }
    }
}
```

Custom exception classes (extending `Exception` or `RuntimeException`) are
covered in [Level 2](../level-2/05-exceptions-advanced.md), along with
try-with-resources for automatic cleanup.

## Exercise

Write a method `safeDivide(int a, int b)` that returns the division result, but
catches `ArithmeticException` internally and returns `0` instead of crashing
when dividing by zero (print a warning message when this happens). Then write
a method `parseAge(String input)` that parses a string to an `int` using
`Integer.parseInt`, catches `NumberFormatException`, and throws a new
`IllegalArgumentException` with a clearer message ("Invalid age: " + input) if
parsing fails.
