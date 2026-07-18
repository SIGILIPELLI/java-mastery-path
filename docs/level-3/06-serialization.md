# 06 · Serialization

Serialization converts an object's in-memory state into a format that can be
stored or transmitted, and deserialization reconstructs it. Java has a
built-in binary mechanism, but modern applications almost always use JSON
instead — this module covers both.

## `java.io.Serializable`

A class becomes serializable simply by implementing the `Serializable`
marker interface (it has no methods to implement):

```java
import java.io.Serializable;

public class Employee implements Serializable {
    private static final long serialVersionUID = 1L;   // versioning id, explained below

    private String name;
    private double salary;

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    @Override
    public String toString() {
        return "Employee{name='" + name + "', salary=" + salary + "}";
    }
}
```

## `serialVersionUID`

The JVM uses `serialVersionUID` to check that a serialized object's class
matches the class currently on the classpath during deserialization. If you
don't declare it explicitly, Java computes one automatically from the class's
structure — which means adding or removing a field can silently invalidate
*previously serialized* data (`InvalidClassException`). Declaring it
explicitly and bumping it deliberately when you make an incompatible change is
the safer practice.

## Writing an object — `ObjectOutputStream`

```java
import java.io.*;

public class SerializeDemo {
    public static void main(String[] args) throws IOException {
        Employee alice = new Employee("Alice", 75000.0);

        try (ObjectOutputStream out =
                     new ObjectOutputStream(new FileOutputStream("employee.ser"))) {
            out.writeObject(alice);
        }
        System.out.println("Saved: " + alice);
    }
}
```

## Reading an object — `ObjectInputStream`

```java
import java.io.*;

public class DeserializeDemo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        try (ObjectInputStream in =
                     new ObjectInputStream(new FileInputStream("employee.ser"))) {
            Employee loaded = (Employee) in.readObject();   // requires an explicit cast
            System.out.println("Loaded: " + loaded);
        }
    }
}
// Output:
// Loaded: Employee{name='Alice', salary=75000.0}
```

## Why raw Java serialization is discouraged for external data

Java's built-in serialization is convenient for quick same-JVM persistence,
but it has real drawbacks for anything crossing a process or language
boundary:

- **Not language-agnostic** — the binary format is Java-specific; nothing
  written in another language can read it.
- **Security risk** — deserializing untrusted bytes can execute arbitrary
  code via crafted payloads ("deserialization gadget chains"); this is a
  well-known class of vulnerability.
- **Fragile versioning** — as noted above, class changes can break
  compatibility with previously serialized data.
- **Not human-readable** — impossible to inspect or debug without code.

For anything sent over a network, saved to a config file, or shared between
services, JSON (or similar text formats) is the practical default.

## JSON with Jackson

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.1</version>
</dependency>
```

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.annotation.JsonProperty;

public class Person {
    @JsonProperty("full_name")   // maps this field to a different JSON key
    private String fullName;
    private int age;

    public Person() { }   // Jackson needs a no-arg constructor for deserialization

    public Person(String fullName, int age) {
        this.fullName = fullName;
        this.age = age;
    }

    public String getFullName() { return fullName; }
    public int getAge() { return age; }

    @Override
    public String toString() {
        return "Person{fullName='" + fullName + "', age=" + age + "}";
    }
}

public class JacksonDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        Person alice = new Person("Alice Smith", 30);

        String json = mapper.writeValueAsString(alice);
        System.out.println(json);
        // {"full_name":"Alice Smith","age":30}

        Person parsed = mapper.readValue(json, Person.class);
        System.out.println(parsed);
        // Person{fullName='Alice Smith', age=30}
    }
}
```

Records work with Jackson too, with no annotations needed for the basic case
— Jackson matches JSON keys to record component names automatically.

```java
public record Point(int x, int y) { }

ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(new Point(3, 4));   // {"x":3,"y":4}
Point p = mapper.readValue(json, Point.class);              // Point[x=3, y=4]
```

| Approach | Format | Best for |
|----------|--------|----------|
| `Serializable` + `ObjectOutputStream` | Binary, Java-only | Quick same-JVM persistence, caching |
| Jackson `ObjectMapper` | JSON, text, cross-language | APIs, config files, inter-service data |
| `@JsonProperty` | — | Renaming a Java field's JSON key |

Spring Boot uses Jackson automatically under the hood to serialize
`@RestController` responses to JSON — see
[Level 3, Module 4](04-rest-apis-spring-boot.md).

## Exercise

Create a `Product` record with `name`, `price`, and `inStock` fields. Use
Jackson's `ObjectMapper` to serialize a list of three `Product` instances to a
JSON array string with `writeValueAsString`, print it, then deserialize it
back into a `List<Product>` using `mapper.readValue(json, new
TypeReference<List<Product>>() {})` and print each product.
