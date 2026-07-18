# 04 · Lambdas & Streams API

Lambda expressions let you pass a block of behavior around like a value.
Combined with the Streams API, they let you describe *what* transformation
you want on a collection instead of writing manual loops for *how* to do it.

## Functional interfaces

A **functional interface** is any interface with exactly one abstract method
— that single method is what a lambda expression implements. `Runnable` (one
method, `run()`) is a classic built-in example.

```java
Runnable task = () -> System.out.println("Running!");
task.run();   // Running!
```

You can define your own with `@FunctionalInterface` (the annotation isn't
required, but it makes intent clear and the compiler will error if you
accidentally add a second abstract method):

```java
@FunctionalInterface
interface Calculator {
    int operate(int a, int b);
}

Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.operate(3, 4));        // 7
System.out.println(multiply.operate(3, 4));   // 12
```

## Lambda syntax

```java
// Full form
Calculator subtract = (int a, int b) -> { return a - b; };

// Type inference + implicit return (no braces needed for a single expression)
Calculator divide = (a, b) -> a / b;

// No parameters
Runnable greet = () -> System.out.println("Hi!");

// Exactly one parameter -- parentheses optional
java.util.function.Function<String, Integer> length = s -> s.length();
```

## Built-in functional interfaces (`java.util.function`)

Java ships a standard library of general-purpose functional interfaces so you
rarely need to declare your own:

```java
import java.util.function.*;

Function<String, Integer> parseLength = String::length;          // T -> R
Predicate<Integer> isEven = n -> n % 2 == 0;                       // T -> boolean
Consumer<String> print = System.out::println;                     // T -> void
Supplier<Double> random = Math::random;                            // () -> R
BiFunction<Integer, Integer, Integer> sum = (a, b) -> a + b;       // (T, U) -> R

System.out.println(parseLength.apply("hello"));   // 5
System.out.println(isEven.test(7));                // false
print.accept("printed via Consumer");
System.out.println(random.get());                  // some double between 0 and 1
System.out.println(sum.apply(3, 4));                // 7
```

| Interface | Method | Signature | Typical use |
|-----------|--------|-----------|--------------|
| `Function<T,R>` | `apply(T)` | `T -> R` | Transform a value |
| `Predicate<T>` | `test(T)` | `T -> boolean` | A yes/no check, e.g. for `filter` |
| `Consumer<T>` | `accept(T)` | `T -> void` | Do something with a value, no result |
| `Supplier<T>` | `get()` | `() -> T` | Produce/lazily generate a value |
| `BiFunction<T,U,R>` | `apply(T,U)` | `(T,U) -> R` | Combine two values into a result |

## Method references

When a lambda does nothing but call an existing method, a **method
reference** is a shorter, often clearer alternative.

```java
List<String> names = new ArrayList<>(List.of("charlie", "alice", "bob"));

names.forEach(System.out::println);          // instance method on an implied argument
names.sort(String::compareToIgnoreCase);      // instance method, two args
names.replaceAll(String::toUpperCase);        // instance method on the element
names.forEach(name -> print.accept(name));    // equivalent lambda, for comparison
```

| Kind | Example | Equivalent lambda |
|------|---------|--------------------|
| Static method | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| Instance method (bound) | `str::toUpperCase` | `() -> str.toUpperCase()` |
| Instance method (unbound) | `String::toUpperCase` | `s -> s.toUpperCase()` |
| Constructor | `ArrayList::new` | `() -> new ArrayList<>()` |

## The Streams API pipeline

A stream is a one-time-use pipeline over a data source: zero or more
**intermediate operations** (like `filter`, `map`, `sorted` — each returns a
new stream) followed by exactly one **terminal operation** (like `collect`,
`forEach`, `count` — which triggers the actual processing).

```java
import java.util.List;
import java.util.stream.Collectors;

List<String> words = List.of("banana", "kiwi", "apple", "fig", "cherry");

List<String> result = words.stream()
    .filter(w -> w.length() > 4)     // keep words longer than 4 chars
    .map(String::toUpperCase)         // transform each to uppercase
    .sorted()                         // sort alphabetically
    .collect(Collectors.toList());    // terminal op -- materialize into a List

System.out.println(result);   // [APPLE, BANANA, CHERRY]
```

Nothing runs until `collect` is called — intermediate operations just build
up the pipeline description ("lazy evaluation").

## Common terminal operations

```java
List<Integer> numbers = List.of(4, 9, 2, 7, 5, 1);

long count = numbers.stream().filter(n -> n > 3).count();
System.out.println(count);   // 4

int total = numbers.stream().reduce(0, Integer::sum);
System.out.println(total);   // 28

int max = numbers.stream().reduce(Integer.MIN_VALUE, Integer::max);
System.out.println(max);   // 9

// IntStream avoids boxing overhead for primitive sums
int sum = numbers.stream().mapToInt(Integer::intValue).sum();
System.out.println(sum);   // 28

Set<Integer> asSet = numbers.stream().collect(Collectors.toSet());
Map<Boolean, List<Integer>> byParity = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
System.out.println(byParity);   // {false=[9, 7, 5, 1], true=[4, 2]}
```

## A realistic example: filter + map + collect over objects

```java
public record Employee(String name, String department, double salary) {}

List<Employee> employees = List.of(
    new Employee("Alice", "Engineering", 95000),
    new Employee("Bob", "Sales", 62000),
    new Employee("Cara", "Engineering", 88000),
    new Employee("Dave", "Sales", 71000)
);

List<String> highEarnersInEngineering = employees.stream()
    .filter(e -> e.department().equals("Engineering"))
    .filter(e -> e.salary() > 90000)
    .map(Employee::name)
    .collect(Collectors.toList());

System.out.println(highEarnersInEngineering);   // [Alice]

double totalSalary = employees.stream()
    .mapToDouble(Employee::salary)
    .sum();
System.out.println(totalSalary);   // 316000.0

Map<String, List<String>> namesByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::department,
        Collectors.mapping(Employee::name, Collectors.toList())
    ));
System.out.println(namesByDept);
// {Engineering=[Alice, Cara], Sales=[Bob, Dave]}
```

Records (used for `Employee` above) are covered fully in
[Module 9](09-enums-records-sealed.md).

## Exercise

Given a `List<String>` of product names with mixed casing and some
duplicates, use a stream pipeline to: remove duplicates, filter out any name
shorter than 4 characters, convert the rest to title case (first letter
uppercase, `String::toLowerCase` on the remainder is fine for a simple
version), sort alphabetically, and collect into a `List<String>`. Print the
result. Then write a `Predicate<String>` variable that checks whether a
string starts with a given letter, and use it inside a second `filter` call.
