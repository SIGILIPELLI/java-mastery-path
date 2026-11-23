---
description: "JVM Internals, Garbage Collection & Performance Profiling — Part 1: JVM Internals & Garbage Collection; Part 2: Performance Profiling & Optimization."
---

# 09 · JVM Internals, Garbage Collection & Performance Profiling

This module covers how the JVM runs your code and manages memory, then how to measure and improve performance using that knowledge: first the JVM's structure and garbage collector, then profiling tools, benchmarking pitfalls and common optimizations.

## Part 1 · JVM Internals & Garbage Collection

Java code doesn't run directly on the CPU — it runs on the Java Virtual
Machine (JVM), which loads compiled bytecode, manages memory, and translates
it into native instructions. Understanding the JVM's structure helps explain
both performance and the automatic memory management that makes Java easier
(if less predictable) than manual memory management.

### JVM architecture overview

| Component | Role |
|-----------|------|
| Class loader | Loads `.class` files into memory, links and initializes classes |
| Heap | Stores all objects and arrays; shared across all threads |
| Stack | Per-thread; stores method call frames, local variables, partial results |
| Metaspace | Stores class metadata (replaces the old "permanent generation") |
| Execution engine | Interprets bytecode, or compiles it to native code via the JIT compiler |
| Garbage collector | Reclaims heap memory occupied by objects no longer reachable |

