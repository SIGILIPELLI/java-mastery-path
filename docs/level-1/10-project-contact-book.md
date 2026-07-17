# 10 · Project — CLI Contact Book

A small end-to-end project combining everything from Level 1: classes,
objects, arrays/collections, strings, exception handling, and packages.

## What you'll build

A multi-class command-line contact book that:

- Adds contacts (name, phone, email)
- Lists all contacts
- Finds a contact by name
- Deletes a contact
- Persists everything to a plain-text file between runs

## Project layout

```text
contactbook/
    Contact.java
    ContactStorage.java
    ContactBook.java
```

## Contact.java — the data class

```java
// Contact.java
public class Contact {
    private final String name;
    private final String phone;
    private final String email;

    public Contact(String name, String phone, String email) {
        this.name = name;
        this.phone = phone;
        this.email = email;
    }

    public String getName() { return name; }
    public String getPhone() { return phone; }
    public String getEmail() { return email; }

    // Serialize to a single line for file storage: name,phone,email
    public String toCsvLine() {
        return name + "," + phone + "," + email;
    }

    // Parse a stored line back into a Contact
    public static Contact fromCsvLine(String line) {
        String[] parts = line.split(",", -1);
        if (parts.length != 3) {
            throw new IllegalArgumentException("Malformed contact line: " + line);
        }
        return new Contact(parts[0], parts[1], parts[2]);
    }

    @Override
    public String toString() {
        return String.format("%-15s %-15s %s", name, phone, email);
    }
}
```

## ContactStorage.java — file persistence

```java
// ContactStorage.java
import java.io.*;
import java.nio.file.*;
import java.util.ArrayList;
import java.util.List;

public class ContactStorage {
    private final Path filePath;

    public ContactStorage(String fileName) {
        this.filePath = Paths.get(fileName);
    }

    public List<Contact> load() {
        List<Contact> contacts = new ArrayList<>();
        if (!Files.exists(filePath)) {
            return contacts;   // no file yet -- start empty
        }
        try (BufferedReader reader = Files.newBufferedReader(filePath)) {
            String line;
            while ((line = reader.readLine()) != null) {
                if (!line.isBlank()) {
                    contacts.add(Contact.fromCsvLine(line));
                }
            }
        } catch (IOException e) {
            System.out.println("Warning: could not read contacts file: " + e.getMessage());
        }
        return contacts;
    }

    public void save(List<Contact> contacts) {
        try (BufferedWriter writer = Files.newBufferedWriter(filePath)) {
            for (Contact c : contacts) {
                writer.write(c.toCsvLine());
                writer.newLine();
            }
        } catch (IOException e) {
            System.out.println("Error: could not save contacts: " + e.getMessage());
        }
    }
}
```

## ContactBook.java — CLI logic

```java
// ContactBook.java
import java.util.ArrayList;
import java.util.List;

public class ContactBook {

    static List<Contact> contacts;
    static ContactStorage storage = new ContactStorage("contacts.csv");

    public static void main(String[] args) {
        contacts = storage.load();

        if (args.length == 0) {
            printUsage();
            return;
        }

        String command = args[0];

        try {
            switch (command) {
                case "add" -> addContact(args);
                case "list" -> listContacts();
                case "find" -> findContact(args);
                case "delete" -> deleteContact(args);
                default -> printUsage();
            }
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    static void printUsage() {
        System.out.println("Usage: ContactBook [add <name> <phone> <email> | list | find <name> | delete <name>]");
    }

    static void addContact(String[] args) {
        if (args.length != 4) {
            throw new IllegalArgumentException("add requires: <name> <phone> <email>");
        }
        Contact c = new Contact(args[1], args[2], args[3]);
        contacts.add(c);
        storage.save(contacts);
        System.out.println("Added: " + c);
    }

    static void listContacts() {
        if (contacts.isEmpty()) {
            System.out.println("No contacts yet.");
            return;
        }
        for (Contact c : contacts) {
            System.out.println(c);
        }
    }

    static void findContact(String[] args) {
        if (args.length != 2) {
            throw new IllegalArgumentException("find requires: <name>");
        }
        String query = args[1];
        boolean found = false;
        for (Contact c : contacts) {
            if (c.getName().equalsIgnoreCase(query)) {
                System.out.println(c);
                found = true;
            }
        }
        if (!found) {
            System.out.println("No contact named " + query);
        }
    }

    static void deleteContact(String[] args) {
        if (args.length != 2) {
            throw new IllegalArgumentException("delete requires: <name>");
        }
        String query = args[1];
        List<Contact> remaining = new ArrayList<>();
        boolean removed = false;
        for (Contact c : contacts) {
            if (c.getName().equalsIgnoreCase(query)) {
                removed = true;
            } else {
                remaining.add(c);
            }
        }
        contacts = remaining;
        storage.save(contacts);
        System.out.println(removed ? "Deleted " + query : "No contact named " + query);
    }
}
```

## Running it

```bash
javac Contact.java ContactStorage.java ContactBook.java

java ContactBook add "Alice Smith" 555-1234 alice@example.com
# Added: Alice Smith     555-1234        alice@example.com

java ContactBook add "Bob Jones" 555-5678 bob@example.com
java ContactBook list
# Alice Smith     555-1234        alice@example.com
# Bob Jones       555-5678        bob@example.com

java ContactBook find Alice Smith
java ContactBook delete "Bob Jones"
java ContactBook list
# Alice Smith     555-1234        alice@example.com
```

Each run reloads `contacts.csv` from disk, so contacts persist across separate
invocations of the program — just like the file-backed to-do apps common in
introductory courses, but with proper classes instead of loose data.

## Stretch goals

- Validate email format with a simple check (contains `@` and a `.` after it).
- Add an `update <name> <field> <value>` command to edit an existing contact.
- Switch storage from CSV to JSON (you'll use a real JSON library and modern
  file I/O for this in [Level 2](../level-2/06-file-io-nio.md) — for now,
  plain CSV parsing is fine).
- Add unit tests for `Contact.fromCsvLine` and `toCsvLine` (formalized properly
  with JUnit 5 in [Level 2, Module 7](../level-2/07-junit-testing.md)).

Completing this project means you're ready for **Level 2 · Intermediate**.
