Nice, let’s build a **separate Kafka-based orchestration repo**.

I’ll give you everything as if it’s a real Git repo: structure, key `pom.xml`s, core Kafka saga flow, and `docker-compose` for Kafka.

---

## 1️⃣ New Repo: `saga-kafka-orchestration-spring`

```text
saga-kafka-orchestration-spring/
├─ README.md
├─ pom.xml                          # parent pom (multi-module)
├─ docker-compose.yml               # Kafka + UI
│
├─ common-dto/
│  ├─ pom.xml
│  └─ src/main/java/com/example/common/dto/...
│
├─ orchestrator-kafka-service/
│  ├─ pom.xml
│  └─ src/main/java/com/example/orchestrator/...
│
├─ order-service-kafka/
│  ├─ pom.xml
│  └─ src/main/java/com/example/order/...
│
├─ payment-service-kafka/
│  ├─ pom.xml
│  └─ src/main/java/com/example/payment/...
│
├─ inventory-service-kafka/
│  ├─ pom.xml
│  └─ src/main/java/com/example/inventory/...
└─ shipping-service-kafka/
   ├─ pom.xml
   └─ src/main/java/com/example/shipping/...
```

---

## 2️⃣ Parent `pom.xml` (root)

```xml
<!-- saga-kafka-orchestration-spring/pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>saga-kafka-orchestration-spring</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>common-dto</module>
        <module>orchestrator-kafka-service</module>
        <module>order-service-kafka</module>
        <module>payment-service-kafka</module>
        <module>inventory-service-kafka</module>
        <module>shipping-service-kafka</module>
    </modules>

    <properties>
        <java.version>17</java.version>
        <spring.boot.version>3.3.4</spring.boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

---

## 3️⃣ `common-dto` module

### `common-dto/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>saga-kafka-orchestration-spring</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>common-dto</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.34</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```

### DTOs

```java
// common-dto/src/main/java/com/example/common/dto/OrderSagaPayload.java
package com.example.common.dto;

import lombok.Data;

@Data
public class OrderSagaPayload {
    private String sagaId;
    private String orderId;
    private String userId;
    private String productId;
    private int quantity;
    private double amount;
    private String step;     // e.g. "ORDER_CREATED", "PAYMENT_SUCCESS", etc.
    private String status;   // e.g. "SUCCESS", "FAILED"
    private String reason;   // failure reason if any
}
```

```java
// common-dto/src/main/java/com/example/common/dto/TopicNames.java
package com.example.common.dto;

public final class TopicNames {

    private TopicNames() {}

    // Commands
    public static final String ORDER_CREATE_CMD     = "order.command.create";
    public static final String ORDER_CANCEL_CMD     = "order.command.cancel";
    public static final String PAYMENT_PROCESS_CMD  = "payment.command.process";
    public static final String PAYMENT_REFUND_CMD   = "payment.command.refund";
    public static final String INV_RESERVE_CMD      = "inventory.command.reserve";
    public static final String INV_REVERT_CMD       = "inventory.command.revert";
    public static final String SHIPPING_SHIP_CMD    = "shipping.command.ship";

    // Events
    public static final String ORDER_EVENT          = "order.event";
    public static final String PAYMENT_EVENT        = "payment.event";
    public static final String INVENTORY_EVENT      = "inventory.event";
    public static final String SHIPPING_EVENT       = "shipping.event";
}
```

---

## 4️⃣ Orchestrator Kafka Service

### `orchestrator-kafka-service/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>saga-kafka-orchestration-spring</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>orchestrator-kafka-service</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common-dto</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
</project>
```

### Main class

```java
// orchestrator-kafka-service/src/main/java/com/example/orchestrator/OrchestratorKafkaApplication.java
package com.example.orchestrator;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrchestratorKafkaApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrchestratorKafkaApplication.class, args);
    }
}
```

### `application.yml`

```yaml
# orchestrator-kafka-service/src/main/resources/application.yml
server:
  port: 8080

spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: orchestrator-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "com.example.common.dto"
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

### Kafka Config

