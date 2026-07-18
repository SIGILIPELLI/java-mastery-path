# 05 · Security Best Practices

[Module 1](01-spring-ecosystem-production.md) covered authentication and
authorization. This module covers the layer beneath that: making sure the
data flowing into your application can't be weaponized against it, and that
secrets never end up somewhere an attacker can read them.

## Input validation with Bean Validation

Never trust data from a client. Bean Validation (`jakarta.validation`)
declares constraints directly on a DTO, and Spring enforces them
automatically when the DTO is a `@Valid` controller parameter.

```java
// CreateUserRequest.java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    String name,

    @Email(message = "Must be a valid email address")
    @NotBlank
    String email,

    @Size(min = 8, max = 100, message = "Password must be 8-100 characters")
    String password
) {}
```

```java
// UserController.java
import jakarta.validation.Valid;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping
    public UserResponse createUser(@Valid @RequestBody CreateUserRequest request) {
        // If validation fails, Spring throws MethodArgumentNotValidException
        // before this method body ever runs, and returns 400 Bad Request.
        return userService.create(request);
    }
}
```

A `@RestControllerAdvice` (see [Module 10](10-capstone-project.md)) can turn
the resulting `MethodArgumentNotValidException` into a clean JSON error body
listing every failed field instead of a raw stack trace.

## SQL injection — always use parameter binding

SQL injection happens when untrusted input is concatenated directly into a
query string. Both JDBC's `PreparedStatement` and Spring Data's query
methods bind parameters safely by default — the fix is simply "never build
SQL with string concatenation."

```java
// VULNERABLE -- never do this
String query = "SELECT * FROM users WHERE email = '" + userInput + "'";
// userInput = "' OR '1'='1" turns this into a query that returns every row

// SAFE -- PreparedStatement treats userInput strictly as a value, never as SQL
String sql = "SELECT * FROM users WHERE email = ?";
try (PreparedStatement stmt = connection.prepareStatement(sql)) {
    stmt.setString(1, userInput);
    ResultSet rs = stmt.executeQuery();
}

// SAFE -- Spring Data's @Param binding (see Module 1) is equally safe
@Query("SELECT u FROM User u WHERE u.email = :email")
Optional<User> findByEmail(@Param("email") String email);
```

## Common vulnerabilities and mitigations

| Vulnerability | What it is | Mitigation |
|----------------|------------|------------|
| SQL Injection | Untrusted input executed as SQL | Always use `PreparedStatement` / parameterized JPQL, never string-concatenate queries |
| XSS (Cross-Site Scripting) | Untrusted input rendered as HTML/JS in a browser | Escape output on render; rely on template engines' auto-escaping; set a `Content-Security-Policy` header |
| CSRF (Cross-Site Request Forgery) | A tricked browser submits a request using the victim's session | Use Spring Security's CSRF tokens for cookie-based sessions; not needed for stateless token-based APIs |
| Insecure Deserialization | Deserializing untrusted data executes attacker-controlled code | Avoid Java native serialization for untrusted input; validate/allow-list types when deserializing JSON into polymorphic types |

## Secrets management

Never hardcode credentials, API keys, or tokens in source code — they end up
in git history forever, visible to anyone with repo access.

```properties
# application.properties -- BAD
spring.datasource.password=SuperSecret123!

# application.properties -- GOOD: placeholder resolved from the environment
spring.datasource.password=${DB_PASSWORD}
```

```bash
# The actual secret lives in the environment, not the repo
export DB_PASSWORD='SuperSecret123!'
java -jar app.jar
```

```yaml
# docker-compose.yml -- secrets passed at runtime, not baked into the image
services:
  app:
    image: order-service:latest
    environment:
      DB_PASSWORD: ${DB_PASSWORD}   # pulled from the host's/.env's environment
```

For production systems beyond a single host, a dedicated **secrets
manager** (HashiCorp Vault, AWS Secrets Manager, or a Kubernetes `Secret`
mounted as a file) centralizes storage, rotation, and access-audit logging
for credentials — the application fetches secrets at startup rather than
having them baked into any config file at all.

## OWASP Top 10

The [OWASP Top 10](https://owasp.org/www-project-top-ten/) is the industry
reference list of the most critical web application security risks —
injection, broken authentication, sensitive data exposure, security
misconfiguration, and others covered above among them. Treat it as a
checklist to review against before shipping anything that accepts external
input.

## Exercise

Add Bean Validation annotations to a `RegisterDeviceRequest` record with
fields `deviceId` (must not be blank), `ownerEmail` (must be a valid email),
and `pin` (must be exactly 4-6 characters). Wire it into a
`@PostMapping("/devices")` controller method using `@Valid`. Then rewrite an
imaginary vulnerable line `"SELECT * FROM devices WHERE id = '" + id + "'"`
as a safe parameterized query using either `PreparedStatement` or a Spring
Data `@Query`.