Every thread gets its own stack (so recursion in one thread can't corrupt
another thread's local variables), but all threads share the same heap —
which is exactly why concurrent access to heap-allocated objects needs the
coordination covered in
[Level 3, Module 1](01-concurrency-basics.md).

### Class loading, briefly

Before any of your code runs, the class loader subsystem finds a class's
bytecode (from the classpath, a JAR, wherever) and takes it through three
phases: **loading** (reading the `.class` bytes into memory), **linking**
(verifying the bytecode is well-formed, allocating storage for static
fields), and **initialization** (running static initializers and static field
assignments, top to bottom in source order).

```java
public class LoadingDemo {
    static {
        System.out.println("Static initializer running");
    }

    static int counter = initCounter();

    static int initCounter() {
        System.out.println("Initializing counter");
        return 0;
    }

    public static void main(String[] args) {
        System.out.println("main() running");
    }
}
// Output:
// Static initializer running
// Initializing counter
// main() running
```

A class is only loaded and initialized the first time it's actually
referenced (a `new`, a static method call, accessing a static field) — this
"lazy initialization" is why a class you never use adds essentially no
startup cost.

### The generational heap

Most objects die young — a temporary `String` built inside a loop, a local
list used and discarded within one method call. The HotSpot JVM exploits this
observation by splitting the heap into **generations**:

- **Young generation** — new objects are allocated here first (in an area
  called "Eden"). Minor GCs run frequently but are cheap, since most objects
  in Eden are already garbage by the time a collection runs.
- **Old generation (tenured)** — objects that survive several young-generation
  collections get "promoted" here. Old-gen collections are less frequent but
  more expensive, since they scan a much larger region.

```text
Heap
 ├── Young Generation
 │     ├── Eden          (new objects allocated here)
 │     ├── Survivor 0
 │     └── Survivor 1
 └── Old Generation       (long-lived, promoted objects)
```

### Common GC algorithms

| Collector | Flag | Characteristics |
|-----------|------|------------------|
| Serial GC | `-XX:+UseSerialGC` | Single-threaded; simplest, best for small heaps/apps |
| Parallel GC | `-XX:+UseParallelGC` | Multi-threaded stop-the-world; maximizes throughput |
| G1 (Garbage-First) | `-XX:+UseG1GC` | Default since Java 9; balances throughput and pause times |
| ZGC | `-XX:+UseZGC` | Ultra-low pause times (sub-millisecond), scales to huge heaps |

G1 is the JVM's default collector today and a reasonable choice for most
applications; ZGC and Shenandoah trade some throughput for dramatically lower
pause times, useful for latency-sensitive services.

### Heap sizing flags

```bash
java -Xms256m -Xmx1024m -XX:+UseG1GC -jar myapp.jar
```

- `-Xms` — initial heap size (256 MB here).
- `-Xmx` — maximum heap size (1024 MB here) — if the JVM can't reclaim enough
  memory to satisfy an allocation once it's hit this ceiling, it throws
  `OutOfMemoryError`.
- `-XX:+UseG1GC` — selects the G1 collector explicitly (already the default
  on modern JVMs, but useful to know how to force it).

### Observing allocation and GC pressure

```java
public class GcPressureDemo {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        // Allocate a large number of short-lived objects to pressure the young generation
        for (int i = 0; i < 5_000_000; i++) {
            String s = "object-" + i;   // each concatenation allocates a new String
            if (s.length() > 1000) {    // never true -- keeps the compiler from optimizing it away
                System.out.println(s);
            }
        }

        long elapsed = System.currentTimeMillis() - start;
        System.out.println("Allocated 5,000,000 short-lived strings in " + elapsed + " ms");
    }
}
```

Run it with GC logging enabled to watch collections happen in real time:

```bash
java -Xlog:gc GcPressureDemo
```

```text
[0.012s][info][gc] Using G1
[0.045s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 12M->3M(64M) 4.112ms
[0.089s][info][gc] GC(1) Pause Young (Normal) (G1 Evacuation Pause) 15M->3M(64M) 3.887ms
...
Allocated 5,000,000 short-lived strings in 612 ms
```

(`-Xlog:gc` is the modern unified logging flag; older material may reference
`-verbose:gc`, which still works but is less detailed.)

### Inspecting a running JVM with `jcmd`

`jcmd` is a general-purpose diagnostic tool bundled with the JDK — no code
changes needed:

```bash
jcmd <pid> GC.heap_info      # summary of heap regions and usage
jcmd <pid> GC.run            # request a full garbage collection
jcmd <pid> VM.flags          # show the JVM flags currently in effect
```

### How It Actually Works

The heap is generationally split because most objects die young (the
"weak generational hypothesis"): the **young generation** (Eden + two
Survivor spaces) is collected frequently with a fast copying collector
— live objects are copied between Survivor spaces and get their
**age** incremented in the object header each survived cycle, promoted
to the **old generation** once they cross a tenuring threshold. This
generational split is why minor GCs are typically sub-millisecond to a
few milliseconds while full/major GCs (scanning the much larger old
generation) can pause for much longer.

G1 (the default collector since Java 9) divides the heap into fixed-size
**regions** instead of contiguous generations, tracks "remembered sets"
(which regions hold references into which other regions) to avoid
scanning the whole heap for roots, and picks the regions with the most
garbage to collect first ("garbage-first") to hit a target pause-time
goal rather than a fixed heap-size trigger. ZGC/Shenandoah go further,
doing almost all marking and relocation **concurrently with application
threads**, using colored pointers or load barriers so a thread reading a
reference to a moved object gets redirected on the fly instead of the
world stopping.

Object headers matter beyond GC: the **mark word** also encodes lock
state (unlocked/biased/thin/fat) and identity hash code, which is why
calling `System.identityHashCode()` on an object can force the JVM to
materialize a hash into the header it might otherwise never need to
compute.

### Exercise

Write a program that builds a `List<byte[]>`, repeatedly adding a new 1 MB
`byte[]` array in a loop, printing the iteration count every 50 iterations.
Run it with `-Xmx200m -Xlog:gc` and observe both the GC log output and, once
the list grows too large for the heap, the resulting
`OutOfMemoryError: Java heap space` — then explain in a comment why holding a
reference to every array in the list (instead of discarding each one) is what
prevents the garbage collector from reclaiming them.

## Part 2 · Performance Profiling & Optimization

Before optimizing anything, you need to know *where* time and memory are
actually being spent — guessing usually leads to effort spent on code that was
never the bottleneck. This module covers the JDK's built-in diagnostic tools
and a handful of well-known optimization techniques.

### Built-in diagnostic tools

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

### Java Flight Recorder

JFR records detailed runtime events (GC pauses, method sampling, allocation
profiles, lock contention) with low overhead — safe to enable even in
production.

```bash
java -XX:StartFlightRecording=duration=60s,filename=recording.jfr -jar myapp.jar
```

The resulting `recording.jfr` file can be opened in VisualVM or the standalone
JDK Mission Control tool for a detailed, visual breakdown of where time went.

### Manual benchmarking with `System.nanoTime()`

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

### Why microbenchmarks are tricky

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

### Optimization technique: `StringBuilder` vs. concatenation in loops

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

### Optimization technique: avoiding unnecessary allocation

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

### Optimization technique: choosing the right collection

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

### How It Actually Works

HotSpot's name is literal: the interpreter runs bytecode directly at
first, while a background thread counts method invocations and loop
back-edges; once a method crosses a threshold it's compiled by the
**C1 (client) compiler** for fast, lightly-optimized native code, and if
it stays hot, later recompiled by **C2 (server)** with aggressive
optimizations — inlining, escape analysis, loop unrolling — informed by
actual runtime profile data (branch frequencies, observed types at call
sites) that a purely static compiler could never have. This is why a
Java benchmark's first few thousand iterations are typically much slower
than its steady state, and why microbenchmarks without a JIT warm-up
phase (JMH exists specifically to handle this correctly) produce
misleading numbers.

**Escape analysis** lets C2 prove an object never leaves the current
method/thread and, when it can, allocate it on the stack (or eliminate
the allocation entirely, "scalar replacement") instead of the heap —
real, load-bearing optimization, not a JIT curiosity, and one reason
"avoid allocation" advice is less absolute in Java than in
non-JIT'd languages.

A profiler like async-profiler samples real stack traces using
safepoint-biased or (better) **AsyncGetCallTrace**/perf-event-based
sampling to avoid the classic safepoint bias where naive JVM profilers
only ever see threads paused at safepoints, systematically
under-sampling code that runs between safepoint polls.

### 🔀 See this in another language

- [Python — 10 · Profiling & Performance](https://learnmasterypath.com/python-mastery-path/level-3/09-profiling-performance/)
- [C++ — 09 · Performance Profiling](https://learnmasterypath.com/cpp-mastery-path/level-3/09-performance-profiling/)
- [Ruby — 06 · Performance & Profiling](https://learnmasterypath.com/ruby-mastery-path/level-3/06-performance-profiling/)

### Exercise

Write a benchmark comparing `ArrayList.contains()` against `HashSet.contains()`
when checking membership of 10,000 lookups against a collection of 20,000
integers. Populate both collections with the same data, time each set of
10,000 `contains()` calls with `System.nanoTime()`, and print the elapsed
milliseconds for each — then write a one-sentence comment explaining the
result in terms of each collection's underlying data structure.
