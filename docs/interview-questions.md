---
title: "Java Interview Questions and Answers (183 Questions)"
description: "183 Java interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Java Mastery Path."
---

# Java Interview Questions and Answers

**183 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Java Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · Setup &amp; First Program](level-1/01-setup.md)

**Q1.** What do JDK, JRE and JVM mean?

**Answer:** JDK has the tools to build; JRE runs apps; the JVM executes bytecode

Install a JDK to develop.

**Q2.** What does `javac Main.java` produce?

**Answer:** Main.class bytecode run by the JVM

Write once, run anywhere.

**Q3.** What is the program entry point?

**Answer:** public static void main(String[] args)

In a class.

**Q4.** What does the file name have to match?

**Answer:** The public class name

Main.java for class Main.

### [02 · Variables, Data Types &amp; Operators](level-1/02-variables-data-types.md)

**Q5.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
int a = 7, b = 2;
System.out.println(a / b + " " + (double) a / b + " " + a % b);
}
}
```

**Answer:** 3 3.5 1

Integer division truncates. Verified by compiling and running it.

**Q6.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
Integer x = 127, y = 127, p = 1000, q = 1000;
System.out.println((x == y) + " " + (p == q) + " " + p.equals(q));
}
}
```

**Answer:** true false true

Integer caching: == compares references outside -128..127. Verified by compiling and running it.

**Q7.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
byte b = (byte) 200;
long big = Integer.MAX_VALUE + 1L;
System.out.println(b + " " + big);
}
}
```

**Answer:** -56 2147483648

Narrowing wraps; use L for long arithmetic. Verified by compiling and running it.

**Q8.** What does `var` do?

**Answer:** Infers the local variable's type at compile time (still statically typed)

Java 10+.

### [03 · Control Flow](level-1/03-control-flow.md)

**Q9.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
int s = 0;
for (int i = 1; i <= 10; i++) { if (i % 2 == 0) continue; if (i > 7) break; s += i; }
System.out.println(s);
}
}
```

**Answer:** 16

1+3+5+7. Verified by compiling and running it.

**Q10.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
int n = 2;
String r = switch (n) { case 1 -> "one"; case 2, 3 -> "few"; default -> "many"; };
System.out.println(r);
}
}
```

**Answer:** few

Switch expressions (Java 14+). Verified by compiling and running it.

**Q11.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
int n = 1;
switch (n) { case 1: System.out.print("a"); case 2: System.out.print("b"); break; case 3: System.out.print("c"); }
}
}
```

**Answer:** ab

Old-style switch falls through without break. Verified by compiling and running it.

**Q12.** Which loop runs at least once?

**Answer:** do { } while (cond);

Condition after the body.

### [04 · Methods &amp; Parameters](level-1/04-methods-parameters.md)

**Q13.** What does this print?

```java
public class Main {
static void change(int[] a) { a[0] = 99; }
static void bump(int n) { n++; }
public static void main(String[] args) throws Exception {
int[] a = {1, 2, 3};
change(a);
int x = 1;
bump(x);
System.out.println(a[0] + " " + x);
}
}
```

**Answer:** 99 1

Java passes references by value: arrays change, primitives don't. Verified by compiling and running it.

**Q14.** What does this print?

```java
public class Main {
static int sum(int... n) { int t = 0; for (int v : n) t += v; return t; }
public static void main(String[] args) throws Exception {
System.out.println(sum(1, 2, 3) + " " + sum());
}
}
```

**Answer:** 6 0

Varargs. Verified by compiling and running it.

**Q15.** What does this print?

```java
public class Main {
static String f(int x) { return "int"; }
static String f(long x) { return "long"; }
static String f(double x) { return "double"; }
public static void main(String[] args) throws Exception {
System.out.println(f(1) + " " + f(1L) + " " + f(1.0));
}
}
```

**Answer:** int long double

Overload resolution picks the best match. Verified by compiling and running it.

**Q16.** What's the difference between a method signature and its return type?

**Answer:** The signature is name plus parameter types; return type isn't part of overload identity

Can't overload by return type.

### [05 · Arrays &amp; Basic Collections](level-1/05-arrays-collections.md)

**Q17.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
int[] a = {5, 3, 8};
Arrays.sort(a);
List<Integer> l = new ArrayList<>(List.of(3, 1, 2));
Collections.sort(l);
l.add(0, 9);
System.out.println(Arrays.toString(a) + " " + l + " " + l.indexOf(2));
}
}
```

**Answer:** [3, 5, 8] [9, 1, 2, 3] 2

ArrayList supports insertion by index. Verified by compiling and running it.

**Q18.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
Map<String,Integer> m = new HashMap<>();
m.merge("a", 1, Integer::sum); m.merge("a", 1, Integer::sum); m.putIfAbsent("b", 7);
System.out.println(new TreeMap<>(m));
}
}
```

**Answer:** {a=2, b=7}

merge combines with an existing value. Verified by compiling and running it.

**Q19.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
List<Integer> l = new ArrayList<>(List.of(1, 2, 3, 4));
l.removeIf(x -> x % 2 == 0);
try { for (Integer x : l) if (x == 1) l.add(9); } catch (ConcurrentModificationException e) { System.out.print("CME "); }
System.out.println(l);
}
}
```

**Answer:** CME [1, 3, 9]

Structural modification during iteration throws. Verified by compiling and running it.

**Q20.** When use a HashSet?

**Answer:** For uniqueness and fast membership tests

No guaranteed order.

### [06 · Strings &amp; String Formatting](level-1/06-strings-formatting.md)

**Q21.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
String a = "hi", b = "hi", c = new String("hi");
System.out.println((a == b) + " " + (a == c) + " " + a.equals(c) + " " + a.equals(c.intern()));
}
}
```

**Answer:** true false true true

String literals are interned; == compares references. Verified by compiling and running it.

