# 02 · java.util.concurrent

Manually creating and joining `Thread` objects gets unwieldy fast. The
`java.util.concurrent` package provides higher-level building blocks — thread
pools, futures, and composable asynchronous pipelines — for managing
concurrent work.

## Thread pools with `ExecutorService`

An `ExecutorService` manages a pool of reusable worker threads so you don't pay
the cost of creating a new OS thread per task:

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorDemo {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(3);   // 3 worker threads

        for (int i = 1; i <= 5; i++) {
            int taskId = i;
            pool.submit(() -> {
                System.out.println("Task " + taskId + " running on " + Thread.currentThread().getName());
            });
        }

        pool.shutdown();   // stop accepting new tasks, let submitted ones finish
        pool.awaitTermination(5, java.util.concurrent.TimeUnit.SECONDS);
        System.out.println("All tasks complete");
    }
}
```

Five tasks share only three threads — once a thread finishes a task, the pool
reassigns it to the next queued task.

## Submitting `Callable` tasks and using `Future`

`Runnable` can't return a value or throw a checked exception. `Callable<T>`
can — and submitting one gives you back a `Future<T>` representing the
result, which may not be ready yet.

```java
import java.util.concurrent.*;

public class CallableFutureDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(2);

        Callable<Integer> sumTask = () -> {
            int sum = 0;
            for (int i = 1; i <= 1000; i++) sum += i;
            return sum;
        };

        Future<Integer> future = pool.submit(sumTask);

        System.out.println("Doing other work while the task runs...");
        int result = future.get();   // blocks until the result is ready
        System.out.println("Sum: " + result);   // Sum: 500500

        pool.shutdown();
    }
}
```

`future.get()` blocks the calling thread until the task completes (an
overload also accepts a timeout: `future.get(2, TimeUnit.SECONDS)`, throwing
`TimeoutException` if it's too slow). If the task threw an exception,
`get()` wraps it in an `ExecutionException`.

## Shutting down executors properly

An `ExecutorService`'s threads keep the JVM alive, so forgetting to shut one
down is a common resource leak.

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
try {
    // submit tasks...
} finally {
    pool.shutdown();                                  // reject new tasks, finish queued ones
    if (!pool.awaitTermination(5, TimeUnit.SECONDS)) {
        pool.shutdownNow();                           // force-cancel anything still running
    }
}
```

Since Java 19, `ExecutorService` implements `AutoCloseable`, so try-with-resources
works too:

```java
try (ExecutorService pool = Executors.newFixedThreadPool(4)) {
    pool.submit(() -> System.out.println("Task running"));
}   // close() calls shutdown() + waits for tasks automatically
```

## `CompletableFuture` basics

`CompletableFuture` represents an asynchronous computation you can chain,
combine, and react to without ever blocking manually.

```java
import java.util.concurrent.CompletableFuture;

public class CompletableFutureDemo {
    public static void main(String[] args) {
        CompletableFuture<Integer> future = CompletableFuture
                .supplyAsync(() -> {          // runs on the common ForkJoinPool
                    System.out.println("Fetching a number...");
                    return 21;
                })
                .thenApply(n -> n * 2);       // transform the result once available

        future.thenAccept(result -> System.out.println("Final result: " + result));

        future.join();   // block main() so the async chain finishes before exit
        // Fetching a number...
        // Final result: 42
    }
}
```

## Combining two async computations

```java
CompletableFuture<Integer> priceFuture = CompletableFuture.supplyAsync(() -> 100);
CompletableFuture<Double> taxRateFuture = CompletableFuture.supplyAsync(() -> 0.08);

CompletableFuture<Double> totalFuture = priceFuture.thenCombine(
        taxRateFuture,
        (price, taxRate) -> price + (price * taxRate)
);

System.out.println("Total: " + totalFuture.join());   // Total: 108.0
```

## Handling exceptions in a `CompletableFuture` chain

```java
CompletableFuture<Integer> risky = CompletableFuture.supplyAsync(() -> {
    if (true) {
        throw new RuntimeException("boom");
    }
    return 42;
});

CompletableFuture<Integer> recovered = risky.exceptionally(ex -> {
    System.out.println("Recovered from: " + ex.getMessage());
    return -1;   // fallback value
});

System.out.println(recovered.join());
// Recovered from: java.lang.RuntimeException: boom
// -1
```

`handle((result, ex) -> ...)` is similar but runs whether or not an exception
occurred, letting you inspect both possibilities in one place.

## A quick look ahead: virtual threads

Java 21 introduced **virtual threads** — extremely lightweight threads managed
by the JVM instead of the OS, letting you write simple blocking-style code
(one thread per task, even millions of tasks) without the overhead of
platform threads:

```java
try (ExecutorService pool = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        pool.submit(() -> {
            // each task gets its own cheap virtual thread
        });
    }
}
```

This module only scratches the surface — structured concurrency, virtual
thread pinning, and scaling patterns are covered in depth in Level 4.

| Type | Purpose |
|------|---------|
| `ExecutorService` | Manages a pool of worker threads |
| `Runnable` | Task with no return value |
| `Callable<T>` | Task that returns a value (or throws a checked exception) |
| `Future<T>` | Handle to a result that may not be ready yet |
| `CompletableFuture<T>` | Composable, non-blocking asynchronous pipeline |
| Virtual thread | Lightweight, JVM-managed thread (Java 21+) |

## Exercise

Using an `ExecutorService` with a fixed pool of 4 threads, submit 8 `Callable<Integer>`
tasks that each compute the square of their index (0-7). Collect all 8
`Future<Integer>` objects in a list, then loop over them calling `get()` to
print each result in order. Afterward, rewrite the same computation using
`CompletableFuture.supplyAsync` for each task and `CompletableFuture.allOf`
to wait for all of them before printing a "done" message.
