I have created a **ready-to-paste, minimal but production-minded codebase** for an **Order → Inventory** flow using **OpenFeign + Resilience4j**.  
Two simple Spring Boot services:

- `inventory-service` (port **8081**) — provides inventory info (with caching).
    
- `order-service` (port **8080**) — calls `inventory-service` via **Feign** and protects the call with **Resilience4j** (`@CircuitBreaker`, `@Retry`, `@TimeLimiter`) plus a graceful fallback.
    

No service registry required (Feign uses a URL property), so you can run locally easily. I included Maven `pom.xml` files, `application.yml` configs, Java sources, and a README with example curl commands.

---

## Project layout (3 modules / directories)

```
order-inventory-sample/
├─ common/                    # shared DTOs
├─ inventory-service/         # Inventory microservice (8081)
└─ order-service/             # Order microservice (8080) -> Feign + Resilience4j
```

---

# 1) `common` module (shared DTOs)

**pom.xml**

```xml
<!-- common/pom.xml -->
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>common</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  <properties>
    <java.version>17</java.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <scope>provided</scope>
    </dependency>
  </dependencies>
</project>
```

**src/main/java/com/example/common/dto/InventoryDto.java**

```java
package com.example.common.dto;

public class InventoryDto {
    private String productId;
    private int available;

    public InventoryDto() {}

    public InventoryDto(String productId, int available) {
        this.productId = productId;
        this.available = available;
    }

    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    public int getAvailable() { return available; }
    public void setAvailable(int available) { this.available = available; }

    @Override
    public String toString() {
        return "InventoryDto{productId='" + productId + "', available=" + available + '}';
    }
}
```

**src/main/java/com/example/common/dto/OrderRequest.java**

```java
package com.example.common.dto;

public class OrderRequest {
    private String productId;
    private int qty;

    public OrderRequest() {}
    public OrderRequest(String productId, int qty) {
        this.productId = productId;
        this.qty = qty;
    }

    public String getProductId() { return productId; }
    public void setProductId(String productId) { this.productId = productId; }
    public int getQty() { return qty; }
    public void setQty(int qty) { this.qty = qty; }
}
```

**src/main/java/com/example/common/dto/OrderConfirmation.java**

```java
package com.example.common.dto;

public class OrderConfirmation {
    private String orderId;
    private String status;
    private String message;

    public OrderConfirmation() {}
    public OrderConfirmation(String orderId, String status, String message) {
        this.orderId = orderId;
        this.status = status;
        this.message = message;
    }

    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }

    @Override
    public String toString() {
        return "OrderConfirmation{orderId='" + orderId + "', status='" + status + "', message='" + message + "'}";
    }
}
```

---

# 2) `inventory-service` (simple service with caching)

**pom.xml**

```xml
<!-- inventory-service/pom.xml -->
<project ...>
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
  </parent>
  <groupId>com.example</groupId>
  <artifactId>inventory-service</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <properties>
    <java.version>17</java.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>com.example</groupId>
      <artifactId>common</artifactId>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
  </dependencies>
</project>
```

**src/main/resources/application.yml**

```yaml
server:
  port: 8081

spring:
  cache:
    type: simple   # use simple cache for demo; switch to redis if you have redis
```

**src/main/java/com/example/inventory/InventoryServiceApplication.java**

```java
package com.example.inventory;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class InventoryServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(InventoryServiceApplication.class, args);
    }
}
```

**src/main/java/com/example/inventory/controller/InventoryController.java**

```java
package com.example.inventory.controller;

import com.example.common.dto.InventoryDto;
import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;

@RestController
@RequestMapping("/api/inventory")
public class InventoryController {

    // simple in-memory simulated inventory
    @GetMapping("/{productId}")
    public ResponseEntity<InventoryDto> getInventory(@PathVariable String productId) {
        // In real app: query DB or service; here, we simulate
        int available = "p1".equals(productId) ? 10 : 0;
        InventoryDto dto = new InventoryDto(productId, available);
        return ResponseEntity.ok(dto);
    }
}
```

_(Optional)_ If you want caching in inventory (for demo simplicity, `@Cacheable` on controller or service). For a demo simple cache we left it without cache but enabled caching support.