**Q22.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
System.out.println(String.format("%5.2f|%-4s|%04d|%,d", 3.14159, "ab", 42, 1234567));
}
}
```

**Answer:** 3.14|ab  |0042|1,234,567

printf-style formatting. Verified by compiling and running it.

**Q23.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
String s = "a,b,,c";
System.out.println(Arrays.toString(s.split(",")) + " " + "  x ".strip().length() + " " + String.join("-", "a", "b"));
}
}
```

**Answer:** [a, b, , c] 1 a-b

split keeps interior empty strings. Verified by compiling and running it.

**Q24.** Why use StringBuilder in loops?

**Answer:** Strings are immutable; concatenation in a loop creates many objects

Build then toString().

### [07 · Classes &amp; Objects Basics](level-1/07-classes-objects.md)

**Q25.** What does this print?

```java
public class Main {
static class Counter { static int created = 0; private int n; Counter() { created++; } void inc() { n++; } int get() { return n; } }
public static void main(String[] args) throws Exception {
Counter c = new Counter(); c.inc(); c.inc();
System.out.println(c.get() + " " + Counter.created);
}
}
```

**Answer:** 2 1

static fields belong to the class. Verified by compiling and running it.

**Q26.** What does this print?

```java
public class Main {
static class Point { final int x, y; Point(int x, int y) { this.x = x; this.y = y; }
 @Override public boolean equals(Object o) { return o instanceof Point p && p.x == x && p.y == y; }
 @Override public int hashCode() { return java.util.Objects.hash(x, y); } }
public static void main(String[] args) throws Exception {
Point a = new Point(1, 2), b = new Point(1, 2);
System.out.println(a.equals(b) + " " + (a == b) + " " + (a.hashCode() == b.hashCode()));
}
}
```

**Answer:** true false true

equals/hashCode must be overridden consistently. Verified by compiling and running it.

**Q27.** What does `private` do?

**Answer:** Restricts access to the declaring class

Encapsulation.

**Q28.** What does `this` refer to?

**Answer:** The current object instance

Constructor chaining too.

### [08 · Exception Handling Basics](level-1/08-exception-handling.md)

**Q29.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
try { System.out.print("A"); Integer.parseInt("x"); System.out.print("B"); }
catch (NumberFormatException e) { System.out.print("C"); }
finally { System.out.print("D"); }
}
}
```

**Answer:** ACD

finally always runs. Verified by compiling and running it.

**Q30.** What does this print?

```java
public class Main {
static void r() { try { int x = 1 / 0; } catch (ArithmeticException e) { throw new IllegalStateException("wrapped", e); } }
public static void main(String[] args) throws Exception {
try { r(); } catch (IllegalStateException e) { System.out.println(e.getMessage() + " / " + e.getCause().getClass().getSimpleName()); }
}
}
```

**Answer:** wrapped / ArithmeticException

Exception chaining. Verified by compiling and running it.

**Q31.** What's the difference between checked and unchecked exceptions?

**Answer:** Checked must be declared or caught; unchecked (RuntimeException) need not be

IOException vs NullPointerException.

**Q32.** What does try-with-resources do?

**Answer:** Closes AutoCloseable resources automatically

Even on exceptions.

### [09 · Packages &amp; Build Tools Intro](level-1/09-packages-build-tools.md)

**Q33.** What does a package declaration do?

**Answer:** Namespaces classes and maps to directories

Reverse-domain names.

**Q34.** What do Maven/Gradle provide?

**Answer:** Dependency management and build lifecycle

pom.xml/build.gradle.

**Q35.** What's in a Maven `pom.xml`?

**Answer:** Coordinates, dependencies, plugins and properties

Project model.

**Q36.** What does `mvn test` run?

**Answer:** The test phase (compile and unit tests)

Lifecycle.

### [10 · Project — CLI Contact Book](level-1/10-project-contact-book.md)

**Q37.** What does the contact book project combine?

**Answer:** Classes, collections, exceptions, file I/O and build tooling

Level 1 project.

**Q38.** How persist contacts?

**Answer:** Write to a file (CSV/JSON) and load on startup

Files.

**Q39.** How validate input?

**Answer:** Check and report errors rather than crash

User-friendly.

**Q40.** How test it?

**Answer:** Unit tests for the model/service layer

JUnit.

## Level 2 · Intermediate

### [01 · OOP Deep Dive](level-2/01-oop-deep-dive.md)

**Q41.** What does this print?

```java
public class Main {
static class Animal { String speak() { return "Generic"; } static String kind() { return "animal"; } }
static class Dog extends Animal { @Override String speak() { return "Woof"; } }
public static void main(String[] args) throws Exception {
Animal a = new Dog();
System.out.println(a.speak() + " " + (a instanceof Dog) + " " + Animal.kind());
}
}
```

**Answer:** Woof true animal

Polymorphism picks the overriding method; static methods belong to the class. Verified by compiling and running it.

**Q42.** What does an abstract class allow?

**Answer:** Shared state/behavior with abstract methods subclasses must implement

Template pattern.

**Q43.** What's the difference between interface and abstract class?

**Answer:** Interfaces define contracts (multiple allowed); abstract classes can hold state and constructors

Default methods blur the line.

**Q44.** What does `super` do?

**Answer:** Refers to the parent class's members/constructor

Constructor chaining.

### [02 · Collections Framework](level-2/02-collections-framework.md)

**Q45.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
Deque<Integer> d = new ArrayDeque<>(); d.push(1); d.push(2); d.addLast(3);
Queue<String> q = new PriorityQueue<>(List.of("pear", "fig", "apple"));
System.out.println(d + " " + q.poll() + " " + new TreeSet<>(List.of(3, 1, 2)).first());
}
}
```

**Answer:** [2, 1, 3] apple 1

Stack push adds to the front; priority queue orders naturally. Verified by compiling and running it.

