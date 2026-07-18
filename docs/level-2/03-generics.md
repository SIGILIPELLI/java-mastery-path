# 03 · Generics

Generics let a class, method, or interface operate on a type that's decided
when it's *used*, not when it's written — the same way `List<String>` and
`List<Integer>` share one `List` implementation. This module shows how to
write your own generic code.

## Why generics — the problem they solve

Before generics, a general-purpose container had to store `Object` and cast
back, which is both verbose and unsafe (the compiler can't catch a wrong
cast):

```java
// The old, unsafe way
public class RawBox {
    private Object value;
    public void set(Object value) { this.value = value; }
    public Object get() { return value; }
}

RawBox box = new RawBox();
box.set("hello");
Integer n = (Integer) box.get();   // compiles, but throws ClassCastException at runtime!
```

## A generic class

```java
public class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

Box<String> stringBox = new Box<>();
stringBox.set("hello");
String s = stringBox.get();   // no cast needed -- the compiler knows it's a String
// stringBox.set(42);         // won't compile -- type safety enforced
```

`T` is a **type parameter** — a placeholder that gets filled in with a real
type (`String`, `Integer`, `Employee`, ...) at the point of use.

## A generic class with two type parameters

```java
public class Pair<A, B> {
    private final A first;
    private final B second;

    public Pair(A first, B second) {
        this.first = first;
        this.second = second;
    }

    public A getFirst() { return first; }
    public B getSecond() { return second; }

    @Override
    public String toString() {
        return "(" + first + ", " + second + ")";
    }
}

Pair<String, Integer> nameAge = new Pair<>("Alice", 30);
System.out.println(nameAge);            // (Alice, 30)
System.out.println(nameAge.getFirst());  // Alice
```

## Generic methods

A single method can introduce its own type parameter, independent of the
class it lives in — useful for static utility methods.

```java
public class ArrayUtils {
    // <T> before the return type declares the type parameter for this method
    public static <T> T firstElement(T[] array) {
        if (array.length == 0) {
            throw new IllegalArgumentException("Array is empty");
        }
        return array[0];
    }

    public static <T> void printAll(List<T> items) {
        for (T item : items) {
            System.out.println(item);
        }
    }
}

String[] names = {"Alice", "Bob"};
String first = ArrayUtils.firstElement(names);   // T is inferred as String
System.out.println(first);   // Alice
```

## Bounded type parameters

Sometimes a generic method needs to *do* something with the type beyond
storing and returning it — like compare two values. `extends` restricts the
type parameter to types that provide the needed capability.

```java
// T must be Comparable to itself so we can call compareTo
public static <T extends Comparable<T>> T max(List<T> items) {
    T largest = items.get(0);
    for (T item : items) {
        if (item.compareTo(largest) > 0) {
            largest = item;
        }
    }
    return largest;
}

System.out.println(max(List.of(3, 7, 2, 9, 4)));         // 9
System.out.println(max(List.of("pear", "apple", "kiwi"))); // pear
```

`<T extends Comparable<T>>` reads as "T can be any type, as long as it
implements `Comparable<T>`." A bound can also require a class (rather than an
interface), or combine one class with multiple interfaces
(`<T extends Animal & Comparable<T>>`).

## Wildcards: `?`, `? extends`, `? super`

Wildcards describe an *unknown* type when you don't need to name it — most
often as a parameter type for methods that only read from or only write to a
generic collection.

```java
// Unbounded wildcard -- accepts a List of any type, we only care about size
public static void printSize(List<?> list) {
    System.out.println("Size: " + list.size());
}

// Upper-bounded wildcard ("? extends Number") -- read-only access to any
// list of Number or its subclasses (Integer, Double, ...)
public static double sumAll(List<? extends Number> numbers) {
    double total = 0;
    for (Number n : numbers) {
        total += n.doubleValue();
    }
    return total;
}

System.out.println(sumAll(List.of(1, 2, 3)));        // 6.0 -- List<Integer>
System.out.println(sumAll(List.of(1.5, 2.5)));        // 4.0 -- List<Double>

// Lower-bounded wildcard ("? super Integer") -- safe to add Integers into
// any list of Integer or its supertypes (Number, Object)
public static void addNumbers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
}

List<Number> numberList = new ArrayList<>();
addNumbers(numberList);
System.out.println(numberList);   // [1, 2]
```

The common mnemonic is **PECS — Producer `extends`, Consumer `super`**: use
`? extends T` when you only read `T`s out of the structure, and `? super T`
when you only put `T`s into it.

| Wildcard | Meaning | Typical use |
|----------|---------|-------------|
| `List<?>` | Unknown type | Read-only, type-agnostic operations (size, clear) |
| `List<? extends T>` | T or any subtype | Producer -- safe to read as T |
| `List<? super T>` | T or any supertype | Consumer -- safe to write T into it |

## Type erasure

Generic type information exists only at compile time — the compiler checks
it, then erases it, compiling `Box<String>` and `Box<Integer>` down to the
same `Box` bytecode using `Object` internally (with automatic casts inserted
for you). Consequences worth knowing:

```java
Box<String> b1 = new Box<>();
Box<Integer> b2 = new Box<>();
System.out.println(b1.getClass() == b2.getClass());   // true -- both are just Box

// You cannot create a generic array or check a generic type at runtime:
// T[] array = new T[10];         // won't compile
// if (obj instanceof Box<String>) { }   // won't compile ("unchecked" / erased)
```

## Why avoid raw types

Using a generic class without a type argument (a "raw type", like the
`RawBox` example above, or `List list = new ArrayList();`) disables all
compile-time type checking for that variable — you lose the entire benefit of
generics and reintroduce the risk of `ClassCastException`. Always supply a
type argument (or `<>, the diamond operator, when it can be inferred).

## Exercise

Write a generic class `Stack<T>` backed by a `List<T>` (or an array), with
methods `push(T item)`, `T pop()` (removes and returns the top item, throwing
`IllegalStateException` if empty), and `boolean isEmpty()`. Then write a
generic static method `<T extends Comparable<T>> boolean isSorted(List<T> list)`
that returns `true` if the list is in non-decreasing order. Test both with a
`Stack<String>` and a `List<Integer>`.
