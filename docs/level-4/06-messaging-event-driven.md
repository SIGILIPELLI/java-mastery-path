# 06 · Messaging & Event-Driven Systems

Synchronous REST calls couple a caller to a callee's uptime and latency. An
event-driven architecture decouples services with a message broker in
between: producers publish messages without knowing who (if anyone) is
listening, and consumers process them on their own schedule. This module
covers the two dominant brokers — Kafka and RabbitMQ — and how to talk to
each from Java.

## Pub/sub vs. point-to-point

- **Point-to-point (queue)**: one message, delivered to exactly one
  consumer. Good for work distribution — a pool of workers competing for
  jobs off a queue.
- **Publish/subscribe (topic)**: one message, delivered to every subscriber.
  Good for broadcasting an event (`OrderCreated`) to multiple independent
  interested services (billing, shipping, analytics) without the producer
  knowing they exist.

Kafka topics support both patterns depending on consumer group
configuration; RabbitMQ makes the distinction explicit through exchange
types.

## Kafka producer

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.7.0</version>
</dependency>
```

```java
// OrderEventProducer.java
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class OrderEventProducer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.ACKS_CONFIG, "all");   // wait for all in-sync replicas to ack

        try (KafkaProducer<String, String> producer = new KafkaProducer<>(props)) {
            String orderId = "order-1042";
            String payload = """
                {"orderId": "order-1042", "status": "CREATED", "total": 129.99}
                """;

            ProducerRecord<String, String> record =
                new ProducerRecord<>("order-events", orderId, payload);   // key = orderId

            producer.send(record, (metadata, exception) -> {
                if (exception != null) {
                    System.err.println("Send failed: " + exception.getMessage());
                } else {
                    System.out.printf("Sent to partition %d, offset %d%n",
                        metadata.partition(), metadata.offset());
                }
            });
        }
    }
}
```

Using the order ID as the message **key** guarantees every event for the
same order lands in the same partition, so consumers see them in order.

## Kafka consumer

```java
// OrderEventConsumer.java
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.List;
import java.util.Properties;

public class OrderEventConsumer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "billing-service");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {
            consumer.subscribe(List.of("order-events"));

            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("Consumed key=%s value=%s partition=%d offset=%d%n",
                        record.key(), record.value(), record.partition(), record.offset());
                    processBillingEvent(record.value());
                }
                consumer.commitSync();   // acknowledge -- won't be redelivered after this
            }
        }
    }

    static void processBillingEvent(String payload) {
        // parse and act on the event
    }
}
```

Every consumer in the same `GROUP_ID_CONFIG` shares the work — Kafka assigns
each partition to exactly one consumer in the group, giving point-to-point
distribution; a *different* group ID subscribing to the same topic gets its
own full copy of every message, giving pub/sub.

## Running Kafka locally

```yaml
# docker-compose.yml
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

```bash
docker compose up -d
```

## RabbitMQ comparison

RabbitMQ is a traditional message broker built around **exchanges**,
**queues**, and **routing keys**: a producer publishes to an exchange,
which routes the message to zero or more bound queues based on the routing
key and exchange type (`direct`, `topic`, `fanout`). Consumers subscribe to
a queue and the broker pushes messages to them.

```text
Producer -> Exchange (type: topic) -> routing key "order.created" -> Queue "billing-queue"
                                    -> routing key "order.created" -> Queue "analytics-queue"
```

| | Kafka | RabbitMQ |
|---|---|---|
| Model | Distributed log — consumers track their own offset | Traditional broker — pushes to queues, tracks delivery |
| Message retention | Retained for a configured period regardless of consumption | Typically removed once acknowledged/consumed |
| Replay | Easy — reset a consumer group's offset and re-read history | Not designed for replay |
| Throughput | Very high, built for streaming/event-log workloads | High, optimized for flexible routing and lower-latency queuing |
| Routing flexibility | Simple (topic + partition key) | Rich (exchange types, routing keys, bindings) |
| Typical use | Event streaming, analytics pipelines, event sourcing | Task queues, RPC-style messaging, complex routing |

See [Module 2](02-microservices-architecture.md) for how Saga choreography
commonly rides on top of this kind of event backbone.

## Exercise

Write a Kafka producer that publishes a `PaymentReceived` JSON event (fields:
`orderId`, `amount`, `receivedAt`) to a topic named `payment-events`, keyed
by `orderId`. Then write a consumer in a group called `order-service` that
polls the topic and prints each event. Finally, describe in one or two
sentences whether you'd model "notify three unrelated services about a
new order" as Kafka pub/sub or a RabbitMQ fanout exchange, and why.