```java
// orchestrator-kafka-service/src/main/java/com/example/orchestrator/config/KafkaConfig.java
package com.example.orchestrator.config;

import com.example.common.dto.OrderSagaPayload;
import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;

import static com.example.common.dto.TopicNames.*;

@Configuration
public class KafkaConfig {

    @Bean
    public NewTopic orderCommandCreate() {
        return TopicBuilder.name(ORDER_CREATE_CMD).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic orderCommandCancel() {
        return TopicBuilder.name(ORDER_CANCEL_CMD).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic paymentProcessCmd() {
        return TopicBuilder.name(PAYMENT_PROCESS_CMD).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic paymentRefundCmd() {
        return TopicBuilder.name(PAYMENT_REFUND_CMD).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic invReserveCmd() {
        return TopicBuilder.name(INV_RESERVE_CMD).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic invRevertCmd() {
        return TopicBuilder.name(INV_REVERT_CMD).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic shipCmd() {
        return TopicBuilder.name(SHIPPING_SHIP_CMD).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic orderEventTopic() {
        return TopicBuilder.name(ORDER_EVENT).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic paymentEventTopic() {
        return TopicBuilder.name(PAYMENT_EVENT).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic inventoryEventTopic() {
        return TopicBuilder.name(INVENTORY_EVENT).partitions(3).replicas(1).build();
    }

    @Bean
    public NewTopic shippingEventTopic() {
        return TopicBuilder.name(SHIPPING_EVENT).partitions(3).replicas(1).build();
    }
}
```

### Orchestrator Service (driving the saga)

```java
// orchestrator-kafka-service/src/main/java/com/example/orchestrator/service/OrderSagaOrchestrator.java
package com.example.orchestrator.service;

import com.example.common.dto.OrderSagaPayload;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

import static com.example.common.dto.TopicNames.*;

@Service
public class OrderSagaOrchestrator {

    private final KafkaTemplate<String, OrderSagaPayload> kafkaTemplate;

    // simple in-memory saga store (for demo)
    private final Map<String, String> sagaStatus = new ConcurrentHashMap<>();

    public OrderSagaOrchestrator(KafkaTemplate<String, OrderSagaPayload> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public String startSaga(OrderSagaPayload payload) {
        String sagaId = UUID.randomUUID().toString();
        String orderId = UUID.randomUUID().toString();

        payload.setSagaId(sagaId);
        payload.setOrderId(orderId);
        payload.setStep("SAGA_STARTED");
        payload.setStatus("PENDING");

        sagaStatus.put(sagaId, "STARTED");

        // Step 1: tell Order Service to create order
        kafkaTemplate.send(ORDER_CREATE_CMD, sagaId, payload);

        return sagaId;
    }

    // 🔁 Orchestrator reacting to events

    @KafkaListener(topics = ORDER_EVENT, groupId = "orchestrator-group")
    public void onOrderEvent(OrderSagaPayload event) {
        if (!event.getStatus().equalsIgnoreCase("SUCCESS")) {
            sagaStatus.put(event.getSagaId(), "FAILED_AT_ORDER");
            // Nothing to compensate before order
            return;
        }

        // Next step: Payment
        event.setStep("PAYMENT_PROCESS");
        kafkaTemplate.send(PAYMENT_PROCESS_CMD, event.getSagaId(), event);
    }

    @KafkaListener(topics = PAYMENT_EVENT, groupId = "orchestrator-group")
    public void onPaymentEvent(OrderSagaPayload event) {
        if (event.getStatus().equalsIgnoreCase("SUCCESS")) {
            event.setStep("INVENTORY_RESERVE");
            kafkaTemplate.send(INV_RESERVE_CMD, event.getSagaId(), event);
        } else {
            // compensate: cancel order
            event.setStep("ORDER_CANCEL");
            kafkaTemplate.send(ORDER_CANCEL_CMD, event.getSagaId(), event);
            sagaStatus.put(event.getSagaId(), "FAILED_AT_PAYMENT");
        }
    }

    @KafkaListener(topics = INVENTORY_EVENT, groupId = "orchestrator-group")
    public void onInventoryEvent(OrderSagaPayload event) {
        if (event.getStatus().equalsIgnoreCase("SUCCESS")) {
            event.setStep("SHIPPING_SHIP");
            kafkaTemplate.send(SHIPPING_SHIP_CMD, event.getSagaId(), event);
        } else {
            // compensate: refund payment + cancel order
            event.setStep("PAYMENT_REFUND");
            kafkaTemplate.send(PAYMENT_REFUND_CMD, event.getSagaId(), event);

            event.setStep("ORDER_CANCEL");
            kafkaTemplate.send(ORDER_CANCEL_CMD, event.getSagaId(), event);

            sagaStatus.put(event.getSagaId(), "FAILED_AT_INVENTORY");
        }
    }

    @KafkaListener(topics = SHIPPING_EVENT, groupId = "orchestrator-group")
    public void onShippingEvent(OrderSagaPayload event) {
        if (event.getStatus().equalsIgnoreCase("SUCCESS")) {
            sagaStatus.put(event.getSagaId(), "COMPLETED");
        } else {
            // compensate: refund + revert + cancel
            event.setStep("PAYMENT_REFUND");
            kafkaTemplate.send(PAYMENT_REFUND_CMD, event.getSagaId(), event);

            event.setStep("INVENTORY_REVERT");
            kafkaTemplate.send(INV_REVERT_CMD, event.getSagaId(), event);

            event.setStep("ORDER_CANCEL");
            kafkaTemplate.send(ORDER_CANCEL_CMD, event.getSagaId(), event);

            sagaStatus.put(event.getSagaId(), "FAILED_AT_SHIPPING");
        }
    }

    public String getSagaStatus(String sagaId) {
        return sagaStatus.getOrDefault(sagaId, "UNKNOWN");
    }
}
```