---

# 3) `order-service` (Feign + Resilience4j + fallback)

**pom.xml**

```xml
<!-- order-service/pom.xml -->
<project ...>
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
  </parent>

  <groupId>com.example</groupId>
  <artifactId>order-service</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <properties>
    <java.version>17</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>com.example</groupId>
      <artifactId>common</artifactId>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- OpenFeign -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
      <version>4.0.3</version>
    </dependency>

    <!-- Resilience4j for Spring Boot 3 -->
    <dependency>
      <groupId>io.github.resilience4j</groupId>
      <artifactId>resilience4j-spring-boot3</artifactId>
      <version>2.0.2</version>
    </dependency>

    <!-- For async TimeLimiter example (optional) -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>

    <!-- Optional: actuator for metrics -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
  </dependencies>

  <repositories>
    <repository>
      <id>spring-milestones</id>
      <url>https://repo.spring.io/milestone</url>
    </repository>
  </repositories>
</project>
```

**src/main/resources/application.yml**

```yaml
server:
  port: 8080

spring:
  application:
    name: order-service

# Feign: point to inventory service running on localhost:8081
inventory:
  service:
    url: http://localhost:8081

feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000

resilience4j:
  circuitbreaker:
    instances:
      inventoryService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 3
        failureRateThreshold: 50
        waitDurationInOpenState: 15s
  retry:
    instances:
      inventoryService:
        maxAttempts: 3
        waitDuration: 1s
  timelimiter:
    instances:
      inventoryService:
        timeoutDuration: 2s

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
```

**src/main/java/com/example/order/OrderServiceApplication.java**

```java
package com.example.order;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

**src/main/java/com/example/order/feign/InventoryClient.java**

```java
package com.example.order.feign;

import com.example.common.dto.InventoryDto;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "inventory-service", url = "${inventory.service.url}")
public interface InventoryClient {

    @GetMapping("/api/inventory/{productId}")
    InventoryDto getInventory(@PathVariable("productId") String productId);
}
```

**src/main/java/com/example/order/service/OrderService.java**

```java
package com.example.order.service;

import com.example.common.dto.InventoryDto;
import com.example.common.dto.OrderConfirmation;
import com.example.common.dto.OrderRequest;
import com.example.order.feign.InventoryClient;
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.retry.annotation.Retry;
import io.github.resilience4j.timelimiter.annotation.TimeLimiter;
import org.springframework.stereotype.Service;

import java.util.UUID;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CompletionStage;

@Service
public class OrderService {

    private final InventoryClient inventoryClient;

    public OrderService(InventoryClient inventoryClient) {
        this.inventoryClient = inventoryClient;
    }

    /**
     * Primary method used by controller. It's async to demonstrate TimeLimiter usage.
     * It is protected by:
     *  - @TimeLimiter (timeout)
     *  - @CircuitBreaker (circuit breaker + fallback)
     *  - @Retry (retries before fallback)
     */
    @TimeLimiter(name = "inventoryService")
    @CircuitBreaker(name = "inventoryService", fallbackMethod = "inventoryFallback")
    @Retry(name = "inventoryService")
    public CompletableFuture<OrderConfirmation> placeOrderAsync(OrderRequest req) {
        return CompletableFuture.supplyAsync(() -> placeOrderSync(req));
    }

    // synchronous internal implementation
    public OrderConfirmation placeOrderSync(OrderRequest req) {
        InventoryDto inv = inventoryClient.getInventory(req.getProductId());
        if (inv == null) {
            return new OrderConfirmation(null, "FAILED", "Inventory service returned null");
        }
        if (inv.getAvailable() < req.getQty()) {
            return new OrderConfirmation(null, "FAILED", "Insufficient stock");
        }
        // create an order id and return success
        String orderId = UUID.randomUUID().toString();
        return new OrderConfirmation(orderId, "CONFIRMED", "Order placed successfully");
    }