**Q46.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
Map<String,Integer> m = new LinkedHashMap<>();
m.put("b", 2); m.put("a", 1); m.put("b", 3);
System.out.println(m + " " + m.getOrDefault("z", 0));
}
}
```

**Answer:** {b=3, a=1} 0

LinkedHashMap keeps insertion order. Verified by compiling and running it.

**Q47.** What's HashMap's average lookup?

**Answer:** O(1)

TreeMap is O(log n).

**Q48.** What must keys implement?

**Answer:** Consistent equals() and hashCode()

Else lookups fail.

### [03 · Generics](level-2/03-generics.md)

**Q49.** What does this print?

```java
import java.util.*;
public class Main {
static <T extends Comparable<T>> T max(T a, T b) { return a.compareTo(b) >= 0 ? a : b; }
public static void main(String[] args) throws Exception {
List<String> a = new ArrayList<>(); a.add("x");
System.out.println(max(3, 7) + " " + max("a", "b") + " " + a);
}
}
```

**Answer:** 7 b [x]

Bounded generics. Verified by compiling and running it.

**Q50.** What is type erasure?

**Answer:** Generic type info is removed at runtime (List&lt;String&gt; becomes List)

No new T().

**Q51.** What does `? extends Number` mean?

**Answer:** An unknown subtype of Number (read-only use)

PECS: producer extends.

**Q52.** Why use generics?

**Answer:** Compile-time type safety and no casts

Collections.

### [04 · Lambdas &amp; Streams API](level-2/04-lambdas-streams.md)

**Q53.** What does this print?

```java
import java.util.*; import java.util.stream.*;
public class Main {

public static void main(String[] args) throws Exception {
List<String> w = List.of("apple", "bob", "cat", "avocado");
System.out.println(w.stream().filter(s -> s.startsWith("a")).map(String::toUpperCase).collect(Collectors.toList()));
Map<Character, Long> g = w.stream().collect(Collectors.groupingBy(s -> s.charAt(0), TreeMap::new, Collectors.counting()));
System.out.println(g);
System.out.println(IntStream.rangeClosed(1, 5).sum() + " " + w.stream().mapToInt(String::length).max().getAsInt());
}
}
```

**Answer:** [APPLE, AVOCADO]
{a=2, b=1, c=1}
15 7

Streams: filter, map, group, reduce. Verified by compiling and running it.

**Q54.** What does this print?

```java
import java.util.function.*;
public class Main {

public static void main(String[] args) throws Exception {
Function<Integer,Integer> inc = x -> x + 1;
Function<Integer,Integer> dbl = x -> x * 2;
Supplier<String> s = () -> "hi";
System.out.println(inc.andThen(dbl).apply(3) + " " + inc.compose(dbl).apply(3) + " " + s.get());
}
}
```

**Answer:** 8 7 hi

andThen applies inc first; compose applies dbl first. Verified by compiling and running it.

**Q55.** Are streams lazy?

**Answer:** Yes: intermediate ops run only when a terminal operation is called

Pipeline.

**Q56.** What can a lambda capture?

**Answer:** Effectively final local variables

Immutable captures.

### [05 · Exception Handling Advanced](level-2/05-exceptions-advanced.md)

**Q57.** What does this print?

```java
public class Main {
static class R implements AutoCloseable { public void close() { throw new IllegalArgumentException("close"); } }
static void f() { try (R r = new R()) { throw new IllegalStateException("body"); } }
public static void main(String[] args) throws Exception {
try { f(); } catch (Exception e) { System.out.println(e.getClass().getSimpleName() + ":" + e.getMessage() + ":" + e.getSuppressed().length); }
}
}
```

**Answer:** IllegalStateException:body:1

Close exceptions become suppressed on the primary one. Verified by compiling and running it.

**Q58.** What's a custom exception good for?

**Answer:** Expressing domain errors callers can catch specifically

Extend Exception or RuntimeException.

**Q59.** Why not catch Exception broadly?

**Answer:** It hides bugs and swallows unexpected failures

Be specific.

**Q60.** What does `throw` vs `throws` mean?

**Answer:** throw raises an exception; throws declares it in a signature

Checked exceptions.

### [06 · File I/O &amp; NIO](level-2/06-file-io-nio.md)

**Q61.** What's the modern API for files in Java?

**Answer:** java.nio.file.Files and Path

NIO.2.

**Q62.** What does this print?

```java
import java.nio.file.*; import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
Path p = Files.createTempFile("t", ".txt");
Files.writeString(p, "a\nb\nc");
List<String> lines = Files.readAllLines(p);
System.out.println(lines.size() + " " + lines.get(1) + " " + Files.size(p));
Files.delete(p);
}
}
```

**Answer:** 3 b 5

Files helpers read/write whole files. Verified by compiling and running it.

**Q63.** Why close streams?

**Answer:** Release file handles (use try-with-resources)

Resource leaks.

**Q64.** What encoding should you specify?

**Answer:** UTF-8 explicitly

Portability.

### [07 · Unit Testing with JUnit 5](level-2/07-junit-testing.md)

**Q65.** What does @Test mark in JUnit 5?

**Answer:** A test method

org.junit.jupiter.api.

**Q66.** What do @BeforeEach and @AfterEach do?

**Answer:** Run before/after every test

Fresh state.

**Q67.** What does assertThrows verify?

**Answer:** That code throws an expected exception type

Error handling.

**Q68.** What makes tests reliable?

**Answer:** Independent, deterministic and fast

Isolation.

### [08 · Build Tools Deep Dive](level-2/08-build-tools.md)

**Q69.** What do Gradle/Maven handle?

**Answer:** Dependencies, compile, test, package and run tasks

Reproducible builds.

**Q70.** What's a dependency scope 'test' for?

**Answer:** Libraries only needed for tests

JUnit.

**Q71.** What does a lock/wrapper provide?

**Answer:** Reproducible tool versions (mvnw/gradlew)

Consistency.

**Q72.** What does `mvn package` produce?

**Answer:** A jar/war in target/

Artifact.

### [09 · Enums, Records &amp; Sealed Classes](level-2/09-enums-records-sealed.md)

**Q73.** What does this print?

```java
public class Main {
sealed interface Shape permits Circle, Square {}
record Circle(double r) implements Shape {}
record Square(double s) implements Shape {}
record P(int x, int y) {}
enum Level { LOW, MID, HIGH }
public static void main(String[] args) throws Exception {
Shape s = new Circle(1);
String d = s instanceof Circle c ? "circle r=" + c.r() : "square";
System.out.println(d + " " + new P(1, 2) + " " + new P(1, 2).equals(new P(1, 2)) + " " + Level.valueOf("HIGH").ordinal());
}
}
```

**Answer:** circle r=1.0 P[x=1, y=2] true 2

Records, enums and sealed hierarchies. Verified by compiling and running it.

**Q74.** What do records give you?

**Answer:** Immutable data classes with equals/hashCode/toString/accessors

Java 16+.

**Q75.** What does `sealed` restrict?

**Answer:** Which classes can extend/implement a type (permits)

Exhaustive switches.

**Q76.** When use enums?

**Answer:** A fixed set of constants with optional behavior

Type safe.

### [10 · Project — Inventory Management System](level-2/10-project-inventory-system.md)

**Q77.** What does the inventory project combine?

**Answer:** OOP design, collections, streams, file I/O, tests and build

Level 2 project.

**Q78.** How model stock changes safely?

**Answer:** Validate (no negatives) and throw domain exceptions

Invariants.

**Q79.** How report totals?

**Answer:** Stream operations over collections

Concise.

**Q80.** What proves correctness?

**Answer:** Unit tests for services and edge cases

Quality.

## Level 3 · Advanced

### [01 · Concurrency Basics](level-3/01-concurrency-basics.md)

**Q81.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
int[] c = {0};
Object lock = new Object();
Thread[] ts = new Thread[4];
for (int i = 0; i < 4; i++) { ts[i] = new Thread(() -> { for (int j = 0; j < 1000; j++) synchronized (lock) { c[0]++; } }); ts[i].start(); }
for (Thread t : ts) t.join();
System.out.println(c[0]);
}
}
```