### REST Controller to start/check saga

```java
// orchestrator-kafka-service/src/main/java/com/example/orchestrator/controller/SagaController.java
package com.example.orchestrator.controller;

import com.example.common.dto.OrderSagaPayload;
import com.example.orchestrator.service.OrderSagaOrchestrator;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/saga")
public class SagaController {

    private final OrderSagaOrchestrator orchestrator;

    public SagaController(OrderSagaOrchestrator orchestrator) {
        this.orchestrator = orchestrator;
    }

    @PostMapping("/order")
    public String startOrderSaga(@RequestBody OrderSagaPayload payload) {
        return orchestrator.startSaga(payload);
    }

    @GetMapping("/{sagaId}")
    public String getSagaStatus(@PathVariable String sagaId) {
        return orchestrator.getSagaStatus(sagaId);
    }
}
```

---

## 5️⃣ Example: Order Service (Kafka-based)

You’ll repeat similar patterns for payment, inventory, shipping.

### `order-service-kafka/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>saga-kafka-orchestration-spring</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>order-service-kafka</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common-dto</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
</project>
```

### Main + Kafka Listener

```java
// order-service-kafka/src/main/java/com/example/order/OrderServiceKafkaApplication.java
package com.example.order;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderServiceKafkaApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceKafkaApplication.class, args);
    }
}
```

```yaml
# order-service-kafka/src/main/resources/application.yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: order-service-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: "com.example.common.dto"
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

