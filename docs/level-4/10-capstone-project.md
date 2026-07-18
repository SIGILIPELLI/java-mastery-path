# 10 · Capstone Project

The capstone pulls together everything from every module of Level 4: a
layered Spring Boot service backed by a real database, validated input, a
global error handler, unit and integration tests, a container image, and a
CI pipeline that runs the tests automatically. This is the shape of a real,
small production microservice.

## What you'll build

An **Order Management Service** — a REST API for creating, retrieving, and
cancelling orders — with:

- `@RestController` → `@Service` → `@Repository` layering
- An `@Entity` with Bean Validation on its DTOs
- A global exception handler (`@RestControllerAdvice`)
- A Mockito unit test for the service layer
- A `@SpringBootTest` + `MockMvc` integration test
- A multi-stage `Dockerfile` and a `docker-compose.yml` (app + Postgres)
- A GitHub Actions CI workflow
- A Spring Boot Actuator health endpoint

## Project layout

```text
order-management/
    pom.xml
    Dockerfile
    docker-compose.yml
    .github/
        workflows/
            ci.yml
    src/
        main/
            java/
                com/example/orders/
                    OrderManagementApplication.java
                    model/
                        Order.java
                        OrderStatus.java
                    dto/
                        CreateOrderRequest.java
                        OrderResponse.java
                    repository/
                        OrderRepository.java
                    service/
                        OrderService.java
                    controller/
                        OrderController.java
                    exception/
                        OrderNotFoundException.java
                        InvalidOrderStateException.java
                        GlobalExceptionHandler.java
            resources/
                application.properties
        test/
            java/
                com/example/orders/
                    service/
                        OrderServiceTest.java
                    controller/
                        OrderControllerIT.java
```

## pom.xml

```xml
<!-- pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>order-management</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## OrderManagementApplication.java — entry point

```java
// src/main/java/com/example/orders/OrderManagementApplication.java
package com.example.orders;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderManagementApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderManagementApplication.class, args);
    }
}
```

## model/OrderStatus.java — a sealed-friendly status enum

```java
// src/main/java/com/example/orders/model/OrderStatus.java
package com.example.orders.model;

public enum OrderStatus {
    PENDING,
    CONFIRMED,
    CANCELLED
}
```

## model/Order.java — the JPA entity

```java
// src/main/java/com/example/orders/model/Order.java
package com.example.orders.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Positive;

import java.time.Instant;

@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    private String customerEmail;

    @Positive
    private double total;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    private Instant createdAt;

    protected Order() {
        // required by JPA
    }

    public Order(String customerEmail, double total) {
        this.customerEmail = customerEmail;
        this.total = total;
        this.status = OrderStatus.PENDING;
        this.createdAt = Instant.now();
    }

    public Long getId() { return id; }
    public String getCustomerEmail() { return customerEmail; }
    public double getTotal() { return total; }
    public OrderStatus getStatus() { return status; }
    public Instant getCreatedAt() { return createdAt; }

    public void confirm() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Only PENDING orders can be confirmed");
        }
        this.status = OrderStatus.CONFIRMED;
    }

    public void cancel() {
        if (status == OrderStatus.CANCELLED) {
            throw new IllegalStateException("Order is already cancelled");
        }
        this.status = OrderStatus.CANCELLED;
    }
}
```

## dto/CreateOrderRequest.java and OrderResponse.java

```java
// src/main/java/com/example/orders/dto/CreateOrderRequest.java
package com.example.orders.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Positive;

public record CreateOrderRequest(
    @NotBlank @Email String customerEmail,
    @Positive double total
) {}
```

```java
// src/main/java/com/example/orders/dto/OrderResponse.java
package com.example.orders.dto;

import com.example.orders.model.Order;
import com.example.orders.model.OrderStatus;

import java.time.Instant;

public record OrderResponse(
    Long id,
    String customerEmail,
    double total,
    OrderStatus status,
    Instant createdAt
) {
    public static OrderResponse from(Order order) {
        return new OrderResponse(
            order.getId(),
            order.getCustomerEmail(),
            order.getTotal(),
            order.getStatus(),
            order.getCreatedAt()
        );
    }
}
```

## repository/OrderRepository.java

```java
// src/main/java/com/example/orders/repository/OrderRepository.java
package com.example.orders.repository;

import com.example.orders.model.Order;
import com.example.orders.model.OrderStatus;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerEmailIgnoreCase(String customerEmail);
    List<Order> findByStatus(OrderStatus status);
}
```

## exception classes and the global handler

```java
// src/main/java/com/example/orders/exception/OrderNotFoundException.java
package com.example.orders.exception;

