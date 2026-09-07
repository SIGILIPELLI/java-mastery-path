# 01 · Setup & First Program

## Install the JDK

Java programs run on the Java Virtual Machine (JVM), and you write them against
the Java Development Kit (JDK) — the compiler, runtime, and standard library
bundled together. Install a recent LTS release (Java 21):

```bash
# macOS (Homebrew)
brew install openjdk@21

# Ubuntu/Debian
sudo apt install openjdk-21-jdk

# Windows: use the installer from https://adoptium.net (Eclipse Temurin builds)
```

Verify the install:

```bash
java --version
# openjdk 21.0.x

javac --version
# javac 21.0.x
```

`java` runs compiled programs; `javac` compiles source (`.java`) files into
bytecode (`.class` files).

## Compiling and running by hand

Create `Hello.java`. In Java, the **public class name must match the file
name** exactly (case-sensitive):

```java
// Hello.java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

Compile, then run:

```bash
javac Hello.java
# produces Hello.class (JVM bytecode)

java Hello
# Hello, world!
```

`public static void main(String[] args)` is the entry point the JVM looks for:
`public` so the JVM (outside the class) can call it, `static` so it can be
called without creating an instance first, `void` because it returns nothing,
and `String[] args` holds command-line arguments.

## Single-file source launch (quick scripting)

Since Java 11 you can skip the separate compile step for quick experiments:

```bash
java Hello.java
# Hello, world!
```

This compiles in memory and runs immediately — handy for small scripts, but
real projects still compile explicitly (usually via a build tool, see
[Module 9](09-packages-build-tools.md)).

## Anatomy of the program

| Piece | Meaning |
|-------|---------|
| `public class Hello` | Declares a class named `Hello` — everything in Java lives inside a class. |
| `public static void main(String[] args)` | The program's entry point. |
| `System.out.println(...)` | Prints text followed by a newline to standard output. |
| `;` | Every statement ends with a semicolon. |
| `{ }` | Curly braces delimit blocks — class bodies, method bodies, loop bodies. |

## Choosing an IDE

Any of these work well for this program: **IntelliJ IDEA Community** (free,
excellent Java-specific tooling and refactoring), **VS Code** with the
"Extension Pack for Java" (free, lightweight), or Eclipse. IntelliJ is the most
common choice for serious Java work — pick one and move on, since the editor
matters far less than practice.

## How It Actually Works

`javac` doesn't produce machine code — it produces **bytecode**, a compact
stack-based instruction set stored in a `.class` file as a sequence of
constant-pool entries, method descriptors, and opcodes (`invokestatic`,
`getstatic`, `areturn`, and so on). When you run `java Hello`, three things
happen before your first line executes:

1. **Class loading** — the bootstrap/platform/application class loaders (a
   parent-delegating hierarchy) locate `Hello.class` on the classpath, read
   the class file, and build an internal `Klass` metadata structure in the
   JVM's method area (part of metaspace, native memory since Java 8).
2. **Linking** — *verification* walks every method's bytecode with a
   dataflow analyzer to prove type-safety (no stack underflow, no jumping
   into the middle of an instruction, no assigning an `Object` where an
   `int` is expected) without executing anything; *preparation* zeroes
   static fields; *resolution* lazily turns symbolic constant-pool
   references into direct pointers the first time they're used.
3. **Initialization** — runs `<clinit>`, the synthesized static
   initializer, exactly once, under a per-class lock, guaranteeing every
   other thread that later touches the class sees a fully initialized one.

`public static void main(String[] args)` is not a convention the JVM
merely likes — the launcher reflectively looks up a method with that exact
signature and modifiers via `Class.getMethod("main", String[].class)` and
invokes it with `invokestatic`; get the signature wrong and you get
`NoSuchMethodError`, not a compiler error, because nothing stops you from
compiling a class without a `main` at all.

## Exercise

Write a program `Greeter.java` with a `main` method that prints a greeting for
three different names, one per line, using `System.out.println`. Then compile
and run it from the command line with `javac` and `java`.
