# 05 · Exception Handling Advanced

[Level 1](../level-1/08-exception-handling.md) covered `try`/`catch`/`finally`
and the built-in exception types. This module covers writing your *own*
exception classes and Java's modern resource-cleanup syntax,
try-with-resources.

## Custom unchecked exceptions

Extend `RuntimeException` for errors that represent programming mistakes or
conditions callers aren't required to explicitly handle.

```java
public class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);   // pass the message up to Exception's constructor
    }
}

public class Account {
    private double balance;

    public Account(double balance) {
        this.balance = balance;
    }

    public void withdraw(double amount) {
        if (amount > balance) {
            throw new InsufficientFundsException(
                "Cannot withdraw " + amount + "; balance is only " + balance);
        }
        balance -= amount;
    }
}

Account acc = new Account(100);
acc.withdraw(150);   // throws InsufficientFundsException: Cannot withdraw 150.0; balance is only 100.0
```

## Custom checked exceptions

Extend `Exception` directly (not `RuntimeException`) when callers *must*
handle the failure — typically for recoverable conditions tied to external
resources (files, networks, parsing).

```java
public class InvalidOrderException extends Exception {
    public InvalidOrderException(String message) {
        super(message);
    }
}

public class OrderProcessor {
    public void submit(int quantity) throws InvalidOrderException {
        if (quantity <= 0) {
            throw new InvalidOrderException("Quantity must be positive: " + quantity);
        }
        System.out.println("Order submitted for " + quantity + " item(s)");
    }
}

OrderProcessor processor = new OrderProcessor();
try {
    processor.submit(-3);
} catch (InvalidOrderException e) {
    System.out.println("Order failed: " + e.getMessage());
}
// Order failed: Quantity must be positive: -3
```

Because `submit` declares `throws InvalidOrderException`, every caller is
forced by the compiler to either catch it or re-declare it — the same rule
[Level 1](../level-1/08-exception-handling.md) showed for `IOException`.

## Exception chaining

When you catch one exception but throw another (often to translate a
low-level failure into a more meaningful, domain-specific one), pass the
original exception as the **cause** so the full failure history is preserved
for debugging.

```java
public class DataLoadException extends RuntimeException {
    public DataLoadException(String message, Throwable cause) {
        super(message, cause);   // constructor form that sets the cause directly
    }
}

public void loadConfig(String path) {
    try {
        int result = Integer.parseInt(readRawValue(path));
    } catch (NumberFormatException e) {
        throw new DataLoadException("Config value at " + path + " was not numeric", e);
    }
}
```

`e.getCause()` on the resulting `DataLoadException` returns the original
`NumberFormatException`, and printing the stack trace shows both, chained
with "Caused by:". `initCause(Throwable)` achieves the same thing after
construction, for exception types whose constructor doesn't accept a cause.

## try-with-resources and `AutoCloseable`

Any class implementing `AutoCloseable` (a single method, `close()`) can be
declared inside the parentheses of a `try` — Java guarantees `close()` runs
automatically when the block exits, success or failure, without a manual
`finally`.

```java
public class ManagedConnection implements AutoCloseable {
    private final String name;

    public ManagedConnection(String name) {
        this.name = name;
        System.out.println("Opening connection: " + name);
    }

    public void query(String sql) {
        System.out.println("Running on " + name + ": " + sql);
    }

    @Override
    public void close() {
        System.out.println("Closing connection: " + name);
    }
}

try (ManagedConnection conn = new ManagedConnection("db-primary")) {
    conn.query("SELECT * FROM users");
}
// Output:
// Opening connection: db-primary
// Running on db-primary: SELECT * FROM users
// Closing connection: db-primary
```

Multiple resources can be declared in the same `try`, separated by `;` — they
close in reverse order of declaration:

```java
try (ManagedConnection a = new ManagedConnection("a");
     ManagedConnection b = new ManagedConnection("b")) {
    a.query("...");
    b.query("...");
}
// closes b first, then a
```

This is exactly the pattern [Level 1's contact book
project](../level-1/10-project-contact-book.md) used with
`Files.newBufferedReader`/`newBufferedWriter`, and it's the standard idiom for
files, streams, and database connections covered in
[Module 6](06-file-io-nio.md).

## Multi-catch recap

As shown in [Level 1](../level-1/08-exception-handling.md), a single `catch`
can handle several exception types with `|` when the handling logic is
identical:

```java
try {
    ManagedConnection conn = new ManagedConnection("temp");
    conn.query(null);
} catch (NullPointerException | IllegalStateException e) {
    System.out.println("Connection problem: " + e.getMessage());
}
```

## Suppressed exceptions

If both the `try` block *and* a resource's `close()` throw, the exception
from `try` is the one propagated, and the `close()` failure is attached to it
as a **suppressed exception** rather than silently lost:

```java
class FlakyResource implements AutoCloseable {
    @Override
    public void close() {
        throw new IllegalStateException("close failed");
    }
}

try (FlakyResource r = new FlakyResource()) {
    throw new RuntimeException("main failure");
} catch (RuntimeException e) {
    System.out.println("Caught: " + e.getMessage());               // main failure
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println("Suppressed: " + suppressed.getMessage()); // close failed
    }
}
```

## Exercise

Write a custom checked exception `InvalidTemperatureException` with a message
constructor. Write a class `Thermostat` whose `setTemperature(double degrees)`
method throws that exception if `degrees` is below -50 or above 150. Then
write a class `LoggingResource implements AutoCloseable` whose constructor
prints "opened" and whose `close()` prints "closed". In `main`, use a
try-with-resources block that creates a `LoggingResource` and, inside it,
calls `setTemperature` with an invalid value inside a `try`/`catch` that
prints the exception's message — confirm "closed" still prints even though
the temperature call failed.