**Answer:** 4000

synchronized protects the shared counter. Verified by compiling and running it.

**Q82.** What's a race condition?

**Answer:** Result depends on unsynchronized timing of threads

Use locks/atomics.

**Q83.** What does `volatile` guarantee?

**Answer:** Visibility of writes across threads (not atomicity of compound ops)

i++ isn't atomic.

**Q84.** What's a deadlock?

**Answer:** Threads waiting on each other's locks forever

Lock ordering.

### [02 · java.util.concurrent](level-3/02-util-concurrent.md)

**Q85.** What does this print?

```java
import java.util.*; import java.util.concurrent.*; import java.util.concurrent.atomic.*;
public class Main {

public static void main(String[] args) throws Exception {
ExecutorService ex = Executors.newFixedThreadPool(3);
List<Future<Integer>> fs = new ArrayList<>();
for (int i = 1; i <= 5; i++) { final int n = i; fs.add(ex.submit(() -> n * n)); }
int s = 0; for (Future<Integer> f : fs) s += f.get();
ex.shutdown();
AtomicInteger a = new AtomicInteger(); a.incrementAndGet(); a.addAndGet(4);
System.out.println(s + " " + a.get() + " " + CompletableFuture.supplyAsync(() -> 21).thenApply(x -> x * 2).get());
}
}
```

**Answer:** 55 5 42

Executors, atomics and CompletableFuture. Verified by compiling and running it.

**Q86.** Why use thread pools?

**Answer:** Reuse threads and bound concurrency

Executors.

**Q87.** What does ConcurrentHashMap provide?

**Answer:** Thread-safe map with fine-grained concurrency

Better than synchronizedMap.

**Q88.** What are virtual threads (Java 21)?

**Answer:** Lightweight threads managed by the JVM for high-concurrency I/O

Project Loom.

### [03 · JDBC &amp; Databases](level-3/03-jdbc-databases.md)

**Q89.** What's JDBC?

**Answer:** The standard API for relational databases

Connection, Statement.

**Q90.** How prevent SQL injection?

**Answer:** PreparedStatement with parameters

Bind variables.

**Q91.** What does a connection pool do?

**Answer:** Reuses database connections (HikariCP)

Performance.

**Q92.** What does a transaction ensure?

**Answer:** Atomic all-or-nothing changes

commit/rollback.

### [04 · Building REST APIs with Spring Boot](level-3/04-rest-apis-spring-boot.md)

**Q93.** What's Spring Boot?

**Answer:** An opinionated framework for building Spring apps with auto-configuration

Starters.

**Q94.** What does @RestController do?

**Answer:** Marks a class whose methods handle HTTP and return response bodies

JSON.

**Q95.** What does @Autowired/constructor injection do?

**Answer:** Supplies dependencies from the Spring container

Prefer constructors.

**Q96.** What does `spring-boot-starter-web` bring?

**Answer:** Embedded Tomcat, Spring MVC and Jackson

Batteries.

### [05 · Design Patterns in Java](level-3/05-design-patterns.md)

**Q97.** What does this print?

