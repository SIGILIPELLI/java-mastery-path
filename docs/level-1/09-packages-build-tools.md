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

## Exercise

Create a Maven project on disk by hand (no need to actually run `mvn` if you
don't have it installed yet — the goal is the structure): a `pom.xml` for an
artifact called `my-first-app`, with `src/main/java/com/example/App.java`
containing a `main` method that prints a message, following the standard
Maven directory layout shown above.
