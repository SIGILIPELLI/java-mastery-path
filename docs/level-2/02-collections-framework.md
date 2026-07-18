# 02 · Collections Framework

[Level 1](../level-1/05-arrays-collections.md) introduced `ArrayList` as a
resizable array. Java's Collections Framework is much bigger than that — a
family of interfaces and implementations for lists, sets, and maps, each with
different performance trade-offs.

## The Collection hierarchy

At the top sits the `Collection` interface, with three major sub-families:

| Interface | Represents | Allows duplicates? | Ordered? |
|-----------|-----------|---------------------|----------|
| `List` | An ordered, indexable sequence | Yes | Yes (insertion order) |
| `Set` | A collection of unique elements | No | Depends on implementation |
| `Map` | Key → value pairs (not a `Collection`, but part of the framework) | Keys: no, Values: yes | Depends on implementation |

## List implementations: ArrayList vs. LinkedList

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

List<String> arrayList = new ArrayList<>();   // backed by a resizable array
List<String> linkedList = new LinkedList<>();  // backed by a doubly-linked list

arrayList.add("a");
arrayList.add("b");
linkedList.add("x");
linkedList.add(0, "w");   // insert at the front
```

- **`ArrayList`** — fast random access (`get(i)` is O(1)); adding/removing in
  the middle requires shifting elements (O(n)). Use it as your default list.
- **`LinkedList`** — fast insertion/removal at the ends (O(1)); `get(i)` must
  walk the chain (O(n)). Also implements `Deque`, so it doubles as a stack or
  queue.

## Set implementations: HashSet vs. TreeSet vs. LinkedHashSet

```java
import java.util.*;

Set<String> hashSet = new HashSet<>();        // no guaranteed order
Set<String> treeSet = new TreeSet<>();         // sorted order
Set<String> linkedHashSet = new LinkedHashSet<>();   // insertion order

List.of("banana", "apple", "cherry", "apple").forEach(fruit -> {
    hashSet.add(fruit);
    treeSet.add(fruit);
    linkedHashSet.add(fruit);
});

System.out.println(treeSet);         // [apple, banana, cherry] -- sorted, duplicate dropped
System.out.println(linkedHashSet);   // [banana, apple, cherry] -- insertion order preserved
```

| Set type | Ordering | Typical use |
|----------|----------|--------------|
| `HashSet` | None guaranteed | Fastest add/contains, when order doesn't matter |
| `LinkedHashSet` | Insertion order | Fast + predictable iteration order |
| `TreeSet` | Sorted (natural or `Comparator`) | Need elements sorted at all times |

## Map implementations: HashMap vs. TreeMap vs. LinkedHashMap

```java
Map<String, Integer> hashMap = new HashMap<>();
Map<String, Integer> treeMap = new TreeMap<>();
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();

hashMap.put("banana", 3);
hashMap.put("apple", 5);
hashMap.put("cherry", 8);

treeMap.putAll(hashMap);
linkedHashMap.putAll(hashMap);

System.out.println(treeMap);   // {apple=5, banana=3, cherry=8} -- sorted by key
```

Same trade-offs as the `Set` family, since `HashSet`/`TreeSet`/`LinkedHashSet`
are actually implemented on top of `HashMap`/`TreeMap`/`LinkedHashMap`
internally.

## Iterating safely

Using a plain for-each loop is fine for reading, but modifying a collection
while iterating over it with an enhanced for-loop throws
`ConcurrentModificationException`:

```java
List<Integer> numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5));

// DON'T do this:
for (Integer n : numbers) {
    if (n % 2 == 0) {
        numbers.remove(n);   // throws ConcurrentModificationException
    }
}
```

Use an explicit `Iterator` and its `remove()` method instead, or filter into a
new collection:

```java
Iterator<Integer> it = numbers.iterator();
while (it.hasNext()) {
    int n = it.next();
    if (n % 2 == 0) {
        it.remove();   // safe -- goes through the iterator
    }
}
System.out.println(numbers);   // [1, 3, 5]

// Or, often cleaner:
numbers.removeIf(n -> n % 2 == 0);
```

## Comparable — a class's natural order

Implement `Comparable<T>` when a type has one obvious, intrinsic ordering.

```java
public class Employee implements Comparable<Employee> {
    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() { return name; }
    public double getSalary() { return salary; }

    @Override
    public int compareTo(Employee other) {
        return Double.compare(this.salary, other.salary);   // ascending by salary
    }

    @Override
    public String toString() {
        return name + ": $" + salary;
    }
}

List<Employee> team = new ArrayList<>(List.of(
    new Employee("Alice", 75000),
    new Employee("Bob", 62000),
    new Employee("Cara", 90000)
));

Collections.sort(team);        // uses compareTo
System.out.println(team);      // [Bob: $62000.0, Alice: $75000.0, Cara: $90000.0]
```

## Comparator — external, swappable orderings

`Comparator<T>` lets you define orderings *outside* the class, and you can
have as many as you need — handy when there's more than one reasonable way to
sort something.

```java
import java.util.Comparator;

Comparator<Employee> byName = Comparator.comparing(Employee::getName);
Comparator<Employee> bySalaryDesc = Comparator.comparingDouble(Employee::getSalary).reversed();

team.sort(byName);
System.out.println(team);   // sorted alphabetically

team.sort(bySalaryDesc);
System.out.println(team);   // highest salary first

// Chaining: sort by name, then by salary as a tiebreaker
Comparator<Employee> byNameThenSalary =
    Comparator.comparing(Employee::getName).thenComparing(Employee::getSalary);
team.sort(byNameThenSalary);
```

## Sorting a Map's entries

Maps aren't directly sortable (except `TreeMap`, which is always sorted by
key) — but you can stream a map's `entrySet()` and sort that.

```java
Map<String, Integer> scores = Map.of("Alice", 90, "Bob", 75, "Cara", 95);

scores.entrySet().stream()
    .sorted(Map.Entry.comparingByValue(Comparator.reverseOrder()))
    .forEach(e -> System.out.println(e.getKey() + " -> " + e.getValue()));
// Output:
// Cara -> 95
// Alice -> 90
// Bob -> 75
```

## `Comparable` vs. `Comparator`

| | `Comparable` | `Comparator` |
|---|---|---|
| Method | `compareTo(T other)` | `compare(T a, T b)` |
| Defined | Inside the class being compared | Anywhere, as a separate object |
| How many orderings | One ("natural order") | As many as you like |
| Typical use | `Collections.sort(list)` | `list.sort(comparator)` |

## Exercise

Create a `Student` class with `name` (String) and `gpa` (double), implementing
`Comparable<Student>` by natural order of GPA descending (highest GPA first).
Build a `List<Student>` with at least four students, sort it with
`Collections.sort` and print the result. Then write a `Comparator<Student>`
that sorts by name alphabetically, and print the list sorted that way instead
— without modifying the `compareTo` you wrote first.
