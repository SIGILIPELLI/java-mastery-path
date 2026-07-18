# 02 · Microservices Architecture Patterns

A monolith becomes hard to scale and deploy independently once a team and
codebase grow large enough. Microservices split an application into small,
independently deployable services — at the cost of new problems: how do
clients find services, how do failures get contained, and how do
transactions work across service boundaries? This module covers the
patterns that answer those questions.

## API Gateway

Instead of clients calling every microservice directly, they call a single
**API Gateway** that routes requests to the right backend service, and
centralizes cross-cutting concerns: authentication, rate limiting, request
logging, and response aggregation.

```text
                     ┌───────────────┐
   mobile app  ───▶  │  API Gateway   │
   web app     ───▶  │ (routing, auth)│
                     └───────┬───────┘
                 ┌───────────┼───────────┐
                 ▼           ▼           ▼
           Order Service  User Service  Inventory Service
```

A gateway route (conceptually, using Spring Cloud Gateway style config):

```yaml
# gateway routes
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/orders/**
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/users/**
```

Clients only need to know one hostname; the gateway hides how many services
exist behind it and can change routing without clients noticing.

## Service discovery

In a monolith, calling another module is a Java method call. In
microservices, calling another service means finding *where* an instance of
it currently runs — and instances come and go as they scale, restart, or get
redeployed. A **service registry** (Eureka, Consul, or Kubernetes' built-in
DNS-based discovery) keeps a live directory of healthy instances, so a
service asks "who is `ORDER-SERVICE` right now?" instead of hardcoding an IP.

```java
// Using a discovery-aware RestTemplate/WebClient -- "order-service"
// resolves to a live instance via the registry, not a hardcoded host:port
@Service
public class InventoryClient {
    private final WebClient webClient;

    public InventoryClient(WebClient.Builder builder) {
        this.webClient = builder.baseUrl("http://order-service").build();
    }

    public Mono<Order> fetchOrder(Long id) {
        return webClient.get().uri("/orders/{id}", id).retrieve().bodyToMono(Order.class);
    }
}
```

## Circuit Breaker (Resilience4j)

When a downstream service is slow or down, calling it repeatedly wastes
threads and cascades the failure upstream. A **circuit breaker** tracks
failure rates and "opens" — short-circuiting calls immediately with a
fallback — once failures cross a threshold, giving the downstream service
time to recover.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.2.0</version>
</dependency>
```

```java
// InventoryService.java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;

@Service
public class InventoryService {

    private final InventoryClient inventoryClient;

    public InventoryService(InventoryClient inventoryClient) {
        this.inventoryClient = inventoryClient;
    }

    @CircuitBreaker(name = "inventoryService", fallbackMethod = "fallbackStock")
    public int checkStock(String sku) {
        return inventoryClient.getStockLevel(sku);   // may throw if the service is down
    }

    // Fallback signature must match + accept the Throwable
    private int fallbackStock(String sku, Throwable t) {
        return -1;   // sentinel meaning "unknown, service is degraded"
    }
}
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      inventoryService:
        sliding-window-size: 20
        failure-rate-threshold: 50   # percent
        wait-duration-in-open-state: 10s
```

Once 50% of the last 20 calls fail, the circuit opens: further calls skip
straight to `fallbackStock` without hitting the network, until the wait
duration passes and it tries a few "trial" requests again.

## Saga pattern for distributed transactions

A single database transaction can't span multiple services, each with their
own database. The **Saga pattern** replaces one big transaction with a
sequence of local transactions, each publishing an event that triggers the
next step — and if a step fails, **compensating actions** undo the previous
steps.

```text
1. Order Service:      create order (PENDING)        --publishes--> OrderCreated
2. Payment Service:    charge card                    --publishes--> PaymentCompleted
3. Inventory Service:  reserve stock                   --publishes--> StockReserved
4. Order Service:      mark order CONFIRMED

If step 2 fails (card declined):
2'. Order Service:     compensate -- mark order CANCELLED
```

```java
// Simplified saga step -- reacts to an event, does local work, emits the next event
@Service
public class PaymentSagaHandler {

    @EventListener
    public void on(OrderCreatedEvent event) {
        try {
            paymentService.charge(event.orderId(), event.amount());
            eventPublisher.publishEvent(new PaymentCompletedEvent(event.orderId()));
        } catch (PaymentDeclinedException e) {
            // compensating action: tell the order service to roll back
            eventPublisher.publishEvent(new PaymentFailedEvent(event.orderId()));
        }
    }
}
```

Sagas trade strict ACID consistency for **eventual consistency** — every
step must be idempotent (safe to retry) since messages can be redelivered.

## Database-per-service

Each microservice owns its own database schema — no other service reads or
writes it directly. If the `Inventory` service needs order data, it asks the
`Order` service's API (or subscribes to its events); it never queries the
orders table directly. This keeps services independently deployable: an
`Order` schema migration can't accidentally break `Inventory`.

| Pattern | Solves | Trade-off |
|---------|--------|-----------|
| API Gateway | Single entry point, cross-cutting concerns | Gateway becomes a critical path — must scale it too |
| Service discovery | Finding live instances dynamically | Extra moving part (the registry) to keep healthy |
| Circuit breaker | Stopping cascading failures | Fallbacks must degrade gracefully, not just error |
| Saga | Transactions across services | Eventual consistency, requires idempotent, compensable steps |
| Database-per-service | Independent deployability | No cross-service joins; more data duplication/events |

See [Level 3, Module 4](../level-3/04-rest-apis-spring-boot.md) for the REST
API fundamentals these services are built on, and
[Module 6](06-messaging-event-driven.md) for the event backbone that saga
choreography and service-to-service events typically run on.

## Exercise

Sketch (in code comments or a short `text` diagram) a 3-step saga for
placing a food delivery order: `Order Service` creates the order, `Restaurant
Service` accepts or rejects it, `Delivery Service` assigns a courier. Write
the compensating action for the case where the restaurant rejects the order
after a courier has already been tentatively assigned. Then annotate a
Java method `assignCourier(Long orderId)` with `@CircuitBreaker` and write
a fallback method that returns `null` when the delivery service is
unreachable.
