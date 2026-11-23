---
title: "Java Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free Java cheat sheet: 36 quick-reference tables from every level of the Java Mastery Path, each linked to its full lesson."
---

# Java Cheat Sheet

Quick-reference tables from **36 lessons** of the free Java Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [01 · Setup &amp; First Program](level-1/01-setup.md)

*Anatomy of the program*

| Piece | Meaning |
|-------|---------|
| `public class Hello` | Declares a class named `Hello` — everything in Java lives inside a class. |
| `public static void main(String[] args)` | The program's entry point. |
| `System.out.println(...)` | Prints text followed by a newline to standard output. |
| `;` | Every statement ends with a semicolon. |
| `{ }` | Curly braces delimit blocks — class bodies, method bodies, loop bodies. |

### [02 · Variables, Data Types &amp; Operators](level-1/02-variables-data-types.md)

*Primitive types*

| Type | Size | Example | Notes |
|------|------|---------|-------|
| `byte` | 8-bit | `byte b = 100;` | small integers |
| `short` | 16-bit | `short s = 30000;` | rarely used directly |
| `int` | 32-bit | `int x = 42;` | default integer type |
| `long` | 64-bit | `long big = 10_000_000_000L;` | note the `L` suffix |
| `float` | 32-bit | `float f = 3.14f;` | note the `f` suffix |
| `double` | 64-bit | `double d = 3.14159;` | default floating-point type |
| `char` | 16-bit | `char c = 'A';` | single UTF-16 character |
| `boolean` | 1 bit (JVM-dependent) | `boolean ok = true;` | `true`/`false` only |

### [03 · Control Flow](level-1/03-control-flow.md)

*break and continue*

| Construct | Use when |
|-----------|----------|
| `if / else` | branching on a boolean condition |
| `switch` | branching on one variable against many discrete values |
| `while` | repeat while a condition holds, unknown iteration count |
| `do-while` | same, but body always runs at least once |
| `for` | repeat a known number of times, or with an index |
| for-each | iterate every element of an array/collection |

### [04 · Methods &amp; Parameters](level-1/04-methods-parameters.md)

*static vs. instance methods*

| Kind | Called via | Can access instance fields? |
|------|-----------|------------------------------|
| `static` method | `ClassName.method()` | No |
| instance method | `object.method()` | Yes |

### [05 · Arrays &amp; Basic Collections](level-1/05-arrays-collections.md)

*List of numbers (boxed)*

| Feature | Array | ArrayList |
|---------|-------|-----------|
| Size | Fixed at creation | Grows/shrinks dynamically |
| Element type | Primitives or objects | Objects only (auto-boxing for primitives) |
| Syntax | `arr[i]` | `list.get(i)` |
| Declared as | `int[] arr` | `List<Integer> list` |

### [06 · Strings &amp; String Formatting](level-1/06-strings-formatting.md)

*Formatting output*

| Specifier | Meaning |
|-----------|---------|
| `%s` | String |
| `%d` | Integer |
| `%f` | Floating-point (`%.2f` = 2 decimal places) |
| `%n` | Platform-independent newline |
| `%-10s` | Left-justify in a 10-character field |
| `%5d` | Right-justify integer in a 5-character field |

### [07 · Classes &amp; Objects Basics](level-1/07-classes-objects.md)

*Instance methods operate on "this" object's data*

| Concept | Meaning |
|---------|---------|
| Class | Blueprint / type definition |
| Object (instance) | A concrete value created with `new` |
| Field | A variable that lives on each instance |
| Constructor | Special method that initializes a new instance |
| `this` | Reference to the current instance |
| `private` | Restricts access to within the class (encapsulation) |

### [08 · Exception Handling Basics](level-1/08-exception-handling.md)

*Checked vs. unchecked exceptions*