```java
// order-service-kafka/src/main/java/com/example/order/kafka/OrderCommandListener.java
package com.example.order.kafka;

import com.example.common.dto.OrderSagaPayload;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

import java.util.concurrent.ConcurrentHashMap;
import java.util.Map;

import static com.example.common.dto.TopicNames.*;

@Component
public class OrderCommandListener {

    private final KafkaTemplate<String, OrderSagaPayload> kafkaTemplate;
    private final Map<String, String> orderStatus = new ConcurrentHashMap<>();

    public OrderCommandListener(KafkaTemplate<String, OrderSagaPayload> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    @KafkaListener(topics = ORDER_CREATE_CMD, groupId = "order-service-group")
    public void handleCreate(OrderSagaPayload payload) {
        // local transaction (for demo: simple in-memory)
        orderStatus.put(payload.getOrderId(), "CREATED");
        System.out.println("Order created: " + payload.getOrderId());

        payload.setStep("ORDER_CREATED");
        payload.setStatus("SUCCESS");

        kafkaTemplate.send(ORDER_EVENT, payload.getSagaId(), payload);
    }

    @KafkaListener(topics = ORDER_CANCEL_CMD, groupId = "order-service-group")
    public void handleCancel(OrderSagaPayload payload) {
        orderStatus.put(payload.getOrderId(), "CANCELLED");
        System.out.println("Order cancelled: " + payload.getOrderId());

        payload.setStep("ORDER_CANCELLED");
        payload.setStatus("SUCCESS");

        kafkaTemplate.send(ORDER_EVENT, payload.getSagaId(), payload);
    }
}
```

---

## 6️⃣ Payment / Inventory / Shipping Services (pattern)

Each is similar:

* Listen for its **command topic**
* Execute local logic
* Publish **event topic** back

### Payment example (only core class)

```java
// payment-service-kafka/.../PaymentCommandListener.java
package com.example.payment.kafka;

import com.example.common.dto.OrderSagaPayload;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

import static com.example.common.dto.TopicNames.*;

@Component
public class PaymentCommandListener {

    private final KafkaTemplate<String, OrderSagaPayload> kafkaTemplate;

    public PaymentCommandListener(KafkaTemplate<String, OrderSagaPayload> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    @KafkaListener(topics = PAYMENT_PROCESS_CMD, groupId = "payment-service-group")
    public void handlePayment(OrderSagaPayload payload) {
        // Simple rule: if amount < 0 -> fail
        if (payload.getAmount() < 0) {
            payload.setStatus("FAILED");
            payload.setStep("PAYMENT_FAILED");
            payload.setReason("Amount negative");
        } else {
            payload.setStatus("SUCCESS");
            payload.setStep("PAYMENT_SUCCESS");
        }

        kafkaTemplate.send(PAYMENT_EVENT, payload.getSagaId(), payload);
    }

    @KafkaListener(topics = PAYMENT_REFUND_CMD, groupId = "payment-service-group")
    public void handleRefund(OrderSagaPayload payload) {
        System.out.println("Refund processed for order: " + payload.getOrderId());
        payload.setStatus("SUCCESS");
        payload.setStep("PAYMENT_REFUNDED");
        kafkaTemplate.send(PAYMENT_EVENT, payload.getSagaId(), payload);
    }
}
```

Inventory and shipping services follow same pattern: consume command, do local business rule, send event.

---

## 7️⃣ `docker-compose.yml` for Kafka

```yaml
# saga-kafka-orchestration-spring/docker-compose.yml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.6.0
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
    image: obsidiandynamics/kafdrop:4.0.1
    depends_on:
      - kafka
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: kafka:9092
```

Run:

```bash
docker-compose up -d
```

---

## 8️⃣ How to run the whole saga

1. Build everything:

```bash
mvn clean install
```

2. Start Kafka stack:

```bash
docker-compose up -d
```

3. Run each service (separate terminals):

```bash
cd order-service-kafka && mvn spring-boot:run
cd payment-service-kafka && mvn spring-boot:run
cd inventory-service-kafka && mvn spring-boot:run
cd shipping-service-kafka && mvn spring-boot:run
cd orchestrator-kafka-service && mvn spring-boot:run
```

4. Start a saga:

```bash
curl -X POST http://localhost:8080/saga/order \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "U123",
    "productId": "P001",
    "quantity": 2,
    "amount": 1000.0
  }'
```

Response: `"<sagaId>"`

5. Check status:

```bash
curl http://localhost:8080/saga/<sagaId>
```

---

## 9️⃣ Git commands for **this** repo

```bash
cd saga-kafka-orchestration-spring
git init
git add .
git commit -m "Initial commit - Kafka-based Saga Orchestration"

git remote add origin https://github.com/<your-username>/saga-kafka-orchestration-spring.git
git branch -M main
git push -u origin main
```

---
