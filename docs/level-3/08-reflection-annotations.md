# 08 · Reflection & Annotations

Reflection lets a program inspect and manipulate classes, fields, methods, and
annotations *at runtime*, rather than everything being fixed at compile time.
It's the mechanism behind frameworks like Spring and Jackson, which need to
work with arbitrary user classes they've never seen at compile time.

## `Class<?>` basics

Every object carries a runtime `Class` reference describing its type.

```java
public class ReflectionBasics {
    public static void main(String[] args) throws ClassNotFoundException {
        String s = "hello";

        Class<?> c1 = s.getClass();                     // from an instance
        Class<?> c2 = String.class;                     // from a class literal
        Class<?> c3 = Class.forName("java.lang.String"); // by fully qualified name

        System.out.println(c1 == c2);            // true -- only one Class object per type
        System.out.println(c3.getSimpleName());  // String
        System.out.println(c1.getName());         // java.lang.String
    }
}
```

`Class.forName` is especially useful when the class name is only known at
runtime (e.g. read from a config file), since you can't write a `.class`
literal for something you don't know the name of at compile time.

## Inspecting fields and methods

```java
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Widget {
    private String label = "Save";
    private int clickCount = 0;

    public void click() { clickCount++; }
    private void reset() { clickCount = 0; }
}

public class InspectDemo {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Widget.class;

        System.out.println("Fields:");
        for (Field f : clazz.getDeclaredFields()) {
            System.out.println("  " + f.getType().getSimpleName() + " " + f.getName());
        }

        System.out.println("Methods:");
        for (Method m : clazz.getDeclaredMethods()) {
            System.out.println("  " + m.getName());
        }
    }
}
// Output:
// Fields:
//   String label
//   int clickCount
// Methods:
//   click
//   reset
```

`getDeclaredFields`/`getDeclaredMethods` return everything declared directly
on the class, including `private` members — unlike `getFields`/`getMethods`,
which only return `public` members (including inherited ones).

## Invoking a method reflectively

Private members are normally inaccessible from outside the class — reflection
can bypass that with `setAccessible(true)`, which is exactly how frameworks
call your private methods and set your private fields.

```java
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class InvokeDemo {
    public static void main(String[] args) throws Exception {
        Widget widget = new Widget();

        Method click = Widget.class.getDeclaredMethod("click");
        click.invoke(widget);   // equivalent to widget.click()
        click.invoke(widget);

        Field clickCount = Widget.class.getDeclaredField("clickCount");
        clickCount.setAccessible(true);   // bypass "private" -- use sparingly!
        System.out.println("clickCount = " + clickCount.get(widget));   // clickCount = 2
    }
}
```

Reflection is powerful but comes at a cost: it's slower than direct calls,
bypasses compile-time type checking, and can break encapsulation if overused
— reach for it only when you need genuine runtime flexibility (frameworks,
serialization libraries, plugin systems), not in everyday application code.

## Defining a custom annotation

An annotation is a form of metadata attached to code. You define one with
`@interface`, and control its runtime visibility with `@Retention`:

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)   // keep this annotation available at runtime
@Target(ElementType.METHOD)           // only methods may carry this annotation
public @interface Loggable {
    String value() default "";        // an optional element with a default
}
```

`@Retention(RetentionPolicy.RUNTIME)` is essential — without it (the default
is `CLASS`), the annotation would be discarded after compilation and
invisible to reflection.

## Reading a custom annotation at runtime

```java
import java.lang.reflect.Method;

public class Service {
    @Loggable("audit")
    public void deleteUser(String id) {
        System.out.println("Deleting user " + id);
    }

    public void internalHelper() {
        System.out.println("No annotation here");
    }
}

public class AnnotationDemo {
    public static void main(String[] args) {
        for (Method m : Service.class.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Loggable.class)) {
                Loggable loggable = m.getAnnotation(Loggable.class);
                System.out.println(m.getName() + " is loggable, category=" + loggable.value());
            }
        }
    }
}
// Output:
// deleteUser is loggable, category=audit
```

This is the same mechanism that powers annotations like Spring's
`@RestController` or JUnit's `@Test`: the framework scans your classes at
startup (or at build time), finds methods carrying its annotations, and wires
up behavior around them without you writing any glue code.

| Concept | Purpose |
|---------|---------|
| `Class<?>` | Runtime representation of a type |
| `getDeclaredFields()` / `getDeclaredMethods()` | List all members, including private ones |
| `setAccessible(true)` | Bypass access checks (private/protected) |
| `Method.invoke(obj, args...)` | Call a method reflectively |
| `@interface` | Declares a custom annotation type |
| `@Retention(RetentionPolicy.RUNTIME)` | Keeps the annotation visible to reflection |
| `@Target` | Restricts where the annotation may be applied |

## Exercise

Define a custom annotation `@MinValue(int value())` with
`@Retention(RUNTIME)` and `@Target(ElementType.FIELD)`. Create a class with
several `int` fields annotated with different `@MinValue` thresholds. Write a
`validate(Object obj)` method that uses reflection to inspect every declared
field, reads its `@MinValue` annotation (if present) and its current value via
`Field.get`, and prints a warning for any field whose value is below its
minimum.
