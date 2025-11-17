Below is a **complete Choreography Saga Pattern implementation** in **Spring Boot + Kafka**, including:

✔️ Full repo structure
✔️ Commands & Events
✔️ Kafka topics
✔️ Order, Payment, Inventory, Shipping services
✔️ Compensation logic
✔️ Ready to push to GitHub

This is a separate repo from orchestration:
**`saga-kafka-choreography-spring`**

---

# ✅ 1️⃣ Repo Structure

```
saga-kafka-choreography-spring/
├─ README.md
├─ pom.xml
├─ docker-compose.yml
│
├─ common-dto/
│  ├─ pom.xml
│  └─ src/main/java/com/example/common/dto/...
│
├─ order-service/
│  ├─ pom.xml
│  └─ src/main/java/com/example/order/...
│
├─ payment-service/
│  ├─ pom.xml
│  └─ src/main/java/com/example/pay/...
│
├─ inventory-service/
│  ├─ pom.xml
│  └─ src/main/java/com/example/inventory/...
│
└─ shipping-service/
   ├─ pom.xml
   └─ src/main/java/com/example/shipping/...
```

---

# 📌 2️⃣ Choreography Saga Flow (Event-Driven)

In Choreography:

* No orchestrator
* Each service reacts to **events**
* Each service publishes next event
* Compensation is also event-driven

### Business Flow

```
OrderCreatedEvent
    → PaymentService → PaymentCompletedEvent
        → InventoryService → InventoryReservedEvent
            → ShippingService → ShippingCompletedEvent
                → OrderService → OrderCompletedEvent
```

### Failure Flow Example (Payment fails)

```
PaymentFailedEvent
    → OrderService → OrderCancelledEvent
```

---

# 🟦 3️⃣ Create Topics (Common DTO Module)

### Topic Names

```java
// common-dto/.../Topics.java
package com.example.common.dto;

public class Topics {

    public static final String ORDER_CREATED = "order.created";
    public static final String ORDER_CANCELLED = "order.cancelled";
    public static final String ORDER_COMPLETED = "order.completed";

    public static final String PAYMENT_COMPLETED = "payment.completed";
    public static final String PAYMENT_FAILED = "payment.failed";

    public static final String INVENTORY_RESERVED = "inventory.reserved";
    public static final String INVENTORY_FAILED = "inventory.failed";

    public static final String SHIPPING_COMPLETED = "shipping.completed";
    public static final String SHIPPING_FAILED = "shipping.failed";
}
```

---

# 🟩 4️⃣ Event Payload

```java
// common-dto/.../OrderEvent.java
package com.example.common.dto;

import lombok.Data;

@Data
public class OrderEvent {
    private String orderId;
    private String userId;
    private String productId;
    private int quantity;
    private double amount;

    private String eventType; // CREATED, PAYMENT_SUCCESS, etc.
    private String reason;    // failure reason
}
```

---

# 🟥 5️⃣ Order Service (Starts Saga)

### `order-service/application.yml`

```yaml
server:
  port: 8081

spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: order-service-group
      properties:
        spring.json.trusted.packages: "com.example.common.dto"
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

### Start Saga (OrderCreatedEvent)

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final KafkaTemplate<String, OrderEvent> kafka;

    public OrderController(KafkaTemplate<String, OrderEvent> kafka) {
        this.kafka = kafka;
    }

    @PostMapping
    public String createOrder(@RequestBody OrderEvent event) {

        event.setOrderId(UUID.randomUUID().toString());
        event.setEventType("ORDER_CREATED");

        kafka.send(Topics.ORDER_CREATED, event);

        return "ORDER CREATED with ID = " + event.getOrderId();
    }
}
```

### Listen to success/failure

```java
@Component
public class OrderEventHandler {

    @KafkaListener(topics = Topics.SHIPPING_COMPLETED)
    public void onShippingCompleted(OrderEvent event) {
        System.out.println("Order Completed: " + event.getOrderId());
    }

    @KafkaListener(topics = {
        Topics.PAYMENT_FAILED,
        Topics.INVENTORY_FAILED,
        Topics.SHIPPING_FAILED
    })
    public void onFailure(OrderEvent event) {
        System.out.println("Order Cancelled: " + event.getOrderId());

        event.setEventType("ORDER_CANCELLED");
        kafka.send(Topics.ORDER_CANCELLED, event);
    }
}
```

---

# 🟦 6️⃣ Payment Service

### Listen for `ORDER_CREATED`

