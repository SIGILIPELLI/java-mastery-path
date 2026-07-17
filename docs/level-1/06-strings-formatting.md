# 06 · Strings & String Formatting

## String basics

`String` is a reference type, but Java gives it special syntax support
(literals, `+` concatenation). Strings are **immutable** — every "modifying"
method actually returns a new `String`.

```java
String first = "Ada";
String last = "Lovelace";
String full = first + " " + last;

System.out.println(full);            // Ada Lovelace
System.out.println(full.length());    // 12
System.out.println(full.toUpperCase()); // ADA LOVELACE
System.out.println(full.toLowerCase()); // ada lovelace
System.out.println(full.contains("Love")); // true
System.out.println(full.indexOf("Lovelace")); // 4
System.out.println(full.replace("Ada", "Grace")); // Grace Lovelace
System.out.println(full.substring(4));   // Lovelace
System.out.println(full.substring(0, 3)); // Ada
```

Because strings are immutable, `full.toUpperCase()` does not change `full` —
it returns a brand-new `String`. You must capture the result if you want to
keep it:

```java
String name = "ada";
name = name.toUpperCase();   // reassign to keep the change
System.out.println(name);    // ADA
```

## Comparing strings

**Never use `==` to compare string contents** — it compares object references,
not text. Use `.equals()`:

```java
String a = new String("hi");
String b = new String("hi");

System.out.println(a == b);        // false -- different objects!
System.out.println(a.equals(b));   // true  -- same content

System.out.println(a.equalsIgnoreCase("HI"));  // true
```

## Splitting and joining

```java
String csv = "apple,banana,cherry";
String[] fruits = csv.split(",");
System.out.println(fruits[1]);   // banana

String rejoined = String.join(" | ", fruits);
System.out.println(rejoined);    // apple | banana | cherry

String padded = "  hello  ";
System.out.println(padded.trim());   // "hello" -- trims whitespace
```

## StringBuilder — efficient concatenation

Every `+` on strings creates a new object; in a loop that's wasteful.
`StringBuilder` mutates an internal buffer instead:

```java
StringBuilder sb = new StringBuilder();
for (int i = 1; i <= 5; i++) {
    sb.append("Item ").append(i).append("\n");
}
String result = sb.toString();
System.out.print(result);
// Item 1
// Item 2
// Item 3
// Item 4
// Item 5

sb.insert(0, "=== List ===\n");
sb.reverse();  // also supports reverse, delete, replace, etc.
```

## Formatting output

`String.format` and `printf` use format specifiers similar to C's `printf`:

```java
String name = "Ada";
int age = 36;
double gpa = 3.95;

String formatted = String.format("%s is %d years old, GPA %.2f", name, age, gpa);
System.out.println(formatted);
// Ada is 36 years old, GPA 3.95

System.out.printf("%-10s|%5d%n", "Alice", 42);   // left-pad name, right-pad number
System.out.printf("%-10s|%5d%n", "Bob", 7);
// Alice     |   42
// Bob       |    7
```

| Specifier | Meaning |
|-----------|---------|
| `%s` | String |
| `%d` | Integer |
| `%f` | Floating-point (`%.2f` = 2 decimal places) |
| `%n` | Platform-independent newline |
| `%-10s` | Left-justify in a 10-character field |
| `%5d` | Right-justify integer in a 5-character field |

## Text blocks (Java 15+)

For multi-line strings, use triple-quoted text blocks instead of `\n`
concatenation:

```java
String html = """
        <html>
          <body>Hello</body>
        </html>
        """;
System.out.print(html);
```

## Exercise

Write a program that builds a simple receipt: given parallel arrays of item
names (`String[]`) and prices (`double[]`), use a `StringBuilder` to build a
multi-line receipt where each line is formatted with `String.format` so item
names are left-aligned in a 15-character field and prices are right-aligned
with two decimal places, followed by a final "Total" line.