    // Fallback signature for CompletableFuture methods: must return CompletionStage or CompletableFuture
    private CompletableFuture<OrderConfirmation> inventoryFallback(OrderRequest req, Throwable ex) {
        // fallback logic: return a friendly response or use cached data
        OrderConfirmation fallback = new OrderConfirmation(null, "FALLBACK", "Inventory service unavailable - try later");
        return CompletableFuture.completedFuture(fallback);
    }
}
```

**src/main/java/com/example/order/controller/OrderController.java**

```java
package com.example.order.controller;

import com.example.common.dto.OrderConfirmation;
import com.example.common.dto.OrderRequest;
import com.example.order.service.OrderService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.concurrent.CompletableFuture;

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) { this.orderService = orderService; }

    @PostMapping
    public CompletableFuture<ResponseEntity<OrderConfirmation>> placeOrder(@RequestBody OrderRequest req) {
        return orderService.placeOrderAsync(req)
                .thenApply(ResponseEntity::ok)
                .exceptionally(ex -> ResponseEntity.internalServerError().body(
                        new OrderConfirmation(null, "ERROR", "Unexpected error: " + ex.getMessage())
                ));
    }

    @GetMapping("/ping")
    public String ping() { return "order-service up"; }
}
```

---

## How to run (locally)

1. Build the `common` and services (run from `order-inventory-sample/`):
    

```bash
# from project root (if you created a multi-module aggregator, otherwise build each module):
# build common jar first (so other modules can use it)
cd common
mvn clean install

# build inventory-service
cd ../inventory-service
mvn clean package

# build order-service
cd ../order-service
mvn clean package
```

2. Start `inventory-service`:
    

```bash
cd inventory-service
mvn spring-boot:run
# or java -jar target/inventory-service-0.0.1-SNAPSHOT.jar
```

3. Start `order-service`:
    

```bash
cd ../order-service
mvn spring-boot:run
# or java -jar target/order-service-0.0.1-SNAPSHOT.jar
```

4. Test with curl:
    

- Inventory (existing product `p1` → 10 available):
    

```bash
curl -s http://localhost:8081/api/inventory/p1 | jq
# {"productId":"p1","available":10}
```

- Place order (succeeds):
    

```bash
curl -s -X POST http://localhost:8080/api/orders -H "Content-Type: application/json" \
  -d '{"productId":"p1","qty":2}' | jq
# { "orderId":"...", "status":"CONFIRMED", "message":"Order placed successfully" }
```

- Place order for out-of-stock:
    

```bash
curl -s -X POST http://localhost:8080/api/orders -H "Content-Type: application/json" \
  -d '{"productId":"p2","qty":1}' | jq
# { "orderId": null, "status":"FAILED", "message":"Insufficient stock" }
```

- Simulate inventory down: stop inventory service and call order -> you'll see fallback:
    

```bash
# stop inventory-service and call:
curl -s -X POST http://localhost:8080/api/orders -H "Content-Type: application/json" \
  -d '{"productId":"p1","qty":1}' | jq
# { "orderId": null, "status":"FALLBACK", "message":"Inventory service unavailable - try later" }
```

---

## Notes, best practices & next steps

- **Resilience4j**: configuration is in `order-service/application.yml`. Tweak `slidingWindowSize`, `failureRateThreshold`, `waitDurationInOpenState`, `retry.maxAttempts` per real traffic patterns.
    
- **TimeLimiter**: shown as async usage. If you prefer synchronous code, remove `@TimeLimiter` and make fallback signatures synchronous (`OrderConfirmation inventoryFallback(OrderRequest, Throwable)`).
    
- **Feign**: in production you’ll use service discovery (Eureka/Consul) or load balancer (Spring Cloud LoadBalancer). Replace `url` in `@FeignClient` with `lb://inventory-service` and enable discovery.
    
- **Caching**: you can add Redis/Caffeine to cache inventory responses in `inventory-service` (annotate `getInventory` with `@Cacheable`).
    
- **Observability**: add `spring-boot-starter-actuator`, Micrometer Prometheus registry, and Feign / Resilience4j metrics exporters.
    
- **Tests**: create unit tests for the `OrderService` using Mockito to mock `InventoryClient`, and integration tests using Testcontainers for an optional Redis or for real network path checks.
    

---