| Kind | Extends | Must be declared/caught? | Examples |
|------|---------|---------------------------|----------|
| Checked | `Exception` | Yes | `IOException`, `SQLException` |
| Unchecked | `RuntimeException` | No | `NullPointerException`, `ArithmeticException` |
| Error | `Error` | No (don't catch these) | `OutOfMemoryError`, `StackOverflowError` |

### [09 · Packages &amp; Build Tools Intro](level-1/09-packages-build-tools.md)

*Adding a dependency*

| Tool | Config file | Philosophy |
|------|-------------|------------|
| Maven | `pom.xml` (XML) | Convention over configuration, declarative |
| Gradle | `build.gradle` / `build.gradle.kts` | Flexible, script-based (Groovy or Kotlin DSL) |

## Level 2 · Intermediate

### [01 · OOP Deep Dive](level-2/01-oop-deep-dive.md)

*Abstract class vs. interface*

| Aspect | Abstract class | Interface |
|--------|-----------------|-----------|
| Instantiable? | No | No |
| Fields | Any (instance state allowed) | Only `static final` constants |
| Multiple inheritance | No (`extends` one class only) | Yes (`implements` many) |
| Method bodies | Concrete + abstract methods | `default`/`static` bodies + abstract signatures |
| Use when | Sharing state and code among closely related classes | Defining a capability unrelated classes can opt into |

### [02 · Collections Framework](level-2/02-collections-framework.md)

*The Collection hierarchy*

| Interface | Represents | Allows duplicates? | Ordered? |
|-----------|-----------|---------------------|----------|
| `List` | An ordered, indexable sequence | Yes | Yes (insertion order) |
| `Set` | A collection of unique elements | No | Depends on implementation |
| `Map` | Key → value pairs (not a `Collection`, but part of the framework) | Keys: no, Values: yes | Depends on implementation |

### [03 · Generics](level-2/03-generics.md)

*Wildcards: `?`, `? extends`, `? super`*

| Wildcard | Meaning | Typical use |
|----------|---------|-------------|
| `List<?>` | Unknown type | Read-only, type-agnostic operations (size, clear) |
| `List<? extends T>` | T or any subtype | Producer -- safe to read as T |
| `List<? super T>` | T or any supertype | Consumer -- safe to write T into it |

### [04 · Lambdas &amp; Streams API](level-2/04-lambdas-streams.md)

*Built-in functional interfaces (`java.util.function`)*

| Interface | Method | Signature | Typical use |
|-----------|--------|-----------|--------------|
| `Function<T,R>` | `apply(T)` | `T -> R` | Transform a value |
| `Predicate<T>` | `test(T)` | `T -> boolean` | A yes/no check, e.g. for `filter` |
| `Consumer<T>` | `accept(T)` | `T -> void` | Do something with a value, no result |
| `Supplier<T>` | `get()` | `() -> T` | Produce/lazily generate a value |
| `BiFunction<T,U,R>` | `apply(T,U)` | `(T,U) -> R` | Combine two values into a result |

### [06 · File I/O &amp; NIO](level-2/06-file-io-nio.md)

*Checking, creating, copying, deleting*

| Operation | Method |
|-----------|--------|
| Check existence | `Files.exists(path)` / `Files.notExists(path)` |
| Create directory (and parents) | `Files.createDirectories(path)` |
| Copy | `Files.copy(source, target, options...)` |
| Move / rename | `Files.move(source, target, options...)` |
| Delete | `Files.delete(path)` / `Files.deleteIfExists(path)` |
| Read all lines | `Files.readAllLines(path)` |
| Read whole file | `Files.readString(path)` |
| Write lines/string | `Files.write(path, lines)` / `Files.writeString(path, text)` |

### [07 · Unit Testing with JUnit 5](level-2/07-junit-testing.md)

*`@Test` and basic assertions*

| Assertion | Checks |
|-----------|--------|
| `assertEquals(expected, actual)` | Values are equal |
| `assertTrue(condition)` / `assertFalse(condition)` | Boolean condition |
| `assertThrows(Type.class, executable)` | Code throws the given exception type |
| `assertNull(obj)` / `assertNotNull(obj)` | Reference is/isn't null |
| `assertArrayEquals(expected, actual)` | Arrays have equal contents |

### [08 · Build Tools Deep Dive](level-2/08-build-tools.md)

*The full Maven lifecycle*

| Phase | What happens |
|-------|--------------|
| `validate` | Checks the project structure and `pom.xml` are correct |
| `compile` | Compiles `src/main/java` into `target/classes` |
| `test` | Compiles and runs `src/test/java` against a test framework (e.g. JUnit) |
| `package` | Bundles compiled code into a distributable `.jar` (or `.war`) |
| `verify` | Runs additional checks (e.g. integration tests) on the packaged artifact |
| `install` | Copies the artifact into your local `~/.m2` repository, for other local projects to depend on |
| `deploy` | Publishes the artifact to a shared/remote repository for other people's projects |

### [09 · Enums, Records &amp; Sealed Classes](level-2/09-enums-records-sealed.md)

*Cheat sheet*

| Feature | Purpose | Key trait |
|---------|---------|-----------|
| `enum` | Fixed set of named constants | Can hold fields, methods, per-constant bodies |
| `record` | Immutable data carrier | Auto-generates constructor, accessors, `equals`/`hashCode`/`toString` |
| `sealed` | Restrict which types may extend/implement | Enables exhaustive `switch` with no `default` |

## Level 3 · Advanced

### [01 · Concurrency Basics](level-3/01-concurrency-basics.md)

*Thread-safety basics*

| Term | Meaning |
|------|---------|
| Thread-safe | Behaves correctly when accessed by multiple threads concurrently |
| Race condition | Outcome depends on unpredictable thread interleaving |
| Critical section | Code that must not run concurrently on shared state |
| Intrinsic lock (monitor) | Per-object lock acquired/released by `synchronized` |
| `volatile` | Guarantees visibility of a variable's latest value across threads |
| Atomicity | An operation that appears to happen as a single, indivisible step |

### [02 · java.util.concurrent](level-3/02-util-concurrent.md)

*A quick look ahead: virtual threads*

| Type | Purpose |
|------|---------|
| `ExecutorService` | Manages a pool of worker threads |
| `Runnable` | Task with no return value |
| `Callable<T>` | Task that returns a value (or throws a checked exception) |
| `Future<T>` | Handle to a result that may not be ready yet |
| `CompletableFuture<T>` | Composable, non-blocking asynchronous pipeline |
| Virtual thread | Lightweight, JVM-managed thread (Java 21+) |

### [03 · JDBC &amp; Databases](level-3/03-jdbc-databases.md)

*Reading results with `ResultSet`*

| JDBC type | Purpose |
|-----------|---------|
| `DriverManager.getConnection(url, user, pass)` | Opens a connection to the database |
| `Statement` | Runs a fixed SQL string (no parameters) |
| `PreparedStatement` | Runs parameterized SQL safely, reusably |
| `ResultSet` | Cursor over the rows returned by a query |
| `executeUpdate()` | Runs INSERT/UPDATE/DELETE, returns rows affected |
| `executeQuery()` | Runs SELECT, returns a `ResultSet` |

### [04 · Building REST APIs with Spring Boot](level-3/04-rest-apis-spring-boot.md)

*Key annotations at a glance*

| Annotation | Purpose |
|------------|---------|
| `@SpringBootApplication` | Marks the main class; enables auto-configuration |
| `@RestController` | Marks a class whose methods return JSON/text responses |
| `@Service` | Marks a class as a business-logic bean |
| `@GetMapping` / `@PostMapping` | Maps an HTTP GET/POST to a method |
| `@PathVariable` | Binds a `{placeholder}` from the URL to a method parameter |
| `@RequestBody` | Deserializes the JSON request body into a Java object |
| `@Autowired` | Requests dependency injection (optional on a single constructor) |

### [05 · Design Patterns in Java](level-3/05-design-patterns.md)

*Strategy — swappable algorithms*

| Pattern | Problem it solves | Key idea |
|---------|--------------------|----------|
| Factory | Which concrete class to instantiate | Centralize creation behind one method/interface |
| Singleton | Ensure exactly one shared instance | Private constructor + controlled global access point |
| Observer | Notify many interested parties of an event | Subject holds a list of listener interfaces |
| Strategy | Swappable algorithms without conditionals | Extract behavior behind an interface, inject the implementation |

### [06 · Serialization](level-3/06-serialization.md)

*JSON with Jackson*

| Approach | Format | Best for |
|----------|--------|----------|
| `Serializable` + `ObjectOutputStream` | Binary, Java-only | Quick same-JVM persistence, caching |
| Jackson `ObjectMapper` | JSON, text, cross-language | APIs, config files, inter-service data |
| `@JsonProperty` | — | Renaming a Java field's JSON key |

### [07 · Networking Basics](level-3/07-networking-basics.md)

*Asynchronous requests*

| Type | Purpose |
|------|---------|
| `ServerSocket` | Listens for and accepts incoming TCP connections |
| `Socket` | One end of a TCP connection; provides input/output streams |
| `HttpClient` | Modern client for making HTTP requests (Java 11+) |
| `HttpRequest` | Describes a single HTTP request (URI, method, headers, body) |
| `HttpResponse<T>` | The response, with status code, headers, and a typed body |
| `BodyHandlers.ofString()` | Reads the response body as a `String` |

### [08 · Reflection &amp; Annotations](level-3/08-reflection-annotations.md)

*Reading a custom annotation at runtime*

| Concept | Purpose |
|---------|---------|
| `Class<?>` | Runtime representation of a type |
| `getDeclaredFields()` / `getDeclaredMethods()` | List all members, including private ones |
| `setAccessible(true)` | Bypass access checks (private/protected) |
| `Method.invoke(obj, args...)` | Call a method reflectively |
| `@interface` | Declares a custom annotation type |
| `@Retention(RetentionPolicy.RUNTIME)` | Keeps the annotation visible to reflection |
| `@Target` | Restricts where the annotation may be applied |

### [09 · JVM Internals &amp; Garbage Collection Basics](level-3/09-jvm-internals-gc.md)

*JVM architecture overview*

| Component | Role |
|-----------|------|
| Class loader | Loads `.class` files into memory, links and initializes classes |
| Heap | Stores all objects and arrays; shared across all threads |
| Stack | Per-thread; stores method call frames, local variables, partial results |
| Metaspace | Stores class metadata (replaces the old "permanent generation") |
| Execution engine | Interprets bytecode, or compiles it to native code via the JIT compiler |
| Garbage collector | Reclaims heap memory occupied by objects no longer reachable |

### [10 · Performance Profiling &amp; Optimization](level-3/10-profiling-optimization.md)

*Built-in diagnostic tools*

| Tool | Purpose |
|------|---------|
| `jcmd` | General-purpose diagnostic command tool (GC, thread dumps, flags, JFR control) |
| `jstack` (or `jcmd <pid> Thread.print`) | Dumps all thread stack traces — useful for diagnosing deadlocks/hangs |
| `jmap` (or `jcmd <pid> GC.heap_info` / `GC.class_histogram`) | Heap usage summaries, class instance counts |
| VisualVM | Free GUI tool bundled separately; live CPU/memory graphs, profiler, heap dumps |
| JFR (Java Flight Recorder) | Low-overhead, production-safe event recorder built into the JVM |

## Level 4 · Master

### [01 · Spring Ecosystem in Production](level-4/01-spring-ecosystem-production.md)

*Method-level authorization with `@PreAuthorize`*

| Concept | Purpose |
|---------|---------|
| `JpaRepository<T, ID>` | Base interface giving CRUD + paging for free |
| Derived query method | Query generated from the method name |
| `@Query` | Explicit JPQL/native SQL for complex queries |
| `Pageable` / `Page<T>` | Request and receive one page of a large result set |
| `SecurityFilterChain` | Declares which requests need which authorization |
| `BCryptPasswordEncoder` | One-way salted password hashing |
| `@PreAuthorize` | Method-level, expression-based access control |

### [02 · Microservices Architecture Patterns](level-4/02-microservices-architecture.md)

*Database-per-service*

| Pattern | Solves | Trade-off |
|---------|--------|-----------|
| API Gateway | Single entry point, cross-cutting concerns | Gateway becomes a critical path — must scale it too |
| Service discovery | Finding live instances dynamically | Extra moving part (the registry) to keep healthy |
| Circuit breaker | Stopping cascading failures | Fallbacks must degrade gracefully, not just error |
| Saga | Transactions across services | Eventual consistency, requires idempotent, compensable steps |
| Database-per-service | Independent deployability | No cross-service joins; more data duplication/events |

### [03 · Advanced Concurrency Patterns](level-4/03-advanced-concurrency-patterns.md)

*When virtual threads help vs. when they don't*

| Workload | Recommendation |
|----------|-----------------|
| High-concurrency I/O-bound (HTTP calls, DB queries, file I/O) | Virtual threads shine — cheap to block, scale to huge counts |
| CPU-bound (image processing, heavy computation) | Platform threads on a fixed-size pool sized to core count still apply — more virtual threads doesn't create more CPU |
| Long-lived pooled worker threads holding native resources | Keep platform threads — virtual threads are meant to be short-lived, one-per-task |
| Code using `synchronized` blocks around blocking I/O | Can "pin" the virtual thread to its carrier — prefer `ReentrantLock` in hot paths |

### [04 · Testing at Scale](level-4/04-testing-at-scale.md)

*Testcontainers — a real database in tests*

| Test type | Speed | What it verifies | Tool |
|-----------|-------|-------------------|------|
| Unit test with mocks | Milliseconds | One class's logic in isolation | Mockito |
| Spring integration test | ~1 second | Controller routing, validation, wiring | `@SpringBootTest` + `MockMvc` |
| Testcontainers test | Seconds (container startup) | Real SQL against a real database engine | Testcontainers |

### [05 · Security Best Practices](level-4/05-security-best-practices.md)

*Common vulnerabilities and mitigations*

| Vulnerability | What it is | Mitigation |
|----------------|------------|------------|
| SQL Injection | Untrusted input executed as SQL | Always use `PreparedStatement` / parameterized JPQL, never string-concatenate queries |
| XSS (Cross-Site Scripting) | Untrusted input rendered as HTML/JS in a browser | Escape output on render; rely on template engines' auto-escaping; set a `Content-Security-Policy` header |
| CSRF (Cross-Site Request Forgery) | A tricked browser submits a request using the victim's session | Use Spring Security's CSRF tokens for cookie-based sessions; not needed for stateless token-based APIs |
| Insecure Deserialization | Deserializing untrusted data executes attacker-controlled code | Avoid Java native serialization for untrusted input; validate/allow-list types when deserializing JSON into polymorphic types |

### [06 · Messaging &amp; Event-Driven Systems](level-4/06-messaging-event-driven.md)

*RabbitMQ comparison*

| | Kafka | RabbitMQ |
|---|---|---|
| Model | Distributed log — consumers track their own offset | Traditional broker — pushes to queues, tracks delivery |
| Message retention | Retained for a configured period regardless of consumption | Typically removed once acknowledged/consumed |
| Replay | Easy — reset a consumer group's offset and re-read history | Not designed for replay |
| Throughput | Very high, built for streaming/event-log workloads | High, optimized for flexible routing and lower-latency queuing |
| Routing flexibility | Simple (topic + partition key) | Rich (exchange types, routing keys, bindings) |
| Typical use | Event streaming, analytics pipelines, event sourcing | Task queues, RPC-style messaging, complex routing |

### [07 · Containerization &amp; Deployment](level-4/07-containerization-deployment.md)

*CI/CD with GitHub Actions*

| Step | Purpose |
|------|---------|
| Multi-stage `Dockerfile` | Separates build tooling from the slim runtime image |
| `.dockerignore` | Keeps the build context small and avoids leaking local files |
| `docker-compose.yml` | Runs the app and its dependencies together for local dev/testing |
| CI `test` job | Fails fast before any image is built |
| CI `build-image` job | Produces a versioned, deployable artifact tied to a commit SHA |

### [08 · Observability](level-4/08-observability.md)

*Structured logging with SLF4J and Logback*

| Level | Use for |
|-------|---------|
| `ERROR` | Something failed and needs attention |
| `WARN` | Unexpected but handled (e.g. a declined payment) |
| `INFO` | Notable business events (order created, user registered) |
| `DEBUG` | Detailed flow useful only when actively diagnosing an issue |

### [09 · Performance at Scale](level-4/09-performance-at-scale.md)

*JVM tuning flags*

| Flag | Meaning |
|------|---------|
| `-Xms` / `-Xmx` | Initial / maximum heap size — set equal in production to avoid resize pauses |
| `-XX:+UseG1GC` | Use the G1 garbage collector — balances throughput and pause time for most services |
| `-XX:+UseZGC` | Use ZGC — sub-millisecond pauses, favored for very large heaps or strict latency SLAs |
| `-Xss` | Stack size per thread — smaller stacks let you run more platform threads in the same memory |
| `-XX:+HeapDumpOnOutOfMemoryError` | Writes a heap dump automatically on OOM, for later analysis |

---

Go deeper with the full [Java Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