public class OrderNotFoundException extends RuntimeException {
    public OrderNotFoundException(Long id) {
        super("Order not found: " + id);
    }
}
```

```java
// src/main/java/com/example/orders/exception/InvalidOrderStateException.java
package com.example.orders.exception;

public class InvalidOrderStateException extends RuntimeException {
    public InvalidOrderStateException(String message) {
        super(message);
    }
}
```

```java
// src/main/java/com/example/orders/exception/GlobalExceptionHandler.java
package com.example.orders.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.Instant;
import java.util.LinkedHashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<Map<String, Object>> handleNotFound(OrderNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorBody(e.getMessage()));
    }

    @ExceptionHandler(InvalidOrderStateException.class)
    public ResponseEntity<Map<String, Object>> handleInvalidState(InvalidOrderStateException e) {
        return ResponseEntity.status(HttpStatus.CONFLICT).body(errorBody(e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidation(MethodArgumentNotValidException e) {
        String details = e.getBindingResult().getFieldErrors().stream()
            .map(fe -> fe.getField() + ": " + fe.getDefaultMessage())
            .reduce((a, b) -> a + "; " + b)
            .orElse("Invalid request");
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errorBody(details));
    }

    private Map<String, Object> errorBody(String message) {
        Map<String, Object> body = new LinkedHashMap<>();
        body.put("timestamp", Instant.now());
        body.put("message", message);
        return body;
    }
}
```

## service/OrderService.java

```java
// src/main/java/com/example/orders/service/OrderService.java
package com.example.orders.service;

import com.example.orders.dto.CreateOrderRequest;
import com.example.orders.exception.InvalidOrderStateException;
import com.example.orders.exception.OrderNotFoundException;
import com.example.orders.model.Order;
import com.example.orders.repository.OrderRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class OrderService {

    private static final Logger log = LoggerFactory.getLogger(OrderService.class);

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public Order create(CreateOrderRequest request) {
        Order order = new Order(request.customerEmail(), request.total());
        Order saved = orderRepository.save(order);
        log.info("Created order id={} for customer={}", saved.getId(), saved.getCustomerEmail());
        return saved;
    }

    public Order findById(Long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new OrderNotFoundException(id));
    }

    public List<Order> findAll() {
        return orderRepository.findAll();
    }

    public Order confirm(Long id) {
        Order order = findById(id);
        try {
            order.confirm();
        } catch (IllegalStateException e) {
            throw new InvalidOrderStateException(e.getMessage());
        }
        return orderRepository.save(order);
    }

    public Order cancel(Long id) {
        Order order = findById(id);
        try {
            order.cancel();
        } catch (IllegalStateException e) {
            throw new InvalidOrderStateException(e.getMessage());
        }
        return orderRepository.save(order);
    }
}
```

## controller/OrderController.java

```java
// src/main/java/com/example/orders/controller/OrderController.java
package com.example.orders.controller;

import com.example.orders.dto.CreateOrderRequest;
import com.example.orders.dto.OrderResponse;
import com.example.orders.model.Order;
import com.example.orders.service.OrderService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public OrderResponse create(@Valid @RequestBody CreateOrderRequest request) {
        return OrderResponse.from(orderService.create(request));
    }

    @GetMapping("/{id}")
    public OrderResponse get(@PathVariable Long id) {
        return OrderResponse.from(orderService.findById(id));
    }

    @GetMapping
    public List<OrderResponse> list() {
        return orderService.findAll().stream().map(OrderResponse::from).toList();
    }

    @PostMapping("/{id}/confirm")
    public OrderResponse confirm(@PathVariable Long id) {
        return OrderResponse.from(orderService.confirm(id));
    }

    @PostMapping("/{id}/cancel")
    public OrderResponse cancel(@PathVariable Long id) {
        return OrderResponse.from(orderService.cancel(id));
    }
}
```

## application.properties

```properties
# src/main/resources/application.properties
spring.application.name=order-management

spring.datasource.url=jdbc:postgresql://${DB_HOST:localhost}:5432/orders
spring.datasource.username=${DB_USER:orders_app}
spring.datasource.password=${DB_PASSWORD}

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false

management.endpoints.web.exposure.include=health,metrics,info
management.endpoint.health.show-details=when-authorized
```

## Unit test — OrderServiceTest.java

```java
// src/test/java/com/example/orders/service/OrderServiceTest.java
package com.example.orders.service;

