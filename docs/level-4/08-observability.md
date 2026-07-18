# 08 · Observability

Once a service runs in production, `System.out.println` and a debugger are
no longer available to you. Observability — logs, metrics, and traces — is
how you understand what a running system is actually doing.

## Structured logging with SLF4J and Logback

Spring Boot uses SLF4J as a logging facade over Logback by default. Log
through the facade, never `System.out`, so log level and output format are
configurable without touching code.

```java
// OrderService.java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private static final Logger log = LoggerFactory.getLogger(OrderService.class);

    public Order create(CreateOrderRequest request) {
        log.info("Creating order for customer={}", request.customerId());

        try {
            Order order = doCreate(request);
            log.info("Order created id={} total={}", order.id(), order.total());
            return order;
        } catch (PaymentDeclinedException e) {
            log.warn("Payment declined for customer={}: {}", request.customerId(), e.getMessage());
            throw e;
        } catch (Exception e) {
            log.error("Unexpected error creating order for customer={}", request.customerId(), e);
            throw e;
        }
    }
}
```

Using `{}` placeholders instead of string concatenation avoids building the
message string at all when the log level is disabled — cheaper at scale,
and it keeps the log line legible.

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- structured, single-line format: timestamp level logger - message -->
            <pattern>%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Quiet down noisy framework logs -->
    <logger name="org.springframework" level="WARN"/>
    <logger name="com.example.orders" level="INFO"/>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

| Level | Use for |
|-------|---------|
| `ERROR` | Something failed and needs attention |
| `WARN` | Unexpected but handled (e.g. a declined payment) |
| `INFO` | Notable business events (order created, user registered) |
| `DEBUG` | Detailed flow useful only when actively diagnosing an issue |

## Metrics with Micrometer and Spring Boot Actuator

Actuator exposes operational endpoints out of the box once added as a
dependency:

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# application.properties
management.endpoints.web.exposure.include=health,metrics,info
management.endpoint.health.show-details=when-authorized
```

```bash
curl localhost:8080/actuator/health
# {"status":"UP"}

curl localhost:8080/actuator/metrics/jvm.memory.used
# {"name":"jvm.memory.used","measurements":[{"statistic":"VALUE","value":8.4e7}], ...}
```

Micrometer (bundled with Actuator) lets you record your own application
metrics alongside the built-in JVM/HTTP ones:

```java
// OrderMetrics.java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.stereotype.Component;

@Component
public class OrderMetrics {

    private final Counter ordersCreated;
    private final Timer orderCreationTimer;

    public OrderMetrics(MeterRegistry registry) {
        this.ordersCreated = Counter.builder("orders.created")
            .description("Total number of orders created")
            .register(registry);

        this.orderCreationTimer = Timer.builder("orders.creation.duration")
            .description("Time taken to create an order")
            .register(registry);
    }

    public void recordOrderCreated() {
        ordersCreated.increment();
    }

    public <T> T timeCreation(java.util.function.Supplier<T> action) {
        return orderCreationTimer.record(action);
    }
}
```

```java
// usage inside OrderService
public Order create(CreateOrderRequest request) {
    Order order = orderMetrics.timeCreation(() -> doCreate(request));
    orderMetrics.recordOrderCreated();
    return order;
}
```

These custom metrics show up alongside the built-in ones at
`/actuator/metrics/orders.created`, ready to be scraped by Prometheus or any
metrics backend.

## Distributed tracing basics

In a single service, a stack trace tells you what happened. Across a dozen
microservices handling one user request, you need a **trace ID** that
follows the request through every hop, plus a **span ID** per unit of work
within that trace, so you can reconstruct the full timeline afterward.

```text
Trace: 7f3a9c...           (one whole user request, gateway to database)
 ├─ Span: api-gateway        (12ms)
 ├─ Span: order-service      (45ms)
 │   └─ Span: postgres query (30ms)
 └─ Span: inventory-service  (20ms)
```

**OpenTelemetry** is the current standard for generating and propagating
this trace context automatically — it instruments HTTP clients/servers to
attach the trace ID to outgoing headers (`traceparent`), so every downstream
service's logs and spans can be correlated back to the same originating
request, typically visualized in a tool like Jaeger, Zipkin, or a hosted
APM. In Spring applications, adding the `micrometer-tracing` bridge (the
successor to Spring Cloud Sleuth) auto-injects the trace and span ID into
every log line, so a single `grep` for a trace ID across all services'
logs reconstructs the whole request's path.

| Signal | Answers | Tooling in this stack |
|--------|---------|------------------------|
| Logs | What happened, in detail, at a point in time | SLF4J + Logback |
| Metrics | How much / how often / how fast, aggregated over time | Micrometer + Actuator |
| Traces | How one request's time was spent across services | OpenTelemetry / Micrometer Tracing |

## Exercise

Add a `Counter` named `orders.cancelled` and a `Timer` named
`orders.cancellation.duration` to a `CancellationMetrics` component, and
call them from an `OrderService.cancel(Long id)` method. Then write a
`logback-spring.xml` snippet that logs `com.example.orders` at `DEBUG` but
keeps everything else at `INFO`, and expose the `/actuator/metrics` endpoint
alongside `/actuator/health`.
