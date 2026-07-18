# 04 · Testing at Scale

[Level 2](../level-2/07-junit-testing.md) covered JUnit 5 fundamentals for
testing pure logic. Production services need three more layers: mocking
collaborators so a unit test isolates one class, integration tests that
exercise real Spring wiring and HTTP, and tests that run against a real
database in a disposable container.

## Mockito basics

Mockito creates fake implementations of interfaces/classes so a unit test
can isolate the class under test from its dependencies.

```xml
<!-- pom.xml -- included transitively by spring-boot-starter-test -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

```java
// OrderServiceTest.java
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
    private OrderRepository orderRepository;   // fake -- no real database involved

    @InjectMocks
    private OrderService orderService;         // real class, with the mock injected in

    @Test
    void cancelOrder_marksOrderCancelled() {
        Order existing = new Order(1L, "PENDING", 99.0);
        when(orderRepository.findById(1L)).thenReturn(Optional.of(existing));
        when(orderRepository.save(any(Order.class))).thenAnswer(inv -> inv.getArgument(0));

        Order result = orderService.cancel(1L);

        assertEquals("CANCELLED", result.status());
        verify(orderRepository).save(argThat(o -> o.status().equals("CANCELLED")));
    }

    @Test
    void cancelOrder_throwsWhenNotFound() {
        when(orderRepository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(OrderNotFoundException.class, () -> orderService.cancel(99L));
        verify(orderRepository, never()).save(any());   // never saved because it never got that far
    }
}
```

`when(...).thenReturn(...)` stubs behavior; `verify(...)` asserts a method
was actually called (and how many times, with `never()`/`times(n)`). Mockito
never touches a real database — this test runs in milliseconds.

## Spring Boot integration tests with `MockMvc`

Unit tests isolate one class. Integration tests boot the real Spring context
and exercise HTTP-layer behavior — routing, validation, serialization —
without a running server process.

```java
// OrderControllerIT.java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
class OrderControllerIT {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void getOrder_returnsJson() throws Exception {
        mockMvc.perform(get("/orders/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.status").value("PENDING"));
    }

    @Test
    void createOrder_validatesInput() throws Exception {
        String badPayload = """
            {"customerId": null, "total": -5}
            """;

        mockMvc.perform(post("/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(badPayload))
            .andExpect(status().isBadRequest());
    }
}
```

`@SpringBootTest` starts the full application context; `@AutoConfigureMockMvc`
wires a `MockMvc` that dispatches requests through the real controller stack
without opening a network socket — faster than a full end-to-end HTTP test,
but still exercising real request mapping and JSON (de)serialization.

## Testcontainers — a real database in tests

Mocking the repository is fine for service-layer logic, but at some point
you want to know your JPA queries actually work against a real database
engine. **Testcontainers** starts a real, disposable Postgres instance in
Docker just for the test run.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.7</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.7</version>
    <scope>test</scope>
</dependency>
```

```java
// OrderRepositoryTestcontainersIT.java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.junit.jupiter.api.Assertions.assertEquals;

@Testcontainers
@SpringBootTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class OrderRepositoryTestcontainersIT {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
            .withDatabaseName("orders_test")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void datasourceProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private OrderRepository orderRepository;

    @Test
    void savesAndReadsBackFromRealPostgres() {
        Order saved = orderRepository.save(new Order(null, "PENDING", 150.0));

        Order found = orderRepository.findById(saved.id()).orElseThrow();

        assertEquals("PENDING", found.status());
    }
}
```

Testcontainers spins the container up before the test class runs and tears
it down after — no shared, hand-maintained test database, and no drift
between what tests run against and what production runs on (real Postgres,
not an in-memory H2 stand-in that behaves subtly differently).

| Test type | Speed | What it verifies | Tool |
|-----------|-------|-------------------|------|
| Unit test with mocks | Milliseconds | One class's logic in isolation | Mockito |
| Spring integration test | ~1 second | Controller routing, validation, wiring | `@SpringBootTest` + `MockMvc` |
| Testcontainers test | Seconds (container startup) | Real SQL against a real database engine | Testcontainers |

## Exercise

Write a `Mockito`-based unit test for a `PaymentService.refund(Long orderId)`
method that mocks a `PaymentGatewayClient` dependency, asserting the gateway's
`refund` method is called exactly once with the correct amount. Then write a
`MockMvc` integration test for `POST /orders/{id}/refund` asserting it
returns `200 OK` and a JSON body with `"status": "REFUNDED"`.
