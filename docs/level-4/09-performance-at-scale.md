# 09 · Performance at Scale

[Level 3, Module 9](../level-3/09-jvm-internals-gc.md) covered how the JVM
manages memory and garbage collection conceptually. This module is about
turning that understanding into concrete tuning: JVM flags, connection pool
sizing, and caching — the three levers that most commonly separate a service
that falls over under load from one that doesn't.

## JVM tuning flags

```bash
java \
  -Xms512m -Xmx512m \       # initial and max heap -- setting both equal avoids runtime resizing pauses
  -XX:+UseG1GC \             # G1 -- the default low-pause collector for most server workloads
  -Xss512k \                 # per-thread stack size -- lower it to allow more threads in a fixed memory budget
  -jar order-service.jar
```

| Flag | Meaning |
|------|---------|
| `-Xms` / `-Xmx` | Initial / maximum heap size — set equal in production to avoid resize pauses |
| `-XX:+UseG1GC` | Use the G1 garbage collector — balances throughput and pause time for most services |
| `-XX:+UseZGC` | Use ZGC — sub-millisecond pauses, favored for very large heaps or strict latency SLAs |
| `-Xss` | Stack size per thread — smaller stacks let you run more platform threads in the same memory |
| `-XX:+HeapDumpOnOutOfMemoryError` | Writes a heap dump automatically on OOM, for later analysis |

Undersizing the heap causes frequent GC pauses; oversizing it wastes memory
and can make each (rarer) full GC pause longer. Load-test before committing
to numbers — there's no universal "right" heap size.

## Connection pool tuning (HikariCP)

Spring Boot uses HikariCP as its default JDBC connection pool. A pool that's
too small serializes requests waiting for a free connection; too large
overwhelms the database with more concurrent connections than it can
efficiently serve.

```properties
# application.properties
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=3000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
```

| Setting | Effect |
|---------|--------|
| `maximum-pool-size` | Hard cap on concurrent DB connections — size to what the database can actually handle, not to thread count |
| `minimum-idle` | Connections kept warm even when idle, avoiding cold-start latency on a burst of traffic |
| `connection-timeout` | How long a request waits for a free connection before failing fast instead of queuing forever |
| `max-lifetime` | Forces connections to recycle periodically, avoiding stale connections a load balancer or firewall silently dropped |

A common mistake is setting `maximum-pool-size` far higher than the database
can support, assuming "more connections = more throughput" — past a point,
the database spends more time context-switching between connections than
doing work.

## Caching strategies

### In-process caching with Spring's `@Cacheable`

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

```java
// ProductService.java
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Cacheable(value = "products", key = "#sku")
    public Product findBySku(String sku) {
        // Only runs on a cache miss -- subsequent calls with the same sku
        // return the cached value without touching the database at all.
        return productRepository.findBySku(sku)
            .orElseThrow(() -> new ProductNotFoundException(sku));
    }

    @CacheEvict(value = "products", key = "#product.sku()")
    public void updatePrice(Product product) {
        productRepository.save(product);   // invalidate the stale cached entry
    }
}
```

This follows the **cache-aside** pattern: the application checks the cache
first, and on a miss, loads from the database and populates the cache for
next time. The application is fully responsible for keeping the cache
consistent (via `@CacheEvict`) when the underlying data changes.

### Distributed caching with Redis

An in-process cache (the default backing for `@Cacheable`) is per-instance —
each replica of a horizontally scaled service has its own copy, and one
instance's cache invalidation doesn't affect the others. **Redis** as a
shared, external cache solves this: every instance reads and writes the same
cache, so an eviction on one instance is visible to all.

```properties
# application.properties -- swap the cache backend to Redis, code above is unchanged
spring.cache.type=redis
spring.data.redis.host=redis
spring.data.redis.port=6379
```

The `@Cacheable`/`@CacheEvict` annotations don't change at all — only the
configured cache provider does, which is the point of Spring's cache
abstraction: business logic doesn't know or care where the cache lives.

| Lever | Typical effect |
|-------|------------------|
| Right-sizing heap (`-Xms`/`-Xmx`) | Fewer, more predictable GC pauses |
| Choosing a GC algorithm | Trades throughput vs. pause-time latency |
| Connection pool size | Prevents request queuing without overwhelming the database |
| `@Cacheable` for hot, rarely-changing reads | Removes repeated database round-trips entirely |
| Distributed cache (Redis) | Keeps cache consistent and shared across horizontally scaled instances |

## Exercise

Add `@Cacheable(value = "exchangeRates", key = "#currencyCode")` to a
`CurrencyService.getRate(String currencyCode)` method, and a
`@CacheEvict(value = "exchangeRates", allEntries = true)` method
`refreshRates()` that clears the whole cache. Then write the HikariCP
properties you'd set for a service expecting at most 15 concurrent database
queries at once, explaining your choice of `maximum-pool-size` in one
sentence.
