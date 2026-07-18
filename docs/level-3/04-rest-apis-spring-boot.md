# 04 · Building REST APIs with Spring Boot

Spring Boot is a framework for building Java applications — especially web
services — with minimal manual wiring. Its core idea is **inversion of
control**: instead of your code creating and wiring up its own dependencies,
the Spring container creates the objects (called "beans") and injects them
where needed, based on annotations.

## The `pom.xml` dependency

```xml
<!-- pom.xml -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

`spring-boot-starter-web` pulls in everything needed for a web app: an
embedded Tomcat server, Spring MVC, and JSON support (Jackson) — no separate
application server install required.

## The application entry point

```java
// src/main/java/com/example/api/ApiApplication.java
package com.example.api;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication   // enables auto-configuration, component scanning, and more
public class ApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiApplication.class, args);
    }
}
```

Running this starts an embedded web server (port 8080 by default) and scans
the package for Spring-managed components.

## A DTO returned as JSON

```java
// src/main/java/com/example/api/Greeting.java
package com.example.api;

public record Greeting(String message, int id) {
}
```

Records work perfectly as response bodies — Spring uses Jackson to serialize
them to JSON automatically.

## A `@Service` with constructor injection

```java
// src/main/java/com/example/api/GreetingService.java
package com.example.api;

import org.springframework.stereotype.Service;
import java.util.concurrent.atomic.AtomicInteger;

@Service   // marks this as a Spring-managed bean, eligible for injection
public class GreetingService {
    private final AtomicInteger counter = new AtomicInteger();

    public Greeting greet(String name) {
        return new Greeting("Hello, " + name + "!", counter.incrementAndGet());
    }
}
```

## A `@RestController`

```java
// src/main/java/com/example/api/GreetingController.java
package com.example.api;

import org.springframework.web.bind.annotation.*;

@RestController              // combines @Controller + @ResponseBody -- return values become JSON
@RequestMapping("/api")
public class GreetingController {

    private final GreetingService greetingService;

    // Constructor injection -- Spring supplies a GreetingService automatically.
    // @Autowired is optional here: a single constructor is auto-wired by default.
    public GreetingController(GreetingService greetingService) {
        this.greetingService = greetingService;
    }

    @GetMapping("/greet/{name}")
    public Greeting greet(@PathVariable String name) {
        return greetingService.greet(name);
    }

    @PostMapping("/greet")
    public Greeting greetFromBody(@RequestBody GreetingRequest request) {
        return greetingService.greet(request.name());
    }
}
```

```java
// src/main/java/com/example/api/GreetingRequest.java
package com.example.api;

public record GreetingRequest(String name) {
}
```

Constructor injection (rather than field injection with `@Autowired` on a
field) is preferred — it makes dependencies explicit, works with `final`
fields, and is easy to test by just calling the constructor directly.

## Running it

```bash
mvn spring-boot:run
```

```bash
curl http://localhost:8080/api/greet/Alice
# {"message":"Hello, Alice!","id":1}

curl -X POST http://localhost:8080/api/greet \
     -H "Content-Type: application/json" \
     -d '{"name": "Bob"}'
# {"message":"Hello, Bob!","id":2}
```

## Key annotations at a glance

| Annotation | Purpose |
|------------|---------|
| `@SpringBootApplication` | Marks the main class; enables auto-configuration |
| `@RestController` | Marks a class whose methods return JSON/text responses |
| `@Service` | Marks a class as a business-logic bean |
| `@GetMapping` / `@PostMapping` | Maps an HTTP GET/POST to a method |
| `@PathVariable` | Binds a `{placeholder}` from the URL to a method parameter |
| `@RequestBody` | Deserializes the JSON request body into a Java object |
| `@Autowired` | Requests dependency injection (optional on a single constructor) |

Spring resolves the `GreetingController -> GreetingService` dependency
automatically at startup because both classes are annotated as beans
(`@RestController` and `@Service` are both specializations of the general
`@Component` stereotype) — you never write `new GreetingService()` yourself.
Persisting data behind these endpoints with Spring Data JPA is covered in the
[capstone project](11-project-rest-api-db.md).

## Exercise

Add a `@RestController` with a `GET /api/square/{n}` endpoint that returns a
JSON object `{ "input": n, "result": n*n }` (define a small record for the
response), and a `POST /api/square` endpoint that accepts `{ "n": 5 }` in the
request body and returns the same shape. Test both endpoints with `curl`.
