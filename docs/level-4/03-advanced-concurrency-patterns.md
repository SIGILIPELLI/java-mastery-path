# 03 · Advanced Concurrency Patterns

[Level 3](../level-3/02-util-concurrent.md) covered thread pools and the
`java.util.concurrent` primitives. This module goes further: composing
asynchronous work with `CompletableFuture` pipelines, and Java 21's virtual
threads, which change the economics of writing blocking-style concurrent code.

## Chaining `CompletableFuture` stages

A `CompletableFuture` represents a value that will exist eventually. Instead
of blocking on each step, you chain transformations that run when the
previous stage completes.

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class OrderPipeline {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Order> orderFuture = CompletableFuture
            .supplyAsync(() -> fetchOrder(42))          // runs on the common ForkJoinPool
            .thenApply(order -> applyDiscount(order));   // transforms the result

        // thenCompose "flattens" -- use when the next step itself returns a CompletableFuture
        CompletableFuture<Invoice> invoiceFuture = orderFuture
            .thenCompose(order -> generateInvoiceAsync(order));

        System.out.println(invoiceFuture.get());   // blocks only here, at the edge
    }

    static Order fetchOrder(long id) { return new Order(id, 100.0); }
    static Order applyDiscount(Order o) { return new Order(o.id(), o.total() * 0.9); }
    static CompletableFuture<Invoice> generateInvoiceAsync(Order o) {
        return CompletableFuture.supplyAsync(() -> new Invoice(o.id(), o.total()));
    }

    record Order(long id, double total) {}
    record Invoice(long orderId, double amount) {}
}
```

`thenApply` transforms a plain value; `thenCompose` chains onto another
async operation without nesting futures inside futures (`CompletableFuture<CompletableFuture<T>>`).

## Combining independent futures

`thenCombine` joins two independent futures once both complete; `allOf` waits
for an arbitrary number of them.

```java
CompletableFuture<Double> priceFuture = CompletableFuture.supplyAsync(() -> fetchPrice("SKU-1"));
CompletableFuture<Integer> stockFuture = CompletableFuture.supplyAsync(() -> fetchStock("SKU-1"));

// Runs once BOTH complete, combining their results
CompletableFuture<String> summary = priceFuture.thenCombine(stockFuture,
    (price, stock) -> "Price: $" + price + ", in stock: " + stock);

// allOf waits for a whole batch -- useful for fan-out/fan-in
List<CompletableFuture<Double>> priceChecks = skus.stream()
    .map(sku -> CompletableFuture.supplyAsync(() -> fetchPrice(sku)))
    .toList();

CompletableFuture<Void> allDone = CompletableFuture.allOf(
    priceChecks.toArray(new CompletableFuture[0]));

allDone.thenRun(() -> {
    // all price checks finished; safe to call .join() on each without blocking further
    List<Double> prices = priceChecks.stream().map(CompletableFuture::join).toList();
    System.out.println(prices);
});
```

## Error recovery: `handle` and `exceptionally`

Unhandled exceptions inside a pipeline propagate to the final `get()`/`join()`
as a `CompletionException`. Handle them inline instead of letting the whole
chain blow up:

```java
CompletableFuture<Integer> stock = CompletableFuture
    .supplyAsync(() -> fetchStock("SKU-404"))          // might throw
    .exceptionally(ex -> {
        System.out.println("Stock check failed: " + ex.getMessage());
        return 0;                                       // fallback value
    });

// handle sees BOTH the result and the exception (whichever is non-null)
// and lets you recover regardless of which branch happened
CompletableFuture<String> report = CompletableFuture
    .supplyAsync(() -> fetchPrice("SKU-404"))
    .handle((price, ex) -> {
        if (ex != null) {
            return "Price unavailable";
        }
        return "Price: $" + price;
    });
```

`exceptionally` only fires on failure and must return the same type as the
success path. `handle` always fires, giving a single place to normalize both
outcomes.

## Java 21 virtual threads

A platform thread maps 1:1 to an OS thread — expensive to create, and
limited to a few thousand per JVM before memory pressure and context-switch
overhead dominate. A **virtual thread** is a lightweight thread managed by
the JVM; millions can exist at once, because they're only mounted onto a
real OS thread while actively running, and unmounted while blocked on I/O.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class VirtualThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        // One virtual thread PER TASK -- cheap enough that this is fine even for 100,000 tasks
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 10_000; i++) {
                int taskId = i;
                executor.submit(() -> {
                    simulateBlockingCall(taskId);   // e.g. an HTTP call or DB query
                    return null;
                });
            }
        } // executor.close() waits for all submitted tasks to finish
    }

    static void simulateBlockingCall(int id) {
        try {
            Thread.sleep(100);   // blocking sleep -- fine, the carrier thread is freed meanwhile
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

Ordinary blocking code — `Thread.sleep`, JDBC calls, blocking HTTP clients —
"just works" with virtual threads without rewriting it in a reactive style.
When a virtual thread blocks on I/O, the JVM unmounts it from its carrier
platform thread, which is then free to run other virtual threads.

## Structured concurrency (preview)

`StructuredTaskScope` (a preview API in Java 21) treats a group of related
subtasks as a single unit: if one fails, the others are cancelled, and the
scope doesn't exit until all subtasks are accounted for.

```java
import java.util.concurrent.StructuredTaskScope;

// Requires --enable-preview to compile and run on JDK 21
Order fetchOrderDetails(long orderId) throws InterruptedException, ExecutionException {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        var userTask = scope.fork(() -> fetchUser(orderId));
        var itemsTask = scope.fork(() -> fetchItems(orderId));

        scope.join();            // wait for both
        scope.throwIfFailed();   // propagate first failure, cancelling the other task

        return new Order(userTask.get(), itemsTask.get());
    }
}
```

Unlike a bare `CompletableFuture.allOf`, the scope guarantees that if
`fetchItems` fails, `fetchUser` is cancelled too — no orphaned background
work outliving the operation that spawned it.

## When virtual threads help vs. when they don't

| Workload | Recommendation |
|----------|-----------------|
| High-concurrency I/O-bound (HTTP calls, DB queries, file I/O) | Virtual threads shine — cheap to block, scale to huge counts |
| CPU-bound (image processing, heavy computation) | Platform threads on a fixed-size pool sized to core count still apply — more virtual threads doesn't create more CPU |
| Long-lived pooled worker threads holding native resources | Keep platform threads — virtual threads are meant to be short-lived, one-per-task |
| Code using `synchronized` blocks around blocking I/O | Can "pin" the virtual thread to its carrier — prefer `ReentrantLock` in hot paths |

## Exercise

Write a pipeline that fetches a user profile and their order history
concurrently via two `CompletableFuture.supplyAsync` calls, combines them
with `thenCombine` into a single summary string, and uses `exceptionally` so
that if the order-history fetch fails, the summary still prints with
"orders unavailable" instead of throwing. Then rewrite the same fan-out
using `Executors.newVirtualThreadPerTaskExecutor()` and compare how many
lines each approach takes.
