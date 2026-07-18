# 09 · JVM Internals & Garbage Collection Basics

Java code doesn't run directly on the CPU — it runs on the Java Virtual
Machine (JVM), which loads compiled bytecode, manages memory, and translates
it into native instructions. Understanding the JVM's structure helps explain
both performance and the automatic memory management that makes Java easier
(if less predictable) than manual memory management.

## JVM architecture overview

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

## Class loading, briefly

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

## The generational heap

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

## Common GC algorithms

| Collector | Flag | Characteristics |
|-----------|------|------------------|
| Serial GC | `-XX:+UseSerialGC` | Single-threaded; simplest, best for small heaps/apps |
| Parallel GC | `-XX:+UseParallelGC` | Multi-threaded stop-the-world; maximizes throughput |
| G1 (Garbage-First) | `-XX:+UseG1GC` | Default since Java 9; balances throughput and pause times |
| ZGC | `-XX:+UseZGC` | Ultra-low pause times (sub-millisecond), scales to huge heaps |

G1 is the JVM's default collector today and a reasonable choice for most
applications; ZGC and Shenandoah trade some throughput for dramatically lower
pause times, useful for latency-sensitive services.

## Heap sizing flags

```bash
java -Xms256m -Xmx1024m -XX:+UseG1GC -jar myapp.jar
```

- `-Xms` — initial heap size (256 MB here).
- `-Xmx` — maximum heap size (1024 MB here) — if the JVM can't reclaim enough
  memory to satisfy an allocation once it's hit this ceiling, it throws
  `OutOfMemoryError`.
- `-XX:+UseG1GC` — selects the G1 collector explicitly (already the default
  on modern JVMs, but useful to know how to force it).

## Observing allocation and GC pressure

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

## Inspecting a running JVM with `jcmd`

`jcmd` is a general-purpose diagnostic tool bundled with the JDK — no code
changes needed:

```bash
jcmd <pid> GC.heap_info      # summary of heap regions and usage
jcmd <pid> GC.run            # request a full garbage collection
jcmd <pid> VM.flags          # show the JVM flags currently in effect
```

## Exercise

Write a program that builds a `List<byte[]>`, repeatedly adding a new 1 MB
`byte[]` array in a loop, printing the iteration count every 50 iterations.
Run it with `-Xmx200m -Xlog:gc` and observe both the GC log output and, once
the list grows too large for the heap, the resulting
`OutOfMemoryError: Java heap space` — then explain in a comment why holding a
reference to every array in the list (instead of discarding each one) is what
prevents the garbage collector from reclaiming them.