```java
import java.util.*;
public class Main {
static class Config { private static final Config I = new Config(); private Config() {} static Config get() { return I; } }
interface Shape { int area(); }
static class ShapeFactory { static Shape of(String k) { return k.equals("sq") ? () -> 4 : () -> 1; } }
static class Notifier { List<java.util.function.Consumer<String>> subs = new ArrayList<>(); void subscribe(java.util.function.Consumer<String> c) { subs.add(c); } void publish(String m) { subs.forEach(c -> c.accept(m)); } }
public static void main(String[] args) throws Exception {
System.out.println(Config.get() == Config.get());
Shape s = ShapeFactory.of("sq");
System.out.println(s.area());
List<String> log = new ArrayList<>();
Notifier n = new Notifier(); n.subscribe(log::add); n.publish("x"); n.publish("y");
System.out.println(log);
}
}
```

**Answer:** true
4
[x, y]

Singleton, factory and observer patterns. Verified by compiling and running it.

**Q98.** What does the Strategy pattern do?

**Answer:** Swaps interchangeable algorithms behind an interface

Lambdas simplify.

**Q99.** What's wrong with singletons?

**Answer:** Global state hampers testing; use DI

Inject.

**Q100.** What does the Builder pattern solve?

**Answer:** Constructing complex objects step by step

Readable.

### [06 · Serialization](level-3/06-serialization.md)

**Q101.** What does this print?

```java
import java.io.*;
public class Main {
static class U implements java.io.Serializable { String name; transient String pw; U(String n, String p) { name = n; pw = p; } }
public static void main(String[] args) throws Exception {
ByteArrayOutputStream bos = new ByteArrayOutputStream();
try (ObjectOutputStream o = new ObjectOutputStream(bos)) { o.writeObject(new U("ada", "secret")); }
U u = (U) new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray())).readObject();
System.out.println(u.name + " " + u.pw);
}
}
```

**Answer:** ada null

transient fields aren't serialized. Verified by compiling and running it.

**Q102.** Why avoid Java serialization for external data?

**Answer:** It's insecure and brittle; prefer JSON/Protobuf

Deserialization attacks.

**Q103.** What does serialVersionUID do?

**Answer:** Identifies the class version for compatibility checks

Declare it.

**Q104.** What marks a field as not serialized?

**Answer:** transient

Secrets.

### [07 · Networking Basics](level-3/07-networking-basics.md)

**Q105.** Which classes do TCP clients and servers use?

**Answer:** Socket and ServerSocket

Streams.

**Q106.** What does java.net.http.HttpClient provide?

**Answer:** A modern HTTP client (sync and async)

Java 11+.

**Q107.** Why set timeouts?

**Answer:** Avoid hanging threads on slow peers

Connect/read.

**Q108.** What must servers do per connection?

**Answer:** Handle concurrently (threads/pool) and close sockets

Resource safety.

### [08 · Reflection &amp; Annotations](level-3/08-reflection-annotations.md)

**Q109.** What does this print?

```java
public class Main {
@java.lang.annotation.Retention(java.lang.annotation.RetentionPolicy.RUNTIME) @interface Info { String value(); }
@Info("dog") public static class Pet { private final String name; public Pet(String n) { name = n; } public String name() { return name; } }
public static void main(String[] args) throws Exception {
Class<?> c = Pet.class;
System.out.println(c.getSimpleName() + " " + c.getDeclaredFields().length + " " + c.isAnnotationPresent(Info.class));
Object o = c.getDeclaredConstructor(String.class).newInstance("Rex");
System.out.println(c.getMethod("name").invoke(o) + " " + Pet.class.getAnnotation(Info.class).value());
}
}
```

**Answer:** Pet 1 true
Rex dog

Reflection reads annotations and invokes methods. Verified by compiling and running it.

**Q110.** What's reflection used for?

**Answer:** Frameworks (DI, serialization, ORMs) inspecting types at runtime

Powerful but slow/unsafe.

**Q111.** What does @Retention(RUNTIME) mean?

**Answer:** The annotation is available via reflection at runtime

Needed by frameworks.

**Q112.** What's a risk of reflection?

**Answer:** Breaks encapsulation and loses compile-time safety

Use sparingly.

### [09 · JVM Internals &amp; Garbage Collection Basics](level-3/09-jvm-internals-gc.md)

**Q113.** What does the JVM heap hold?

**Answer:** Objects (managed by the garbage collector)

Young/old generations.

**Q114.** What does GC reclaim?

**Answer:** Unreachable objects

Reachability from GC roots.

**Q115.** Which collector is the modern default?

**Answer:** G1 (and ZGC/Shenandoah for low pause)

Tune carefully.

**Q116.** What causes OutOfMemoryError?

**Answer:** Heap (or metaspace) exhausted, often by leaks

Heap dumps.

### [10 · Performance Profiling &amp; Optimization](level-3/10-profiling-optimization.md)

**Q117.** What's the first step in optimizing?

**Answer:** Measure with a profiler/JFR before changing code

Evidence.

**Q118.** What is JFR?

**Answer:** Java Flight Recorder, low-overhead profiling built into the JVM

Production-safe.

**Q119.** What causes allocation pressure?

**Answer:** Creating many short-lived objects in hot paths

Reduce churn.

**Q120.** What are JMH benchmarks for?

**Answer:** Reliable microbenchmarks accounting for JIT warmup

Avoid naive timing.

### [11 · Project — REST API + Database CRUD Service](level-3/11-project-rest-api-db.md)

**Q121.** What does the REST API + DB project combine?

**Answer:** Spring Boot controllers, JDBC/JPA persistence, validation and tests

Level 3 project.

**Q122.** How validate request bodies?

**Answer:** Bean Validation annotations (@Valid, @NotBlank)

400 on failure.

**Q123.** How handle errors consistently?

**Answer:** @ControllerAdvice mapping exceptions to responses

Uniform format.

**Q124.** How test the web layer?

**Answer:** MockMvc or @SpringBootTest with an in-memory DB

Isolation.

## Level 4 · Master

### [01 · Spring Ecosystem in Production](level-4/01-spring-ecosystem-production.md)

**Q125.** What do Spring Boot Actuator endpoints provide?

