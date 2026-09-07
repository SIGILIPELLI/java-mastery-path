# 08 · Build Tools Deep Dive

[Level 1, Module 9](../level-1/09-packages-build-tools.md) introduced Maven's
`pom.xml` and the basic `compile`/`test`/`package` commands. This module goes
deeper into the full lifecycle, dependency scopes, and Gradle as an
alternative.

## The full Maven lifecycle

Maven organizes a build into a sequence of numbered **phases**. Running any
phase automatically runs every phase before it, in order:

```text
validate  -> compile -> test -> package -> verify -> install -> deploy
```

| Phase | What happens |
|-------|--------------|
| `validate` | Checks the project structure and `pom.xml` are correct |
| `compile` | Compiles `src/main/java` into `target/classes` |
| `test` | Compiles and runs `src/test/java` against a test framework (e.g. JUnit) |
| `package` | Bundles compiled code into a distributable `.jar` (or `.war`) |
| `verify` | Runs additional checks (e.g. integration tests) on the packaged artifact |
| `install` | Copies the artifact into your local `~/.m2` repository, for other local projects to depend on |
| `deploy` | Publishes the artifact to a shared/remote repository for other people's projects |

```bash
mvn compile    # runs validate, compile
mvn test       # runs validate, compile, test
mvn package    # ...through package -- produces target/my-app-1.0.0.jar
mvn install    # ...through install -- available to other local Maven projects
```

`mvn clean` is special — it isn't part of the main lifecycle above, it just
deletes `target/`. It's commonly chained: `mvn clean package`.

## Dependency scopes

Every `<dependency>` in `pom.xml` has a **scope** controlling when it's
available and whether it ships with the final artifact:

```xml
<dependencies>
    <!-- default scope: available everywhere, bundled in the final artifact -->
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>33.0.0-jre</version>
        <scope>compile</scope>
    </dependency>

    <!-- only available while compiling/running tests, never shipped -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>

    <!-- needed to compile against, but supplied by the runtime environment
         (e.g. a servlet container providing the Servlet API) -->
    <dependency>
        <groupId>jakarta.servlet</groupId>
        <artifactId>jakarta.servlet-api</artifactId>
        <version>6.0.0</version>
        <scope>provided</scope>
    </dependency>

    <!-- needed only at runtime, not for compiling your own code against -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.3.0</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

| Scope | Available at compile time | Available at test time | Bundled/shipped |
|-------|---------------------------|--------------------------|------------------|
| `compile` (default) | Yes | Yes | Yes |
| `test` | No | Yes | No |
| `provided` | Yes | Yes | No (assumed present at runtime) |
| `runtime` | No | Yes | Yes |

## Multi-module basics

A large project can be split into several Maven modules sharing one parent
`pom.xml`. The parent declares `<modules>`; each child has its own `pom.xml`
with `<parent>` pointing back up:

```xml
<!-- parent pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>inventory-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>inventory-core</module>
        <module>inventory-cli</module>
    </modules>
</project>
```

Running `mvn install` from the parent directory builds every module in
dependency order. This is enough to recognize the pattern when you meet it —
full multi-module configuration is out of scope for this course.

## Gradle as an alternative

Gradle uses a script-based configuration (Groovy or, more commonly today, the
Kotlin DSL, `build.gradle.kts`) instead of Maven's declarative XML. Here is a
rough Gradle equivalent of the JUnit-enabled `pom.xml` from
[Module 7](07-junit-testing.md):

```kotlin
// build.gradle.kts
plugins {
    id("java")
}

group = "com.example"
version = "1.0.0"

repositories {
    mavenCentral()
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.2")
}

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

tasks.test {
    useJUnitPlatform()
}
```

```bash
gradle build     # if Gradle is installed globally
./gradlew build  # using the "Gradle wrapper" checked into the project -- preferred,
                 # guarantees everyone uses the same Gradle version
```

## Maven vs. Gradle

| | Maven | Gradle |
|---|-------|--------|
| Config format | XML (`pom.xml`) | Groovy or Kotlin DSL (`build.gradle[.kts]`) |
| Philosophy | Convention over configuration, declarative | Flexible, imperative/declarative mix |
| Dependency scopes | `compile`, `test`, `provided`, `runtime` | `implementation`, `testImplementation`, `compileOnly`, `runtimeOnly` |
| Build/test command | `mvn package` / `mvn test` | `gradle build` / `gradle test` |
| Wrapper | `mvnw` | `gradlew` |

Both download dependencies from the same shared repository (Maven Central)
and solve the same underlying problem — the choice is largely ecosystem and
team convention.

## How It Actually Works

Maven's dependency resolution builds a full **dependency graph**, not a
flat list — when two dependencies transitively pull in different
versions of the same library, Maven's "nearest wins" rule (shortest path
in the graph, ties broken by declaration order) picks exactly one
version, and every one of your compiled classes that reference that
library get linked against whichever version wins, silently, at build
time — this is the mechanical root cause of dependency-version
conflicts that surface only at runtime as `NoSuchMethodError` when the
resolved JAR is missing a method your code was compiled against.

Gradle's incremental build model is why it's typically faster on
repeated builds: each task declares its inputs/outputs, and Gradle
hashes them to build a task graph, skipping any task whose inputs are
unchanged since the last run (`UP-TO-DATE`) — genuinely reusing prior
work rather than a caching illusion, and it can go further with a build
cache keyed on those same input hashes, sharing results across machines.

Both tools ultimately just orchestrate `javac` (or a compiler daemon
that keeps the compiler warm across builds to skip JVM startup cost) and
package the resulting `.class` files into a JAR — the build tool adds
dependency resolution and lifecycle sequencing on top of primitives you
could still run by hand with `javac`/`jar`.

## Exercise

Take the `pom.xml` from [Module 7](07-junit-testing.md) and add a second
dependency for the Guava library (`com.google.guava:guava:33.0.0-jre`) with
the default `compile` scope, then a third dependency for `logback-classic`
with `runtime` scope. Then write the Gradle (`build.gradle.kts`) equivalent
of that full three-dependency `pom.xml` from scratch, matching each Maven
scope to its Gradle configuration name from the table above.
