# 01 · Concurrency Basics

A single-threaded program does one thing at a time. Concurrency lets a program
run multiple independent tasks that make progress at (roughly) the same time —
useful for I/O-bound work, responsive UIs, and using multiple CPU cores.

## Creating a thread — extending `Thread`

```java
public class HelloThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running on: " + Thread.currentThread().getName());
    }

    public static void main(String[] args) throws InterruptedException {
        HelloThread t = new HelloThread();
        t.start();   // starts a new thread and calls run() on it
        t.join();    // wait for t to finish before continuing
        System.out.println("Main thread done");
    }
}
// Output (order may vary slightly, but join() guarantees t finishes first):
// Running on: Thread-0
// Main thread done
```

Calling `run()` directly would just run the code on the *current* thread —
`start()` is what actually spins up a new OS-backed thread of execution.

## Creating a thread — implementing `Runnable`

Extending `Thread` uses up your one shot at inheritance and mixes "what to run"
with "how it runs." Implementing `Runnable` is preferred — it's just a task:

```java
public class RunnableDemo {
    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            System.out.println("Task running on " + Thread.currentThread().getName());
        };

        Thread t1 = new Thread(task, "worker-1");
        Thread t2 = new Thread(task, "worker-2");
        t1.start();
        t2.start();

        t1.join();
        t2.join();
        System.out.println("Both workers finished");
    }
}
```

## Starting and joining multiple threads

```java
public class MultiWorker {
    public static void main(String[] args) throws InterruptedException {
        Thread[] workers = new Thread[4];

        for (int i = 0; i < workers.length; i++) {
            int id = i;   // effectively final, safe to capture in the lambda
            workers[i] = new Thread(() -> {
                System.out.println("Worker " + id + " starting");
                // simulate some work
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Worker " + id + " done");
            });
            workers[i].start();
        }

        for (Thread w : workers) {
            w.join();   // wait for every worker before printing the summary
        }
        System.out.println("All workers finished");
    }
}
```

## Race conditions

When two threads read and write shared, mutable state without coordination,
the result depends on unpredictable timing — a **race condition**.

```java
public class RaceConditionDemo {
    static int counter = 0;

    static void increment() {
        counter++;   // NOT atomic: read, add 1, write back -- three steps
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Expected 200000, got: " + counter);
        // Often prints something less than 200000 -- lost updates from
        // interleaved read-modify-write operations on the same variable.
    }
}
```

## `synchronized` methods and blocks

`synchronized` ensures only one thread at a time can execute a given piece of
code guarded by the same lock, fixing the race above:

```java
public class SafeCounter {
    private int counter = 0;

    // synchronized method -- locks on "this" for the whole method body
    public synchronized void increment() {
        counter++;
    }

    public synchronized int get() {
        return counter;
    }

    public static void main(String[] args) throws InterruptedException {
        SafeCounter sc = new SafeCounter();

        Runnable task = () -> {
            for (int i = 0; i < 100_000; i++) sc.increment();
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Counter: " + sc.get());   // always 200000
    }
}
```

You can also synchronize on a smaller block instead of a whole method, which
reduces how long the lock is held:

```java
public class SafeBlockCounter {
    private int counter = 0;
    private final Object lock = new Object();   // a dedicated lock object

    public void increment() {
        synchronized (lock) {
            counter++;
        }
    }

    public int get() {
        synchronized (lock) {
            return counter;
        }
    }
}
```

## The intrinsic lock concept

Every Java object has an associated **intrinsic lock** (a.k.a. monitor lock).
`synchronized` acquires that lock before entering the block and releases it on
exit — even if an exception is thrown. Only one thread can hold a given
object's lock at a time; any other thread trying to enter a block synchronized
on the same object blocks until the lock is free. `synchronized void m()` on an
instance method is shorthand for synchronizing on `this`; a `static
synchronized` method synchronizes on the class object (`SomeClass.class`)
instead.

## `volatile` — visibility without locking

`synchronized` provides both mutual exclusion *and* visibility guarantees
(changes made by one thread become visible to others). Sometimes you only need
visibility — for a simple flag read by multiple threads:

```java
public class VolatileFlagDemo {
    private static volatile boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            int iterations = 0;
            while (running) {
                iterations++;   // busy loop until flag flips
            }
            System.out.println("Stopped after noticing the flag, iterations=" + iterations);
        });

        worker.start();
        Thread.sleep(100);
        running = false;   // without volatile, the worker thread might never see this update
        worker.join();
    }
}
```

`volatile` does **not** make compound operations like `counter++` atomic — for
that you still need `synchronized` or an atomic class such as
`AtomicInteger` (covered in [Level 3, Module 2](02-util-concurrent.md)).

## Thread-safety basics

| Term | Meaning |
|------|---------|
| Thread-safe | Behaves correctly when accessed by multiple threads concurrently |
| Race condition | Outcome depends on unpredictable thread interleaving |
| Critical section | Code that must not run concurrently on shared state |
| Intrinsic lock (monitor) | Per-object lock acquired/released by `synchronized` |
| `volatile` | Guarantees visibility of a variable's latest value across threads |
| Atomicity | An operation that appears to happen as a single, indivisible step |

The simplest way to avoid these problems entirely is to avoid shared mutable
state: prefer immutable objects and confine mutable data to a single thread
whenever you can.

## Exercise

Write a `BankAccount` class with a private `long balanceCents` field and
synchronized `deposit(long cents)` and `withdraw(long cents)` methods (the
latter should throw `IllegalStateException` if funds are insufficient). In
`main`, create one `BankAccount` starting at 0, then launch 10 threads that
each deposit 100 cents 1,000 times concurrently. Join all threads and print
the final balance, confirming it is exactly 1,000,000 — then try removing
`synchronized` and observe how the result becomes unreliable.