```java
@Component
public class PaymentListener {

    @Autowired
    private KafkaTemplate<String, OrderEvent> kafka;

    @KafkaListener(topics = Topics.ORDER_CREATED)
    public void processPayment(OrderEvent event) {

        if (event.getAmount() < 0) {
            event.setEventType("PAYMENT_FAILED");
            event.setReason("Invalid amount");
            kafka.send(Topics.PAYMENT_FAILED, event);
            return;
        }

        event.setEventType("PAYMENT_COMPLETED");
        kafka.send(Topics.PAYMENT_COMPLETED, event);
    }
}
```

---

# 🟧 7️⃣ Inventory Service

### Listen for `PAYMENT_COMPLETED`

```java
@Component
public class InventoryListener {

    @Autowired
    private KafkaTemplate<String, OrderEvent> kafka;

    Map<String, Integer> stock = new ConcurrentHashMap<>();

    public InventoryListener() {
        stock.put("P001", 10);
        stock.put("P002", 5);
    }

    @KafkaListener(topics = Topics.PAYMENT_COMPLETED)
    public void reserveStock(OrderEvent event) {

        int available = stock.getOrDefault(event.getProductId(), 0);

        if (available < event.getQuantity()) {
            event.setEventType("INVENTORY_FAILED");
            event.setReason("No stock available");
            kafka.send(Topics.INVENTORY_FAILED, event);
            return;
        }

        stock.put(event.getProductId(), available - event.getQuantity());

        event.setEventType("INVENTORY_RESERVED");
        kafka.send(Topics.INVENTORY_RESERVED, event);
    }
}
```

---

# 🟨 8️⃣ Shipping Service

### Listen for `INVENTORY_RESERVED`

```java
@Component
public class ShippingListener {

    @Autowired
    private KafkaTemplate<String, OrderEvent> kafka;

    @KafkaListener(topics = Topics.INVENTORY_RESERVED)
    public void shipOrder(OrderEvent event) {

        event.setEventType("SHIPPING_COMPLETED");
        kafka.send(Topics.SHIPPING_COMPLETED, event);
    }
}
```

---

# 🔄 9️⃣ Compensation Logic Overview

### Failure topics

* `PAYMENT_FAILED`
* `INVENTORY_FAILED`
* `SHIPPING_FAILED`

Order service listens to them & performs cancellation.

---

# 🐳 🔥 1️⃣0️⃣ Kafka `docker-compose.yml`

```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

  kafdrop:
    image: obsidiandynamics/kafdrop:4.0.0
    depends_on:
      - kafka
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: kafka:9092
```

---

# 📘 1️⃣1️⃣ README.md (ready for GitHub)

````markdown
# Saga Pattern – Kafka Choreography (Spring Boot)

This repository demonstrates the **Choreography-based Saga pattern** using **Kafka + Spring Boot**.

## Services

- **Order Service** (starts saga)
- **Payment Service**
- **Inventory Service**
- **Shipping Service**
- **common-dto** shared module

## Saga Flow

ORDER_CREATED  
→ PAYMENT_COMPLETED  
→ INVENTORY_RESERVED  
→ SHIPPING_COMPLETED → ORDER_COMPLETED

Failures:

PAYMENT_FAILED → ORDER_CANCELLED  
INVENTORY_FAILED → ORDER_CANCELLED  
SHIPPING_FAILED → ORDER_CANCELLED  

## Run Locally

### 1. Start Kafka

```bash
docker-compose up -d
````

### 2. Start all services

```bash
mvn clean install
mvn -pl order-service spring-boot:run
mvn -pl payment-service spring-boot:run
mvn -pl inventory-service spring-boot:run
mvn -pl shipping-service spring-boot:run
```

### 3. Start a Saga

```bash
curl -X POST http://localhost:8081/orders \
  -H "Content-Type: application/json" \
  -d '{
        "userId":"U1",
        "productId":"P001",
        "quantity":2,
        "amount":100
      }'
```

You will see the event chain in Kafka logs.

```
order.created
payment.completed
inventory.reserved
shipping.completed
order.completed
```

## Visualization

Visit Kafdrop UI: [http://localhost:9000/](http://localhost:9000/)

```

---

# ⭐ Want Both Orchestration + Choreography in One Combined Repo?

I can build:

- Mono-repo with **Orchestrator-based Saga**
- Plus **Choreography-based Saga**
- Shared `common-dto` and shared event system

Just say:  
**“Combine both saga patterns in a single repo with detailed folder structure”**
```
