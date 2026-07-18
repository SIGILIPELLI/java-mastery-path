# 10 · Performance Profiling & Optimization

Before optimizing anything, you need to know *where* time and memory are
actually being spent — guessing usually leads to effort spent on code that was
never the bottleneck. This module covers the JDK's built-in diagnostic tools
and a handful of well-known optimization techniques.

## Built-in diagnostic tools

| Tool | Purpose |
|------|---------|
| `jcmd` | General-purpose diagnostic command tool (GC, thread dumps, flags, JFR control) |
| `jstack` (or `jcmd <pid> Thread.print`) | Dumps all thread stack traces — useful for diagnosing deadlocks/hangs |
| `jmap` (or `jcmd <pid> GC.heap_info` / `GC.class_histogram`) | Heap usage summaries, class instance counts |
| VisualVM | Free GUI tool bundled separately; live CPU/memory graphs, profiler, heap dumps |
| JFR (Java Flight Recorder) | Low-overhead, production-safe event recorder built into the JVM |

```bash
# Take a thread dump of a running JVM to check for deadlocks or stuck threads
jcmd <pid> Thread.print

# Show a histogram of live object counts on the heap, sorted by size
jcmd <pid> GC.class_histogram
```

## Java Flight Recorder

JFR records detailed runtime events (GC pauses, method sampling, allocation
profiles, lock contention) with low overhead — safe to enable even in
production.

```bash
java -XX:StartFlightRecording=duration=60s,filename=recording.jfr -jar myapp.jar
```

The resulting `recording.jfr` file can be opened in VisualVM or the standalone
JDK Mission Control tool for a detailed, visual breakdown of where time went.

## Manual benchmarking with `System.nanoTime()`

For a quick sanity check, timing code by hand is fine:

```java
public class ManualBenchmark {
    public static void main(String[] args) {
        int n = 1_000_000;

        long start = System.nanoTime();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < n; i++) {
            sb.append(i);
        }
        long elapsedMs = (System.nanoTime() - start) / 1_000_000;

        System.out.println("StringBuilder: " + elapsedMs + " ms, length=" + sb.length());
    }
}
```

## Why microbenchmarks are tricky

Naively timing a single run like the example above is easy to get wrong:

- **JIT warm-up** — the JVM interprets bytecode at first, then compiles "hot"
  methods to optimized native code after enough invocations. The *first* few
  thousand iterations of a loop can run far slower than the steady state,
  skewing results measured only once.
- **Dead code elimination** — the JIT can notice a computed value is never
  used and optimize the whole computation away, measuring nothing.
- **Garbage collection interference** — a GC pause happening to land inside
  your timed region adds noise unrelated to the code being measured.
- **No statistical rigor** — a single measurement can't distinguish a real
  effect from run-to-run system noise (other processes, CPU frequency
  scaling, etc.).

```java
public class WarmupDemo {
    public static void main(String[] args) {
        int n = 10_000_000;

        // "Cold" timing -- includes interpretation and JIT compilation overhead
        long t1 = time(() -> sumSquares(n));
        // "Warm" timing -- JIT has likely already compiled sumSquares by now
        long t2 = time(() -> sumSquares(n));

        System.out.println("First run:  " + t1 + " ms");
        System.out.println("Second run: " + t2 + " ms");
        // The second run is typically noticeably faster -- the same code,
        // just measured after the JIT has kicked in.
    }

    static long sumSquares(int n) {
        long sum = 0;
        for (int i = 0; i < n; i++) sum += (long) i * i;
        return sum;
    }

    static long time(Runnable task) {
        long start = System.nanoTime();
        task.run();
        return (System.nanoTime() - start) / 1_000_000;
    }
}
```

For real, trustworthy microbenchmarks, use **JMH (Java Microbenchmark
Harness)** — a purpose-built tool from the OpenJDK team that handles warm-up
iterations, dead-code elimination pitfalls, and statistical reporting
automatically. Manual `nanoTime()` timing is fine for rough, order-of-magnitude
checks; JMH is the right tool once a decision actually depends on the number.

## Optimization technique: `StringBuilder` vs. concatenation in loops

```java
public class ConcatBenchmark {
    public static void main(String[] args) {
        int n = 50_000;

        long start = System.nanoTime();
        String result = "";
        for (int i = 0; i < n; i++) {
            result += i;   // each += creates a brand-new String -- O(n^2) overall
        }
        long slow = (System.nanoTime() - start) / 1_000_000;

        start = System.nanoTime();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < n; i++) {
            sb.append(i);   // mutates an internal buffer -- O(n) overall
        }
        String fast = sb.toString();
        long fastMs = (System.nanoTime() - start) / 1_000_000;

        System.out.println("String +=      : " + slow + " ms");
        System.out.println("StringBuilder  : " + fastMs + " ms");
        // StringBuilder is typically an order of magnitude faster at this size
    }
}
```

Each `String` is immutable, so `result += i` allocates an entirely new
`String` and copies the old contents every iteration. `StringBuilder` grows an
internal, mutable buffer instead.

## Optimization technique: avoiding unnecessary allocation

```java
// Wasteful -- creates a new Integer wrapper via autoboxing every loop iteration
long sum = 0;
for (Integer i = 0; i < 1_000_000; i++) {
    sum += i;
}

// Better -- primitive int avoids boxing entirely
long sumFast = 0;
for (int i = 0; i < 1_000_000; i++) {
    sumFast += i;
}
```

## Optimization technique: choosing the right collection

| Need | Prefer | Avoid |
|------|--------|-------|
| Frequent lookups by key | `HashMap` | Linear scan over a `List` |
| Frequent insert/remove at both ends | `ArrayDeque` | `LinkedList` (worse cache locality) |
| Index-based random access | `ArrayList` | `LinkedList` (O(n) get) |
| Uniqueness with no ordering need | `HashSet` | Manually checking `list.contains()` before adding |

Picking the wrong collection for the access pattern is one of the most common
avoidable sources of quadratic behavior in otherwise-correct code — e.g.
calling `list.contains(x)` inside a loop over a large `ArrayList` turns an
O(n) task into O(n²), whereas the same check against a `HashSet` stays O(n)
overall.

## Exercise

Write a benchmark comparing `ArrayList.contains()` against `HashSet.contains()`
when checking membership of 10,000 lookups against a collection of 20,000
integers. Populate both collections with the same data, time each set of
10,000 `contains()` calls with `System.nanoTime()`, and print the elapsed
milliseconds for each — then write a one-sentence comment explaining the
result in terms of each collection's underlying data structure.
