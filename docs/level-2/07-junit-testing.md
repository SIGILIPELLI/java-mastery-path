# 07 · Unit Testing with JUnit 5

Every example so far has been verified by eyeballing printed output. JUnit 5
lets you write automated checks that run in milliseconds and fail loudly the
moment behavior changes unexpectedly.

## Adding JUnit 5 to a Maven project

[Level 1, Module 9](../level-1/09-packages-build-tools.md) showed the
`junit-jupiter` dependency briefly — here it is in context, in a full
`pom.xml`:

```xml
<!-- pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>calculator-app</artifactId>
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

Test classes live in `src/test/java`, mirroring the package structure of
`src/main/java`, as shown in Level 1's standard Maven layout.

## The class under test

```java
// src/main/java/com/example/Calculator.java
package com.example;

public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int divide(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return a / b;
    }
}
```

## `@Test` and basic assertions

```java
// src/test/java/com/example/CalculatorTest.java
package com.example;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    @Test
    void addsTwoNumbers() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertEquals(5, result);   // expected, actual
    }

    @Test
    void divideThrowsOnZero() {
        Calculator calc = new Calculator();
        assertThrows(ArithmeticException.class, () -> calc.divide(10, 0));
    }

    @Test
    void addIsCommutative() {
        Calculator calc = new Calculator();
        assertTrue(calc.add(2, 3) == calc.add(3, 2));
    }
}
```

| Assertion | Checks |
|-----------|--------|
| `assertEquals(expected, actual)` | Values are equal |
| `assertTrue(condition)` / `assertFalse(condition)` | Boolean condition |
| `assertThrows(Type.class, executable)` | Code throws the given exception type |
| `assertNull(obj)` / `assertNotNull(obj)` | Reference is/isn't null |
| `assertArrayEquals(expected, actual)` | Arrays have equal contents |

## `@BeforeEach` and `@AfterEach`

These run before/after **every** `@Test` method in the class — ideal for
setting up (and tearing down) a fresh object so tests don't leak state
between each other.

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.AfterEach;

class CalculatorTest {
    private Calculator calc;

    @BeforeEach
    void setUp() {
        calc = new Calculator();   // fresh instance before each test
        System.out.println("Setting up...");
    }

    @AfterEach
    void tearDown() {
        System.out.println("Tearing down...");
    }

    @Test
    void addsTwoNumbers() {
        assertEquals(5, calc.add(2, 3));
    }
}
```

## `@ParameterizedTest`

Run the same test logic against several inputs without copy-pasting the test
method.

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

class CalculatorParamTest {

    @ParameterizedTest
    @ValueSource(ints = {2, 4, 6, 8, 100})
    void addingZeroReturnsSameNumber(int n) {
        Calculator calc = new Calculator();
        assertEquals(n, calc.add(n, 0));
    }
}
```

`@ValueSource` also accepts `strings = {...}`, `doubles = {...}`, `booleans =
{...}`, etc. — the annotated test method runs once per value supplied.

## Testing a real class: `Contact` from Level 1

Here's a fuller example testing the `Contact` class from
[Level 1's capstone project](../level-1/10-project-contact-book.md):

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class ContactTest {

    @Test
    void toCsvLineFormatsFieldsCorrectly() {
        Contact c = new Contact("Alice", "555-1234", "alice@example.com");
        assertEquals("Alice,555-1234,alice@example.com", c.toCsvLine());
    }

    @Test
    void fromCsvLineParsesBackIntoAContact() {
        Contact c = Contact.fromCsvLine("Bob,555-5678,bob@example.com");
        assertEquals("Bob", c.getName());
        assertEquals("555-5678", c.getPhone());
        assertEquals("bob@example.com", c.getEmail());
    }

    @Test
    void fromCsvLineRejectsMalformedInput() {
        assertThrows(IllegalArgumentException.class,
            () -> Contact.fromCsvLine("Not,Enough"));
    }
}
```

## Running the tests

```bash
mvn test
```

```text
[INFO] Running com.example.CalculatorTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
[INFO] Running com.example.ContactTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] BUILD SUCCESS
```

`mvn test` compiles both `src/main/java` and `src/test/java`, then runs every
class matching JUnit's naming/annotation conventions and reports a summary —
the same `mvn` lifecycle introduced in
[Level 1, Module 9](../level-1/09-packages-build-tools.md) and expanded on in
[Module 8](08-build-tools.md).

## Exercise

Write a `StringUtils` class with a static method `isPalindrome(String s)`
that returns `true` if `s` reads the same forwards and backwards (case-
insensitive). Write a `StringUtilsTest` class with: one `@Test` for a
positive case, one for a negative case, one `@Test` using `assertThrows` to
confirm it throws `NullPointerException` (or handles null however you decide
— just test that behavior), and one `@ParameterizedTest` with `@ValueSource`
covering at least four different palindrome strings.
