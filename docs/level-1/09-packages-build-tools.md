# 09 · Packages & Build Tools Intro

## Packages

A package is a namespace that groups related classes and avoids naming
collisions. The package declaration must be the first line of the file, and
the folder structure must mirror the package name.

```java
// src/com/example/contacts/Contact.java
package com.example.contacts;

public class Contact {
    private String name;
    private String phone;

    public Contact(String name, String phone) {
        this.name = name;
        this.phone = phone;
    }

    public String getName() { return name; }
    public String getPhone() { return phone; }
}
```

```java
// src/com/example/contacts/Main.java
package com.example.contacts;

public class Main {
    public static void main(String[] args) {
        Contact c = new Contact("Alice", "555-1234");
        System.out.println(c.getName() + ": " + c.getPhone());
    }
}
```

Compiling and running a package by hand from the project root:

```bash
javac -d out src/com/example/contacts/*.java
java -cp out com.example.contacts.Main
```

## Importing classes from other packages

```java
package com.example.app;

import java.util.List;
import java.util.ArrayList;
import com.example.contacts.Contact;   // import your own package too

public class Main {
    public static void main(String[] args) {
        List<Contact> contacts = new ArrayList<>();
        contacts.add(new Contact("Bob", "555-5678"));
    }
}
```

Classes in `java.lang` (like `String`, `System`, `Math`) are imported
automatically — no `import` needed.

## Why build tools?

Compiling every file by hand with `javac` and tracking classpaths gets
unmanageable fast, especially once a project depends on external libraries.
**Maven** and **Gradle** are the two dominant Java build tools — they define a
standard project layout, manage dependencies (downloading them from a shared
repository), and run a repeatable build ("lifecycle": compile → test →
package).

## Maven basics

A Maven project is described by a `pom.xml` ("Project Object Model") in the
project root:

```xml
<!-- pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>contact-book</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
</project>
```

Standard Maven layout:

```text
contact-book/
    pom.xml
    src/
        main/
            java/
                com/example/contacts/Contact.java
                com/example/contacts/Main.java
        test/
            java/
                com/example/contacts/ContactTest.java
```

Common commands:

```bash
mvn compile      # compiles src/main/java
mvn test         # runs tests in src/test/java
mvn package      # builds a .jar into target/
mvn clean        # removes the target/ build output
```

## Adding a dependency

Dependencies are declared in `pom.xml` and Maven downloads them automatically
from Maven Central:

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

After adding this, `import org.junit.jupiter.api.Test;` becomes available in
your test code. JUnit itself is covered in depth in
[Level 2](../level-2/07-junit-testing.md); Maven/Gradle lifecycle in more
depth in [Level 2, Module 8](../level-2/08-build-tools.md).

| Tool | Config file | Philosophy |
|------|-------------|------------|
| Maven | `pom.xml` (XML) | Convention over configuration, declarative |
| Gradle | `build.gradle` / `build.gradle.kts` | Flexible, script-based (Groovy or Kotlin DSL) |

## How It Actually Works

A package name isn't just documentation — it becomes part of the
**fully-qualified binary name** stored in every class file's constant
pool (`com/example/Foo`), and the classpath is literally a list of
directories/JARs the class loader scans, expecting to find
`com/example/Foo.class` at the matching relative path. Two classes with
the same simple name in different packages are completely distinct types
to the JVM — there's no collision because the loader keys classes by
`(defining loader, fully-qualified name)`, not simple name.

A JAR file is just a ZIP with a `META-INF/MANIFEST.MF`; when you run
`java -jar app.jar`, the JVM reads `Main-Class:` from that manifest to
find the entry point via reflection — the same
`getMethod("main", String[].class)` lookup as running a class directly.

Maven/Gradle builds matter mechanically because compilation order and
the classpath they assemble determine which class file wins when the
**same fully-qualified class name** appears in two dependency JARs
(classpath order decides, silently — "JAR hell") — the class loader takes
the first one it finds and never checks the rest, which is why dependency
conflicts often manifest as confusing `NoSuchMethodError`s at runtime
rather than build failures.

## Exercise

Create a Maven project on disk by hand (no need to actually run `mvn` if you
don't have it installed yet — the goal is the structure): a `pom.xml` for an
artifact called `my-first-app`, with `src/main/java/com/example/App.java`
containing a `main` method that prints a message, following the standard
Maven directory layout shown above.
