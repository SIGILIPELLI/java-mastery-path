# 07 · Containerization & Deployment

A Java service that "works on my machine" needs a repeatable, portable way
to ship to production. This module covers packaging a Spring Boot app into
a Docker image, running it alongside its database with Compose, and
automating the build/test/publish steps with a CI pipeline.

## Multi-stage Dockerfile

Building inside the final image would drag the entire Maven toolchain and
dependency cache into your production image. A **multi-stage build** compiles
in one stage and copies only the resulting jar into a slim runtime stage.

```dockerfile
# Dockerfile

# ---- Build stage ----
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app

# Copy dependency descriptor first so Docker can cache the download layer
COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn package -DskipTests -B

# ---- Runtime stage ----
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Run as a non-root user
RUN addgroup -S app && adduser -S app -G app
USER app

COPY --from=build /app/target/order-service-*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

The runtime image only contains a JRE (not a full JDK) and the built jar —
typically a fraction of the size of the build stage, and with a much
smaller attack surface since no build tools ship in production.

```text
# .dockerignore
target/
.git/
.idea/
*.iml
Dockerfile
docker-compose.yml
```

## Building and running

```bash
docker build -t order-service:latest .

docker run -p 8080:8080 \
  -e DB_PASSWORD='SuperSecret123!' \
  order-service:latest
```

## docker-compose.yml — app plus database

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/orders
      SPRING_DATASOURCE_USERNAME: orders_app
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: orders_app
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - order_db_data:/var/lib/postgresql/data

volumes:
  order_db_data:
```

```bash
DB_PASSWORD='SuperSecret123!' docker compose up --build
```

The app connects to the database service by its Compose service name (`db`),
not `localhost` — Compose puts both containers on a shared network and
resolves service names via DNS.

## CI/CD with GitHub Actions

A CI pipeline should run the same checks locally available (`mvn test`) on
every push, and build the deployable artifact once tests pass.

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
        run: docker build -t order-service:${{ github.sha }} .

      - name: Log in to registry
        run: echo "${{ secrets.REGISTRY_PASSWORD }}" | docker login -u "${{ secrets.REGISTRY_USER }}" --password-stdin

      - name: Push image
        run: |
          docker tag order-service:${{ github.sha }} myregistry.example.com/order-service:${{ github.sha }}
          docker push myregistry.example.com/order-service:${{ github.sha }}
```

The `build-image` job depends on `test` (`needs: test`) — a failing test
suite blocks the image from ever being built or pushed. Credentials
(`REGISTRY_PASSWORD`) come from GitHub Actions **secrets**, never committed
to the workflow file (see [Module 5](05-security-best-practices.md)).

| Step | Purpose |
|------|---------|
| Multi-stage `Dockerfile` | Separates build tooling from the slim runtime image |
| `.dockerignore` | Keeps the build context small and avoids leaking local files |
| `docker-compose.yml` | Runs the app and its dependencies together for local dev/testing |
| CI `test` job | Fails fast before any image is built |
| CI `build-image` job | Produces a versioned, deployable artifact tied to a commit SHA |

## Exercise

Extend the `Dockerfile` above with a `HEALTHCHECK` instruction that curls
`/actuator/health` every 30 seconds (see [Module 8](08-observability.md) for
the Actuator endpoint). Then add a second service to the `docker-compose.yml`
for a Redis cache (image `redis:7-alpine`) and wire its hostname into the
app's environment as `SPRING_CACHE_REDIS_HOST=redis`.