**Answer:** Health, metrics and operational insight

Production readiness.

**Q126.** What are profiles for?

**Answer:** Environment-specific configuration (dev/prod)

application-prod.yml.

**Q127.** How externalize config?

**Answer:** Environment variables, config server or secrets manager

12-factor.

**Q128.** What does graceful shutdown do?

**Answer:** Finishes in-flight requests before stopping

server.shutdown=graceful.

### [02 · Microservices Architecture Patterns](level-4/02-microservices-architecture.md)

**Q129.** What does a microservice own?

**Answer:** Its own data and API (bounded context)

Autonomy.

**Q130.** What's an API gateway for?

**Answer:** Single entry: routing, auth and rate limiting

Cross-cutting.

**Q131.** What does a circuit breaker do?

**Answer:** Stops calling a failing dependency to let it recover (Resilience4j)

Fail fast.

**Q132.** What's the distributed monolith anti-pattern?

**Answer:** Services tightly coupled so they must deploy together

Avoid.

### [03 · Advanced Concurrency Patterns](level-4/03-advanced-concurrency-patterns.md)

**Q133.** What's structured concurrency's idea?

**Answer:** Scoped lifetimes for concurrent tasks so errors and cancellation propagate

StructuredTaskScope.

**Q134.** What's a bulkhead?

**Answer:** Isolating resources so one failing part doesn't sink others

Resilience.

**Q135.** Which structure handles backpressure?

**Answer:** Bounded queues with rejection or blocking policies

Protect memory.

**Q136.** What does ForkJoinPool suit?

**Answer:** Divide-and-conquer CPU-bound tasks

Work stealing.

### [04 · Testing at Scale](level-4/04-testing-at-scale.md)

**Q137.** What does Testcontainers provide?

**Answer:** Real dependencies (DB/Kafka) in Docker for integration tests

Realistic.

**Q138.** What's the test pyramid?

**Answer:** Many unit tests, fewer integration, very few end-to-end

Speed.

**Q139.** What do contract tests verify?

**Answer:** Consumer/provider API agreements

Pact.

**Q140.** How limit flakiness?

**Answer:** Deterministic data, no sleeps and isolated state

Quality.

### [05 · Security Best Practices](level-4/05-security-best-practices.md)

**Q141.** How hash passwords?

**Answer:** BCrypt/Argon2 via Spring Security's PasswordEncoder

Salted slow.

**Q142.** What does CSRF protection matter for?

**Answer:** Browser sessions/cookies (less for token APIs)

Know context.

**Q143.** What does the OWASP Dependency-Check find?

**Answer:** Known vulnerable libraries

Supply chain.

**Q144.** How avoid injection?

**Answer:** Parameterized queries and input validation

Defense.

### [06 · Messaging &amp; Event-Driven Systems](level-4/06-messaging-event-driven.md)

**Q145.** When use messaging (Kafka/RabbitMQ)?

**Answer:** To decouple services and handle async workloads

Event-driven.

**Q146.** What must consumers be?

**Answer:** Idempotent

At-least-once.

**Q147.** What's the outbox pattern?

**Answer:** Writing events to a table in the same transaction as state, then publishing

Consistency.

**Q148.** What's a dead-letter queue?

**Answer:** Where failing messages go after retries

Inspect.

### [07 · Containerization &amp; Deployment](level-4/07-containerization-deployment.md)

**Q149.** Why multi-stage Docker builds for Java?

**Answer:** Build with the JDK, run on a slim JRE image

Smaller images.

**Q150.** What JVM flag helps containers?

**Answer:** -XX:MaxRAMPercentage (container-aware memory)

Memory limits.

**Q151.** What does jlink/CRaC offer?

**Answer:** Smaller runtimes / faster startup

Optimization.

**Q152.** What's a readiness probe?

**Answer:** Signals when the app can serve traffic

Kubernetes.

### [08 · Observability](level-4/08-observability.md)

**Q153.** What are the observability signals?

**Answer:** Logs, metrics and traces

Together.

**Q154.** What does Micrometer do?

**Answer:** A metrics facade exporting to Prometheus etc.

Spring Boot default.

**Q155.** What does a trace id do?

**Answer:** Correlates a request across services

Propagate headers.

**Q156.** What's structured logging?

**Answer:** JSON/key-value logs with context fields

Searchable.

### [09 · Performance at Scale](level-4/09-performance-at-scale.md)

**Q157.** What first when tuning at scale?

**Answer:** Profile production-like load and fix the bottleneck

Evidence.

**Q158.** What reduces GC pauses?

**Answer:** Lower allocation rates and a suitable collector (ZGC/G1)

Tune.

**Q159.** What's a common DB bottleneck?

**Answer:** N+1 queries and missing indexes

Fetch plans.

**Q160.** What does caching require?

**Answer:** A clear invalidation strategy and TTLs

Hard problem.

### [10 · Capstone Project](level-4/10-capstone-project.md)

**Q161.** What does the order management capstone layer its service as?

**Answer:** Controller → Service → Repository, with DTO validation and a global exception handler

Separation of concerns makes each layer testable.

**Q162.** Which tests does the capstone include?

**Answer:** Mockito unit tests for the service and a @SpringBootTest + MockMvc integration test

Fast unit tests plus a few integration tests.

**Q163.** What does @RestControllerAdvice provide?

**Answer:** A single place that maps exceptions to consistent HTTP error responses

Uniform error format.

**Q164.** What does the CI workflow do for the capstone?

**Answer:** Builds the project and runs the tests on every push/pull request

Fast feedback and a merge gate.

## Mixed questions across all levels

**Q165.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
int a = 7, b = 2;
System.out.println(a / b + " " + (double) a / b);
}
}
```

**Answer:** 3 3.5

Integer division truncates. Verified by compiling and running it.

**Q166.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
Integer x = 127, y = 127, p = 1000, q = 1000;
System.out.println((x == y) + " " + (p == q) + " " + p.equals(q));
}
}
```

