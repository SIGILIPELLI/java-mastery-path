# 10 · Project — Inventory Management System

The Level 2 capstone pulls together everything from this level: OOP design,
the Collections Framework, generics-backed collections, streams, custom
exceptions, `java.nio.file` persistence, and JUnit 5 tests — combined into one
working command-line application, in the same spirit as
[Level 1's contact book](../level-1/10-project-contact-book.md).

## What you'll build

A multi-class CLI Inventory Management System that:

- Adds products (id, name, quantity, price)
- Lists all products
- Updates a product's quantity
- Removes a product
- Finds a product by id
- Persists everything to a CSV file between runs, via `java.nio.file`
- Ships with a JUnit 5 test class covering the core logic

## Project layout

```text
inventory-system/
    pom.xml
    src/
        main/
            java/
                com/example/inventory/Product.java
                com/example/inventory/InventoryStorage.java
                com/example/inventory/InventoryManager.java
        test/
            java/
                com/example/inventory/InventoryStorageTest.java
```

## pom.xml

```xml
<!-- pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>inventory-system</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>
        </plugins>
    </build>
</project>
```

## Product.java — the data model

A `record` is a natural fit here: a product is a plain, immutable bundle of
data (see [Module 9](09-enums-records-sealed.md)). Quantity updates are
modeled by creating a new `Product` rather than mutating one.

```java
// Product.java
package com.example.inventory;

public record Product(String id, String name, int quantity, double price) {

    public Product {
        if (id == null || id.isBlank()) {
            throw new IllegalArgumentException("Product id cannot be blank");
        }
        if (quantity < 0) {
            throw new IllegalArgumentException("Quantity cannot be negative: " + quantity);
        }
        if (price < 0) {
            throw new IllegalArgumentException("Price cannot be negative: " + price);
        }
    }

    // Returns a copy of this product with a different quantity
    public Product withQuantity(int newQuantity) {
        return new Product(id, name, newQuantity, price);
    }

    // Serialize to a single CSV line: id,name,quantity,price
    public String toCsvLine() {
        return id + "," + name + "," + quantity + "," + price;
    }

    // Parse a stored line back into a Product
    public static Product fromCsvLine(String line) {
        String[] parts = line.split(",", -1);
        if (parts.length != 4) {
            throw new IllegalArgumentException("Malformed product line: " + line);
        }
        return new Product(parts[0], parts[1], Integer.parseInt(parts[2]), Double.parseDouble(parts[3]));
    }

    @Override
    public String toString() {
        return String.format("%-8s %-15s qty=%-5d $%.2f", id, name, quantity, price);
    }
}
```

## InventoryStorage.java — file persistence

```java
// InventoryStorage.java
package com.example.inventory;

import java.io.IOException;
import java.nio.file.*;
import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

public class InventoryStorage {
    private final Path filePath;

    public InventoryStorage(String fileName) {
        this.filePath = Paths.get(fileName);
    }

    // LinkedHashMap keeps insertion order, keyed by product id for O(1) lookup
    public Map<String, Product> load() {
        Map<String, Product> products = new LinkedHashMap<>();
        if (!Files.exists(filePath)) {
            return products;   // no file yet -- start empty
        }
        try {
            List<String> lines = Files.readAllLines(filePath);
            for (String line : lines) {
                if (!line.isBlank()) {
                    Product p = Product.fromCsvLine(line);
                    products.put(p.id(), p);
                }
            }
        } catch (IOException e) {
            System.out.println("Warning: could not read inventory file: " + e.getMessage());
        }
        return products;
    }

    public void save(Map<String, Product> products) {
        List<String> lines = new ArrayList<>();
        for (Product p : products.values()) {
            lines.add(p.toCsvLine());
        }
        try {
            if (filePath.getParent() != null) {
                Files.createDirectories(filePath.getParent());
            }
            Files.write(filePath, lines);
        } catch (IOException e) {
            System.out.println("Error: could not save inventory: " + e.getMessage());
        }
    }
}
```

## InventoryManager.java — CLI logic

```java
// InventoryManager.java
package com.example.inventory;

import java.util.LinkedHashMap;
import java.util.Map;

public class InventoryManager {

    static Map<String, Product> products;
    static InventoryStorage storage = new InventoryStorage("inventory.csv");

    public static void main(String[] args) {
        products = storage.load();

        if (args.length == 0) {
            printUsage();
            return;
        }

        String command = args[0];

        try {
            switch (command) {
                case "add" -> addProduct(args);
                case "list" -> listProducts();
                case "update-quantity" -> updateQuantity(args);
                case "remove" -> removeProduct(args);
                case "find" -> findProduct(args);
                default -> printUsage();
            }
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    static void printUsage() {
        System.out.println("Usage: InventoryManager [add <id> <name> <qty> <price> | " +
            "list | update-quantity <id> <qty> | remove <id> | find <id>]");
    }

    static void addProduct(String[] args) {
        if (args.length != 5) {
            throw new IllegalArgumentException("add requires: <id> <name> <qty> <price>");
        }
        Product p = new Product(args[1], args[2], Integer.parseInt(args[3]), Double.parseDouble(args[4]));
        products.put(p.id(), p);
        storage.save(products);
        System.out.println("Added: " + p);
    }

    static void listProducts() {
        if (products.isEmpty()) {
            System.out.println("No products yet.");
            return;
        }
        for (Product p : products.values()) {
            System.out.println(p);
        }
    }

    static void updateQuantity(String[] args) {
        if (args.length != 3) {
            throw new IllegalArgumentException("update-quantity requires: <id> <qty>");
        }
        String id = args[1];
        int newQty = Integer.parseInt(args[2]);
        Product existing = products.get(id);
        if (existing == null) {
            throw new IllegalArgumentException("No product with id " + id);
        }
        Product updated = existing.withQuantity(newQty);
        products.put(id, updated);
        storage.save(products);
        System.out.println("Updated: " + updated);
    }

    static void removeProduct(String[] args) {
        if (args.length != 2) {
            throw new IllegalArgumentException("remove requires: <id>");
        }
        String id = args[1];
        Product removed = products.remove(id);
        storage.save(products);
        System.out.println(removed != null ? "Removed " + id : "No product with id " + id);
    }

    static void findProduct(String[] args) {
        if (args.length != 2) {
            throw new IllegalArgumentException("find requires: <id>");
        }
        Product p = products.get(args[1]);
        System.out.println(p != null ? p : "No product with id " + args[1]);
    }
}
```

## InventoryStorageTest.java — JUnit 5 tests

```java
// InventoryStorageTest.java
package com.example.inventory;

import org.junit.jupiter.api.Test;
import java.util.Map;
import static org.junit.jupiter.api.Assertions.*;

class InventoryStorageTest {

    @Test
    void productRoundTripsThroughCsv() {
        Product original = new Product("P1", "Widget", 10, 4.99);
        Product parsed = Product.fromCsvLine(original.toCsvLine());

        assertEquals(original.id(), parsed.id());
        assertEquals(original.name(), parsed.name());
        assertEquals(original.quantity(), parsed.quantity());
        assertEquals(original.price(), parsed.price(), 0.0001);
    }

    @Test
    void withQuantityReturnsNewProductWithoutMutatingOriginal() {
        Product original = new Product("P2", "Gadget", 5, 9.99);
        Product updated = original.withQuantity(20);

        assertEquals(5, original.quantity());   // original untouched -- records are immutable
        assertEquals(20, updated.quantity());
        assertEquals(original.id(), updated.id());
    }

    @Test
    void constructorRejectsNegativeQuantity() {
        assertThrows(IllegalArgumentException.class,
            () -> new Product("P3", "Broken", -1, 1.0));
    }

    @Test
    void saveAndLoadRoundTripThroughATempFile(@org.junit.jupiter.api.io.TempDir java.nio.file.Path tempDir) {
        InventoryStorage storage = new InventoryStorage(tempDir.resolve("inventory.csv").toString());
        Map<String, Product> original = new java.util.LinkedHashMap<>();
        original.put("P1", new Product("P1", "Widget", 10, 4.99));
        original.put("P2", new Product("P2", "Gadget", 5, 9.99));

        storage.save(original);
        Map<String, Product> loaded = storage.load();

        assertEquals(2, loaded.size());
        assertEquals("Widget", loaded.get("P1").name());
    }
}
```

## Running it

```bash
mvn compile
mvn exec:java -Dexec.mainClass="com.example.inventory.InventoryManager" -Dexec.args="add P1 Widget 10 4.99"
# Added: P1       Widget          qty=10    $4.99

mvn exec:java -Dexec.mainClass="com.example.inventory.InventoryManager" -Dexec.args="add P2 Gadget 5 9.99"
mvn exec:java -Dexec.mainClass="com.example.inventory.InventoryManager" -Dexec.args="list"
# P1       Widget          qty=10    $4.99
# P2       Gadget          qty=5     $9.99

mvn exec:java -Dexec.mainClass="com.example.inventory.InventoryManager" -Dexec.args="update-quantity P2 20"
# Updated: P2       Gadget          qty=20    $9.99

mvn test
# [INFO] Running com.example.inventory.InventoryStorageTest
# [INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
# [INFO] BUILD SUCCESS
```

(Running via plain `javac`/`java` also works, the same way as
[Level 1's contact book](../level-1/10-project-contact-book.md) — just
compile everything under `src/main/java` with `javac -d out ...` and run with
`java -cp out com.example.inventory.InventoryManager`.)

Each invocation reloads `inventory.csv` from disk via
[`java.nio.file`](06-file-io-nio.md), so the inventory persists across
separate runs of the program, and the accompanying test class exercises the
core `Product` and `InventoryStorage` logic the way
[Module 7](07-junit-testing.md) taught, independent of the CLI wiring.

## Stretch goals

- Add a `restock <id> <amount>` command that increases quantity relative to
  the current value instead of replacing it outright.
- Add validation that rejects `add`ing a duplicate id, using a custom
  checked exception (`DuplicateProductException`, tying back to
  [Module 5](05-exceptions-advanced.md)) instead of silently overwriting.
- Add a `low-stock <threshold>` command that streams the products
  (`Module 4`) to print only those with `quantity` below the given threshold.
- Switch storage from CSV to a small in-memory `TreeMap` sorted by product
  name for the `list` command's display order, without changing the on-disk
  format (`Module 2`).
- Expand `InventoryStorageTest` with a `@ParameterizedTest` covering several
  malformed CSV lines and asserting each throws `IllegalArgumentException`.

Completing this project means you're ready for **Level 3 · Advanced**.
