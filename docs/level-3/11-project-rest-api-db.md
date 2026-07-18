# 11 · Project — REST API + Database CRUD Service

A capstone project combining everything from Level 3: Spring Boot controllers
and dependency injection, JDBC-backed persistence (via Spring Data JPA), and
clean layered design.

## What you'll build

A **Task management REST API** backed by an H2 database, exposing full CRUD:

- `GET /tasks` — list all tasks
- `GET /tasks/{id}` — fetch one task
- `POST /tasks` — create a task
- `PUT /tasks/{id}` — update a task
- `DELETE /tasks/{id}` — delete a task

## Project layout

```text
task-api/
    pom.xml
    src/
        main/
            java/
                com/example/taskapi/
                    TaskApiApplication.java
                    Task.java
                    TaskRepository.java
                    TaskService.java
                    TaskController.java
            resources/
                application.properties
```

## pom.xml

```xml
<!-- pom.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>task-api</artifactId>
    <version>1.0.0</version>

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
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
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

## application.properties

```text
# src/main/resources/application.properties

# File-based H2 database so data survives across restarts (use jdbc:h2:mem:... for pure in-memory)
spring.datasource.url=jdbc:h2:file:./data/taskdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# Automatically create/update tables from @Entity classes at startup
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# Enables the H2 web console at /h2-console for inspecting data during development
spring.h2.console.enabled=true
```

## Task.java — the entity

```java
// src/main/java/com/example/taskapi/Task.java
package com.example.taskapi;

import jakarta.persistence.*;

@Entity
@Table(name = "tasks")
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    private boolean completed;

    protected Task() { }   // required no-arg constructor for JPA

    public Task(String title, boolean completed) {
        this.title = title;
        this.completed = completed;
    }

    public Long getId() { return id; }
    public String getTitle() { return title; }
    public boolean isCompleted() { return completed; }

    public void setTitle(String title) { this.title = title; }
    public void setCompleted(boolean completed) { this.completed = completed; }
}
```

## TaskRepository.java — data access

```java
// src/main/java/com/example/taskapi/TaskRepository.java
package com.example.taskapi;

import org.springframework.data.jpa.repository.JpaRepository;

// Spring Data JPA generates the implementation at runtime -- CRUD methods
// like findAll(), findById(), save(), and deleteById() come for free.
public interface TaskRepository extends JpaRepository<Task, Long> {
}
```

## TaskService.java — business logic layer

```java
// src/main/java/com/example/taskapi/TaskService.java
package com.example.taskapi;

import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class TaskService {

    private final TaskRepository repository;

    public TaskService(TaskRepository repository) {
        this.repository = repository;
    }

    public List<Task> findAll() {
        return repository.findAll();
    }

    public Optional<Task> findById(Long id) {
        return repository.findById(id);
    }

    public Task create(String title) {
        return repository.save(new Task(title, false));
    }

    public Optional<Task> update(Long id, String title, boolean completed) {
        return repository.findById(id).map(task -> {
            task.setTitle(title);
            task.setCompleted(completed);
            return repository.save(task);
        });
    }

    public boolean delete(Long id) {
        if (!repository.existsById(id)) {
            return false;
        }
        repository.deleteById(id);
        return true;
    }
}
```

## TaskController.java — REST endpoints

```java
// src/main/java/com/example/taskapi/TaskController.java
package com.example.taskapi;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/tasks")
public class TaskController {

    private final TaskService taskService;

    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }

    // Request/response payload for create and update -- keeps Task (the JPA
    // entity) decoupled from the shape callers send over the wire.
    public record TaskRequest(String title, boolean completed) { }

    @GetMapping
    public List<Task> getAll() {
        return taskService.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Task> getOne(@PathVariable Long id) {
        return taskService.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<Task> create(@RequestBody TaskRequest request) {
        Task created = taskService.create(request.title());
        return ResponseEntity.status(201).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Task> update(@PathVariable Long id, @RequestBody TaskRequest request) {
        return taskService.update(id, request.title(), request.completed())
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        boolean deleted = taskService.delete(id);
        return deleted ? ResponseEntity.noContent().build() : ResponseEntity.notFound().build();
    }
}
```

## TaskApiApplication.java — entry point

```java
// src/main/java/com/example/taskapi/TaskApiApplication.java
package com.example.taskapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TaskApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(TaskApiApplication.class, args);
    }
}
```

## Running it

```bash
mvn spring-boot:run
```

```bash
# Create a task
curl -X POST http://localhost:8080/tasks \
     -H "Content-Type: application/json" \
     -d '{"title": "Write Level 3 capstone docs", "completed": false}'
# {"id":1,"title":"Write Level 3 capstone docs","completed":false}

# List all tasks
curl http://localhost:8080/tasks
# [{"id":1,"title":"Write Level 3 capstone docs","completed":false}]

# Fetch one task
curl http://localhost:8080/tasks/1
# {"id":1,"title":"Write Level 3 capstone docs","completed":false}

# Update a task
curl -X PUT http://localhost:8080/tasks/1 \
     -H "Content-Type: application/json" \
     -d '{"title": "Write Level 3 capstone docs", "completed": true}'
# {"id":1,"title":"Write Level 3 capstone docs","completed":true}

# Delete a task
curl -X DELETE http://localhost:8080/tasks/1 -i
# HTTP/1.1 204 No Content

# Confirm deletion
curl http://localhost:8080/tasks/1 -i
# HTTP/1.1 404 Not Found
```

The service layer (`TaskService`) sits between the controller and the
repository so validation or business rules can grow there later without
touching HTTP-handling code — the same layered structure real production
Spring Boot services use.

## Stretch goals

- Add request validation with `spring-boot-starter-validation`
  (`@NotBlank` on `title`, returning a 400 response for invalid input).
- Add a `GET /tasks?completed=true` filter using a derived query method on
  `TaskRepository`, e.g. `List<Task> findByCompleted(boolean completed)`.
- Add pagination with `Pageable` (`GET /tasks?page=0&size=10`).
- Write integration tests with `@SpringBootTest` and `TestRestTemplate`
  covering all five endpoints.
- Swap H2 for a real PostgreSQL database using Docker, changing only
  `application.properties` and the JDBC dependency.

Completing this project means you're ready for **Level 4 · Master**.