**Answer:** true false true

Integer caching: == compares references outside -128..127. Verified by compiling and running it.

**Q167.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
String a = "hi", c = new String("hi");
System.out.println((a == c) + " " + a.equals(c));
}
}
```

**Answer:** false true

Use equals for content. Verified by compiling and running it.

**Q168.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
try { System.out.print("A"); Integer.parseInt("x"); System.out.print("B"); } catch (NumberFormatException e) { System.out.print("C"); } finally { System.out.print("D"); }
}
}
```

**Answer:** ACD

finally always runs. Verified by compiling and running it.

**Q169.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
List<Integer> l = new ArrayList<>(List.of(1, 2, 3, 4));
l.removeIf(x -> x % 2 == 0);
System.out.println(l);
}
}
```

**Answer:** [1, 3]

removeIf filters in place. Verified by compiling and running it.

**Q170.** What does this print?

```java
import java.util.*;
public class Main {

public static void main(String[] args) throws Exception {
Map<String,Integer> m = new HashMap<>();
for (String w : "a b a c a b".split(" ")) m.merge(w, 1, Integer::sum);
System.out.println(new TreeMap<>(m));
}
}
```

**Answer:** {a=3, b=2, c=1}

merge counts occurrences. Verified by compiling and running it.

**Q171.** What does this print?

```java
public class Main {
static class Animal { String speak() { return "Generic"; } }
static class Dog extends Animal { @Override String speak() { return "Woof"; } }
public static void main(String[] args) throws Exception {
Animal a = new Dog();
System.out.println(a.speak());
}
}
```

**Answer:** Woof

Dynamic dispatch uses the runtime type. Verified by compiling and running it.

**Q172.** What does this print?

```java
public class Main {
record P(int x, int y) {}
public static void main(String[] args) throws Exception {
System.out.println(new P(1, 2) + " " + new P(1, 2).equals(new P(1, 2)));
}
}
```

**Answer:** P[x=1, y=2] true

Records generate toString, equals and hashCode. Verified by compiling and running it.

**Q173.** What does this print?

```java
import java.util.*; import java.util.stream.*;
public class Main {

public static void main(String[] args) throws Exception {
List<String> w = List.of("apple", "bob", "avocado");
System.out.println(w.stream().filter(s -> s.startsWith("a")).map(String::length).collect(Collectors.toList()));
}
}
```

**Answer:** [5, 7]

Streams are lazy until a terminal operation. Verified by compiling and running it.

**Q174.** What does this print?

```java
public class Main {

public static void main(String[] args) throws Exception {
int[] c = {0};
Object lock = new Object();
Thread[] ts = new Thread[4];
for (int i = 0; i < 4; i++) { ts[i] = new Thread(() -> { for (int j = 0; j < 1000; j++) synchronized (lock) { c[0]++; } }); ts[i].start(); }
for (Thread t : ts) t.join();
System.out.println(c[0]);
}
}
```

**Answer:** 4000

synchronized prevents lost updates. Verified by compiling and running it.

**Q175.** What does this print?

```java
import java.util.*; import java.util.concurrent.*;
public class Main {

public static void main(String[] args) throws Exception {
ExecutorService ex = Executors.newFixedThreadPool(3);
int s = 0;
List<Callable<Integer>> tasks = List.of(() -> 1, () -> 2, () -> 3);
for (Future<Integer> f : ex.invokeAll(tasks)) s += f.get();
ex.shutdown();
System.out.println(s);
}
}
```

**Answer:** 6

Pool tasks return results through Futures. Verified by compiling and running it.

**Q176.** A HashMap lookup returns null for a key that was inserted. What's the likely bug?

**Answer:** The key class overrides equals() without hashCode() (or mutates after insertion)

Keep them consistent.

**Q177.** A loop concatenates strings 100,000 times and is slow. What's the fix?

**Answer:** Use StringBuilder (strings are immutable)

Avoid quadratic copying.

**Q178.** Two threads each lock account A then B, and B then A. What happens and what's the fix?

**Answer:** Deadlock; always acquire locks in a consistent global order

Lock ordering.

**Q179.** A REST endpoint concatenates user input into SQL. What's the fix?

**Answer:** PreparedStatement/JPA parameters

Prevents injection.

**Q180.** Passwords are stored with SHA-256. What should you use?

**Answer:** BCrypt/Argon2 via PasswordEncoder

Slow salted hashing.

**Q181.** Spring Boot service is slow due to many queries for a list with children. What's this?

**Answer:** The N+1 select problem; use fetch joins/batch fetching

JPA tuning.

**Q182.** A container gets OOM-killed though the heap seems small. What setting helps?

**Answer:** Container-aware sizing (-XX:MaxRAMPercentage) and checking non-heap memory

JVM in containers.

**Q183.** A consumer processes the same Kafka message twice after a rebalance. What must the handler be?

**Answer:** Idempotent

At-least-once delivery.

## Practical coding questions

### Reports with records and streams

Implement revenueByCategory (total per category, sorted by name) and topCustomers (top n by total spend, ties by name, formatted name=12.34) with the Streams API, using the Order record. The program also prints summary statistics.

Starter code:

```
import java.util.*;
import java.util.stream.*;

public class Main {
    record Order(String customer, String category, int qty, double price) {
        double total() { return qty * price; }
    }

    static Map<String, Double> revenueByCategory(List<Order> orders) {
        // total revenue per category, sorted by category name (use groupingBy with a TreeMap + summingDouble)
        return new TreeMap<>();
    }

    static List<String> topCustomers(List<Order> orders, int n) {
        // the n customers with the highest total spend (ties by name), formatted "name=12.34"
        return List.of();
    }

