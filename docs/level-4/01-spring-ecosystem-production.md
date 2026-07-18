# 01 · Spring Ecosystem in Production

Level 3 introduced Spring Boot for building a REST API. Production systems need
more: repositories that express real query logic without hand-written SQL, and
a security layer that decides who can call what. This module covers Spring
Data JPA beyond basic CRUD, and the fundamentals of Spring Security.

## Spring Data JPA — derived query methods

Spring Data can generate queries from method names alone — no SQL, no
annotations, just a naming convention it parses at startup.

```java
// UserRepository.java
package com.example.orders.repository;

import com.example.orders.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {

    // SELECT * FROM users WHERE email = ? AND active = true
    Optional<User> findByEmailAndActiveTrue(String email);

    // SELECT * FROM users WHERE last_name LIKE ?
    List<User> findByLastNameStartingWithIgnoreCase(String prefix);

    // SELECT COUNT(*) FROM users WHERE active = true
    long countByActiveTrue();

    // SELECT * FROM users WHERE created_at > ? ORDER BY created_at DESC
    List<User> findByCreatedAtAfterOrderByCreatedAtDesc(java.time.Instant since);
}
```

Method names are parsed into predicates: `findBy`, `And`/`Or`, comparison
keywords (`After`, `StartingWith`, `GreaterThan`), and a trailing `True`/`False`
for booleans. Spring Data generates the JPQL at application startup and fails
fast if a method name doesn't map to a real property.

## `@Query` for anything derived names can't express

When a query needs joins, aggregation, or is simply clearer written out,
drop to JPQL (or native SQL) explicitly:

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("""
        SELECT o FROM Order o
        WHERE o.customer.id = :customerId
          AND o.status = 'SHIPPED'
        ORDER BY o.createdAt DESC
        """)
    List<Order> findShippedOrdersForCustomer(@Param("customerId") Long customerId);

    // Native SQL when JPQL can't express a dialect-specific feature
    @Query(value = "SELECT * FROM orders WHERE total_cents > :minCents", nativeQuery = true)
    List<Order> findHighValueOrders(@Param("minCents") long minCents);
}
```

`@Param` binds the named parameter in the query string to the method
argument — this avoids string concatenation entirely, so these queries are
not vulnerable to SQL injection (see
[Module 5](05-security-best-practices.md)).

## Pagination with `Pageable`

Returning every row is fine for a demo, not for a table with a million
orders. Accept a `Pageable` and return a `Page<T>`:

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    Page<Order> findByCustomerId(Long customerId, Pageable pageable);
}
```

```java
// OrderService.java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;

public Page<Order> ordersForCustomer(Long customerId, int page, int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    return orderRepository.findByCustomerId(customerId, pageable);
}
```

`Page<T>` carries the content plus `getTotalElements()`, `getTotalPages()`,
and `hasNext()` — everything a paginated API response needs.

## Spring Security basics

Spring Security wires a filter chain in front of every request. The modern
(non-deprecated) way to configure it is a `SecurityFilterChain` bean:

```java
// SecurityConfig.java
package com.example.orders.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();   // salts + hashes automatically
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health", "/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(basic -> {})          // HTTP Basic for simplicity here
            .formLogin(form -> {})            // also enable form login
            .csrf(csrf -> csrf.disable());     // safe to disable for stateless APIs using tokens

        return http.build();
    }
}
```

Never store plaintext passwords. `BCryptPasswordEncoder` hashes with a random
salt baked into the output, so two identical passwords produce different
hashes:

```java
PasswordEncoder encoder = new BCryptPasswordEncoder();

String hash = encoder.encode("correct-horse-battery-staple");
// stored in the database, e.g. "$2a$10$N9qo8uLOickgx2ZMRZoMy..."

boolean matches = encoder.matches("correct-horse-battery-staple", hash);   // true
boolean wrong = encoder.matches("wrong-password", hash);                    // false
```

## Method-level authorization with `@PreAuthorize`

`.authorizeHttpRequests` covers whole URL patterns. `@PreAuthorize` lets you
guard an individual method with a Spring Expression Language (SpEL)
condition, evaluated before the method body runs:

```java
// OrderController.java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/{id}")
    public void deleteOrder(@PathVariable Long id) {
        orderService.delete(id);
    }

    @PreAuthorize("#customerId == authentication.principal.id or hasRole('ADMIN')")
    @GetMapping("/customer/{customerId}")
    public List<Order> ordersForCustomer(@PathVariable Long customerId) {
        return orderService.forCustomer(customerId);
    }
}
```

`@PreAuthorize` requires `@EnableMethodSecurity` on a configuration class to
be active.

| Concept | Purpose |
|---------|---------|
| `JpaRepository<T, ID>` | Base interface giving CRUD + paging for free |
| Derived query method | Query generated from the method name |
| `@Query` | Explicit JPQL/native SQL for complex queries |
| `Pageable` / `Page<T>` | Request and receive one page of a large result set |
| `SecurityFilterChain` | Declares which requests need which authorization |
| `BCryptPasswordEncoder` | One-way salted password hashing |
| `@PreAuthorize` | Method-level, expression-based access control |

## Exercise

Add a `findByActiveTrueAndCountryCode(String countryCode)` derived query
method to a `CustomerRepository`, then add a paginated
`findByCountryCode(String countryCode, Pageable pageable)` method. Next, write
a `SecurityFilterChain` bean that permits `GET /customers/**` to any
authenticated user but restricts `POST /customers/**` and `DELETE
/customers/**` to users with role `ADMIN`, and add a `@PreAuthorize("hasRole('ADMIN')")`
annotation directly on the controller's delete method as a second line of
defense.