import com.example.orders.dto.CreateOrderRequest;
import com.example.orders.exception.OrderNotFoundException;
import com.example.orders.model.Order;
import com.example.orders.repository.OrderRepository;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void create_savesAndReturnsOrder() {
        when(orderRepository.save(any(Order.class))).thenAnswer(inv -> inv.getArgument(0));

        Order order = orderService.create(new CreateOrderRequest("alice@example.com", 49.99));

        assertEquals("alice@example.com", order.getCustomerEmail());
        verify(orderRepository).save(any(Order.class));
    }

    @Test
    void findById_throwsWhenMissing() {
        when(orderRepository.findById(404L)).thenReturn(Optional.empty());

        assertThrows(OrderNotFoundException.class, () -> orderService.findById(404L));
    }

    @Test
    void cancel_marksOrderCancelled() {
        Order existing = new Order("bob@example.com", 20.0);
        when(orderRepository.findById(1L)).thenReturn(Optional.of(existing));
        when(orderRepository.save(any(Order.class))).thenAnswer(inv -> inv.getArgument(0));

        Order cancelled = orderService.cancel(1L);

        assertEquals("CANCELLED", cancelled.getStatus().name());
    }
}
```

## Integration test — OrderControllerIT.java

```java
// src/test/java/com/example/orders/controller/OrderControllerIT.java
package com.example.orders.controller;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(properties = {
    "spring.datasource.url=jdbc:h2:mem:orders;MODE=PostgreSQL",
    "spring.datasource.driver-class-name=org.h2.Driver",
    "spring.jpa.hibernate.ddl-auto=create-drop"
})
class OrderControllerIT {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void createThenFetchOrder() throws Exception {
        String payload = """
            {"customerEmail": "carol@example.com", "total": 75.50}
            """;

        String response = mockMvc.perform(post("/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(payload))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.customerEmail").value("carol@example.com"))
            .andExpect(jsonPath("$.status").value("PENDING"))
            .andReturn().getResponse().getContentAsString();

        mockMvc.perform(get("/orders"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$[0].customerEmail").value("carol@example.com"));
    }

    @Test
    void createOrder_rejectsInvalidPayload() throws Exception {
        String badPayload = """
            {"customerEmail": "not-an-email", "total": -5}
            """;

        mockMvc.perform(post("/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(badPayload))
            .andExpect(status().isBadRequest());
    }
}
```

## Dockerfile

```dockerfile
# Dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -B
COPY src ./src
RUN mvn package -DskipTests -B

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN addgroup -S app && adduser -S app -G app
USER app
COPY --from=build /app/target/order-management-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## docker-compose.yml

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: db
      DB_USER: orders_app
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: orders_app
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - order_db_data:/var/lib/postgresql/data

volumes:
  order_db_data:
```

## CI workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
          cache: maven

      - name: Run tests
        run: mvn test -B

  build-image:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t order-management:${{ github.sha }} .
```

## Running it

```bash
# Local build and test
mvn test

# Run with Docker Compose (app + Postgres together)
DB_PASSWORD='SuperSecret123!' docker compose up --build

# Create an order
curl -X POST localhost:8080/orders \
  -H "Content-Type: application/json" \
  -d '{"customerEmail": "dave@example.com", "total": 120.00}'
# {"id":1,"customerEmail":"dave@example.com","total":120.0,"status":"PENDING","createdAt":"2026-01-15T10:00:00Z"}

curl -X POST localhost:8080/orders/1/confirm
# {"id":1,"customerEmail":"dave@example.com","total":120.0,"status":"CONFIRMED", ...}

curl localhost:8080/orders
# [{"id":1,"customerEmail":"dave@example.com", ...}]

curl localhost:8080/actuator/health
# {"status":"UP"}
```

## Stretch goals

- Add pagination to `GET /orders` using `Pageable` (see
  [Module 1](01-spring-ecosystem-production.md)).
- Add a `SecurityFilterChain` requiring authentication for `POST` and
  `DELETE` endpoints while leaving `GET` open, with a `@PreAuthorize` on the
  cancel endpoint restricting it to the order's own customer or an admin.
- Wrap `confirm`/`cancel` with a Micrometer `Counter` (see
  [Module 8](08-observability.md)) tracking how many orders reach each
  terminal state.
- Publish an `OrderCreated` event to Kafka when an order is created (see
  [Module 6](06-messaging-event-driven.md)), and have a separate consumer
  log it — a minimal taste of the event-driven patterns from
  [Module 2](02-microservices-architecture.md).
- Add a Testcontainers-backed test (see [Module 4](04-testing-at-scale.md))
  that runs the full flow against a real, disposable Postgres container
  instead of the in-memory H2 database used above.

You've completed the Java Mastery Path — Entry to Master. From `public
static void main` in Level 1 to a tested, containerized, CI-integrated
microservice in Level 4, every module built on the one before it. The
patterns in this capstone — layered architecture, validated input, global
error handling, automated tests at multiple levels, and a repeatable build
pipeline — are the same shape you'll find in real production Java services.
What you build next is up to you.