    public static void main(String[] args) {
        List<Order> orders = List.of(
                new Order("ada", "books", 2, 12.50), new Order("bo", "games", 1, 59.99),
                new Order("ada", "games", 1, 30.00), new Order("cy", "books", 4, 9.99),
                new Order("bo", "books", 1, 12.50));
        System.out.println(revenueByCategory(orders));
        System.out.println(topCustomers(orders, 2));
        System.out.println(orders.stream().mapToInt(Order::qty).summaryStatistics());
    }
}
```

Expected output:

```text
{books=77.46, games=89.99}
[bo=72.49, ada=55.00]
IntSummaryStatistics{count=5, sum=9, min=1, average=1.800000, max=4}
```

??? success "Show a solution"

    ```
    import java.util.*;
    import java.util.stream.*;

    public class Main {
        record Order(String customer, String category, int qty, double price) {
            double total() { return qty * price; }
        }

        static Map<String, Double> revenueByCategory(List<Order> orders) {
            return orders.stream().collect(Collectors.groupingBy(Order::category, TreeMap::new,
                    Collectors.collectingAndThen(Collectors.summingDouble(Order::total), v -> Math.round(v * 100) / 100.0)));
        }

        static List<String> topCustomers(List<Order> orders, int n) {
            return orders.stream()
                    .collect(Collectors.groupingBy(Order::customer, Collectors.summingDouble(Order::total)))
                    .entrySet().stream()
                    .sorted(Map.Entry.<String, Double>comparingByValue().reversed().thenComparing(Map.Entry.comparingByKey()))
                    .limit(n)
                    .map(e -> e.getKey() + "=" + String.format("%.2f", e.getValue()))
                    .collect(Collectors.toList());
        }

        public static void main(String[] args) {
            List<Order> orders = List.of(
                    new Order("ada", "books", 2, 12.50), new Order("bo", "games", 1, 59.99),
                    new Order("ada", "games", 1, 30.00), new Order("cy", "books", 4, 9.99),
                    new Order("bo", "books", 1, 12.50));
            System.out.println(revenueByCategory(orders));
            System.out.println(topCustomers(orders, 2));
            System.out.println(orders.stream().mapToInt(Order::qty).summaryStatistics());
        }
    }
    ```

    groupingBy with a TreeMap and summingDouble builds the category totals; the customer ranking sorts map entries by value descending then key. Verified by running the solution with java Main.java.

### A deadlock-free thread-safe transfer

Implement transfer(from, to, amount): lock both accounts (always the smaller id first), refuse overdrafts, move the money, and unlock in finally blocks. Run 2,000 concurrent opposite-direction transfers on a pool and check the total is preserved.

Starter code:

```
import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    static class Account {
        final int id;
        private long balance;
        final ReentrantLock lock = new ReentrantLock();
        Account(int id, long balance) { this.id = id; this.balance = balance; }
        long balance() { return balance; }
        void adjust(long d) { balance += d; }
    }

    /** Move money atomically between two accounts and refuse overdrafts (return false). Always lock the
     *  account with the smaller id first so opposite-direction transfers cannot deadlock. */
    static boolean transfer(Account from, Account to, long amount) {
        return false;
    }

    public static void main(String[] args) throws Exception {
        Account a = new Account(1, 1_000), b = new Account(2, 1_000);
        ExecutorService pool = Executors.newFixedThreadPool(8);
        List<Future<Boolean>> results = new ArrayList<>();
        for (int i = 0; i < 2_000; i++) {
            final boolean forward = i % 2 == 0;
            results.add(pool.submit(() -> forward ? transfer(a, b, 7) : transfer(b, a, 5)));
        }
        for (Future<Boolean> f : results) f.get();
        pool.shutdown();
        System.out.println("total preserved: " + (a.balance() + b.balance() == 2_000));
        System.out.println("overdraft refused: " + !transfer(a, b, 1_000_000));
    }
}
```

Expected output:

```text
total preserved: true
overdraft refused: true
```

??? success "Show a solution"

    ```
    import java.util.*;
    import java.util.concurrent.*;
    import java.util.concurrent.locks.ReentrantLock;

    public class Main {
        static class Account {
            final int id;
            private long balance;
            final ReentrantLock lock = new ReentrantLock();
            Account(int id, long balance) { this.id = id; this.balance = balance; }
            long balance() { return balance; }
            void adjust(long d) { balance += d; }
        }

        /** Move money atomically. Locks are always taken in id order, so opposite transfers can't deadlock. */
        static boolean transfer(Account from, Account to, long amount) {
            Account first = from.id < to.id ? from : to;
            Account second = first == from ? to : from;
            first.lock.lock();
            try {
                second.lock.lock();
                try {
                    if (from.balance() < amount) return false;
                    from.adjust(-amount);
                    to.adjust(amount);
                    return true;
                } finally {
                    second.lock.unlock();
                }
            } finally {
                first.lock.unlock();
            }
        }

        public static void main(String[] args) throws Exception {
            Account a = new Account(1, 1_000), b = new Account(2, 1_000);
            ExecutorService pool = Executors.newFixedThreadPool(8);
            List<Future<Boolean>> results = new ArrayList<>();
            for (int i = 0; i < 2_000; i++) {
                final boolean forward = i % 2 == 0;
                results.add(pool.submit(() -> forward ? transfer(a, b, 7) : transfer(b, a, 5)));
            }
            for (Future<Boolean> f : results) f.get();
            pool.shutdown();
            System.out.println("total preserved: " + (a.balance() + b.balance() == 2_000));
            System.out.println("overdraft refused: " + !transfer(a, b, 1_000_000));
        }
    }
    ```

    Taking locks in a consistent global order removes the circular wait that causes deadlock, and finally blocks guarantee unlocking. The invariant (total balance) holds because each transfer is atomic. Verified by running the solution with java Main.java.

---

More free practice: the [Java Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
