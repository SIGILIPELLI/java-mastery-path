# 06 · File I/O & NIO

[Level 1's contact book project](../level-1/10-project-contact-book.md)
already used `Files.newBufferedReader`/`newBufferedWriter` to persist data.
This module rounds out classic `java.io` and covers the modern
`java.nio.file` API in more depth.

## Classic `java.io` recap

The original file APIs are stream- and reader/writer-based, wrapped for
efficiency with buffering:

```java
import java.io.*;

// Writing
try (BufferedWriter writer = new BufferedWriter(new FileWriter("notes.txt"))) {
    writer.write("First line");
    writer.newLine();
    writer.write("Second line");
}

// Reading
try (BufferedReader reader = new BufferedReader(new FileReader("notes.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}
// Output:
// First line
// Second line
```

This still works fine, but `java.nio.file` (introduced in Java 7, "NIO.2") is
more convenient for most everyday tasks and is the modern default.

## `Path` and `Paths.get`

A `Path` represents a filesystem location — it doesn't have to exist yet, and
no I/O happens just by creating one.

```java
import java.nio.file.Path;
import java.nio.file.Paths;

Path file = Paths.get("data", "reports", "summary.txt");   // data/reports/summary.txt
Path fromString = Path.of("data/reports/summary.txt");      // Path.of is the newer, preferred form

System.out.println(file.getFileName());   // summary.txt
System.out.println(file.getParent());      // data/reports
System.out.println(file.toAbsolutePath()); // full path from filesystem root
```

## Reading and writing whole files

For small-to-medium text files, `Files` offers one-line read/write helpers —
no manual buffering, no manual `close()`.

```java
import java.nio.file.Files;
import java.util.List;

Path path = Path.of("greeting.txt");

Files.writeString(path, "Hello, NIO!\n");             // write a String directly
Files.write(path, List.of("line one", "line two"));    // write a List<String>, one per line

List<String> lines = Files.readAllLines(path);         // read entire file into a List<String>
String contents = Files.readString(path);              // read entire file into one String

System.out.println(lines);
// [line one, line two]
```

## Checking, creating, copying, deleting

```java
import java.nio.file.*;

Path dir = Path.of("output");
Path source = Path.of("greeting.txt");
Path target = Path.of("output/greeting-copy.txt");

if (!Files.exists(dir)) {
    Files.createDirectories(dir);   // creates all missing parent directories too
}

Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);

System.out.println(Files.exists(target));   // true
System.out.println(Files.size(target));      // size in bytes

Files.delete(target);   // throws NoSuchFileException if it doesn't exist
Files.deleteIfExists(target);   // safe no-op version
```

| Operation | Method |
|-----------|--------|
| Check existence | `Files.exists(path)` / `Files.notExists(path)` |
| Create directory (and parents) | `Files.createDirectories(path)` |
| Copy | `Files.copy(source, target, options...)` |
| Move / rename | `Files.move(source, target, options...)` |
| Delete | `Files.delete(path)` / `Files.deleteIfExists(path)` |
| Read all lines | `Files.readAllLines(path)` |
| Read whole file | `Files.readString(path)` |
| Write lines/string | `Files.write(path, lines)` / `Files.writeString(path, text)` |

## Walking a directory tree

`Files.walk` and `Files.list` return a `Stream<Path>`, so directory traversal
plugs directly into the Streams API from
[Module 4](04-lambdas-streams.md). Both return I/O-backed streams, so they
must be closed — a `try`-with-resources block does that automatically.

```java
import java.nio.file.*;
import java.util.stream.Stream;

// Files.list -- only the immediate children of a directory
try (Stream<Path> entries = Files.list(Path.of("."))) {
    entries.filter(Files::isRegularFile)
           .forEach(System.out::println);
}

// Files.walk -- recurses into subdirectories too
try (Stream<Path> allFiles = Files.walk(Path.of("."))) {
    long javaFileCount = allFiles
        .filter(p -> p.toString().endsWith(".java"))
        .count();
    System.out.println("Java files found: " + javaFileCount);
}
```

## Appending to an existing file

By default `Files.write`/`writeString` overwrite the target. Pass
`StandardOpenOption.APPEND` to add to the end instead — useful for simple log
files:

```java
Files.writeString(
    Path.of("log.txt"),
    "New entry\n",
    StandardOpenOption.CREATE, StandardOpenOption.APPEND
);
```

## Exercise

Write a method `logMessage(String message)` that appends `message` plus a
timestamp-free newline to a file `app.log`, creating the file (and a `logs/`
parent directory) if it doesn't exist yet. Write a second method
`countLogEntries()` that returns how many lines are currently in `app.log`
using `Files.readAllLines`. Call `logMessage` three times with different
strings, then print the result of `countLogEntries()` to confirm it reports
3.
