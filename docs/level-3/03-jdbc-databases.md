# 03 · JDBC & Databases

JDBC (Java Database Connectivity) is the standard API for connecting to
relational databases from Java. The same code works against different
databases (H2, SQLite, PostgreSQL, MySQL...) as long as the right driver is on
the classpath.

## Adding a JDBC driver dependency

For learning and testing, **H2** is a great choice — a pure-Java database that
can run entirely in memory, no separate server install needed:

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>2.2.224</version>
    </dependency>
</dependencies>
```

(SQLite works the same way via the `org.xerial:sqlite-jdbc` artifact — only
the connection URL and driver name differ.)

## Connecting with `DriverManager`

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectDemo {
    public static void main(String[] args) throws SQLException {
        // in-memory H2 database named "testdb"; DB_CLOSE_DELAY=-1 keeps
        // it alive for the whole JVM run instead of vanishing after each connection
        String url = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";

        try (Connection conn = DriverManager.getConnection(url, "sa", "")) {
            System.out.println("Connected: " + !conn.isClosed());
        }
    }
}
// Output:
// Connected: true
```

`Connection`, `Statement`, and `ResultSet` all implement `AutoCloseable`, so
try-with-resources ensures they're released even if an exception occurs —
critical for not leaking database connections.

## Creating a table

```java
import java.sql.*;

public class CreateTableDemo {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";

        try (Connection conn = DriverManager.getConnection(url, "sa", "");
             Statement stmt = conn.createStatement()) {

            stmt.execute("""
                CREATE TABLE employees (
                    id INT AUTO_INCREMENT PRIMARY KEY,
                    name VARCHAR(100) NOT NULL,
                    salary DECIMAL(10, 2)
                )
            """);
            System.out.println("Table created");
        }
    }
}
```

## `PreparedStatement` — parameterized queries

Never build SQL by concatenating raw strings with user input — it invites
**SQL injection**. `PreparedStatement` separates the query structure from its
parameters, which the driver escapes safely:

```java
String sql = "INSERT INTO employees (name, salary) VALUES (?, ?)";

try (PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, "Alice");
    ps.setBigDecimal(2, new java.math.BigDecimal("75000.00"));
    ps.executeUpdate();
}

// NEVER do this -- vulnerable to SQL injection:
// stmt.execute("INSERT INTO employees (name) VALUES ('" + userInput + "')");
```

If `userInput` were `x'); DROP TABLE employees; --`, string concatenation
would let an attacker run arbitrary SQL. A `PreparedStatement` treats `?`
placeholders strictly as data, never as SQL syntax.

## Full CRUD example

```java
import java.math.BigDecimal;
import java.sql.*;

public class EmployeeCrudDemo {
    static final String URL = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";

    public static void main(String[] args) throws SQLException {
        try (Connection conn = DriverManager.getConnection(URL, "sa", "")) {
            createTable(conn);
            insert(conn, "Alice", new BigDecimal("75000.00"));
            insert(conn, "Bob", new BigDecimal("68000.00"));

            System.out.println("-- After insert --");
            findAll(conn);

            update(conn, "Alice", new BigDecimal("80000.00"));
            System.out.println("-- After update --");
            findAll(conn);

            delete(conn, "Bob");
            System.out.println("-- After delete --");
            findAll(conn);
        }
    }

    static void createTable(Connection conn) throws SQLException {
        try (Statement stmt = conn.createStatement()) {
            stmt.execute("""
                CREATE TABLE employees (
                    id INT AUTO_INCREMENT PRIMARY KEY,
                    name VARCHAR(100) NOT NULL,
                    salary DECIMAL(10, 2)
                )
            """);
        }
    }

    static void insert(Connection conn, String name, BigDecimal salary) throws SQLException {
        String sql = "INSERT INTO employees (name, salary) VALUES (?, ?)";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, name);
            ps.setBigDecimal(2, salary);
            ps.executeUpdate();
        }
    }

    static void findAll(Connection conn) throws SQLException {
        String sql = "SELECT id, name, salary FROM employees ORDER BY id";
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            while (rs.next()) {
                System.out.printf("#%d %-10s $%s%n",
                        rs.getInt("id"), rs.getString("name"), rs.getBigDecimal("salary"));
            }
        }
    }

    static void update(Connection conn, String name, BigDecimal newSalary) throws SQLException {
        String sql = "UPDATE employees SET salary = ? WHERE name = ?";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setBigDecimal(1, newSalary);
            ps.setString(2, name);
            int rows = ps.executeUpdate();
            System.out.println(rows + " row(s) updated");
        }
    }

    static void delete(Connection conn, String name) throws SQLException {
        String sql = "DELETE FROM employees WHERE name = ?";
        try (PreparedStatement ps = conn.prepareStatement(sql)) {
            ps.setString(1, name);
            int rows = ps.executeUpdate();
            System.out.println(rows + " row(s) deleted");
        }
    }
}
// Output:
// -- After insert --
// #1 Alice      $75000.00
// #2 Bob        $68000.00
// 1 row(s) updated
// -- After update --
// #1 Alice      $80000.00
// #2 Bob        $68000.00
// 1 row(s) deleted
// -- After delete --
// #1 Alice      $80000.00
```

## Reading results with `ResultSet`

`ResultSet` is a cursor over the rows returned by a query — `next()` advances
it one row at a time and returns `false` once exhausted. Column values are
read by name (`rs.getString("name")`) or 1-based index (`rs.getString(1)`).

| JDBC type | Purpose |
|-----------|---------|
| `DriverManager.getConnection(url, user, pass)` | Opens a connection to the database |
| `Statement` | Runs a fixed SQL string (no parameters) |
| `PreparedStatement` | Runs parameterized SQL safely, reusably |
| `ResultSet` | Cursor over the rows returned by a query |
| `executeUpdate()` | Runs INSERT/UPDATE/DELETE, returns rows affected |
| `executeQuery()` | Runs SELECT, returns a `ResultSet` |

Real applications typically don't call JDBC directly for every query — an ORM
like Spring Data JPA (used in [Level 3, Module 4](04-rest-apis-spring-boot.md)
and the [capstone project](11-project-rest-api-db.md)) builds on JDBC under
the hood but lets you work with objects instead of raw SQL.

## Exercise

Using an in-memory H2 database, create a `books` table with columns `id`
(auto-increment primary key), `title`, and `year`. Write methods `addBook`,
`listBooks`, and `deleteBookByTitle`, all using `PreparedStatement`. In `main`,
insert three books, list them, delete one by title, and list them again to
confirm the deletion.
