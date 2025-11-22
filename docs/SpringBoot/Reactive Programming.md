**Spring WebFlux Functional Routing**, we don’t use controllers (`@RestController`) like in Spring MVC. Instead, we define **handlers** (equivalent to controllers) and **router functions** (equivalent to `@RequestMapping` mappings).

If your MVC app has **multiple controllers**, you can achieve the same modular structure by defining **multiple handler classes** and wiring them together in the **Router Configuration**.

Below is a **production-grade example** showing how to organize and handle **multiple controllers (handlers)** in a reactive way using functional routing.

---

## 🧩 Folder Structure

```
com.example.webfluxmulti
│
├── handler
│   ├── CustomerHandler.java
│   └── OrderHandler.java
│
├── router
│   └── AppRouter.java
│
├── model
│   ├── Customer.java
│   └── Order.java
│
├── repository
│   ├── CustomerRepository.java
│   └── OrderRepository.java
│
└── WebfluxMultiApplication.java
```

---

## 🧠 1. CustomerHandler.java

```java
package com.example.webfluxmulti.handler;

import com.example.webfluxmulti.model.Customer;
import com.example.webfluxmulti.repository.CustomerRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Component
public class CustomerHandler {

    @Autowired
    private CustomerRepository customerRepository;

    public Mono<ServerResponse> getAllCustomers(ServerRequest request) {
        Flux<Customer> customers = customerRepository.findAll();
        return ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(customers, Customer.class);
    }

    public Mono<ServerResponse> getCustomerById(ServerRequest request) {
        String id = request.pathVariable("id");
        Mono<Customer> customerMono = customerRepository.findById(id);
        return customerMono
                .flatMap(customer -> ServerResponse.ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .bodyValue(customer))
                .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> createCustomer(ServerRequest request) {
        Mono<Customer> customerMono = request.bodyToMono(Customer.class);
        return customerMono.flatMap(customerRepository::save)
                .flatMap(saved -> ServerResponse.ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .bodyValue(saved));
    }
}
```

---

## 📦 2. OrderHandler.java

```java
package com.example.webfluxmulti.handler;

import com.example.webfluxmulti.model.Order;
import com.example.webfluxmulti.repository.OrderRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Component
public class OrderHandler {

    @Autowired
    private OrderRepository orderRepository;

    public Mono<ServerResponse> getAllOrders(ServerRequest request) {
        Flux<Order> orders = orderRepository.findAll();
        return ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(orders, Order.class);
    }

    public Mono<ServerResponse> getOrderById(ServerRequest request) {
        String id = request.pathVariable("id");
        Mono<Order> orderMono = orderRepository.findById(id);
        return orderMono
                .flatMap(order -> ServerResponse.ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .bodyValue(order))
                .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> createOrder(ServerRequest request) {
        Mono<Order> orderMono = request.bodyToMono(Order.class);
        return orderMono.flatMap(orderRepository::save)
                .flatMap(saved -> ServerResponse.ok()
                        .contentType(MediaType.APPLICATION_JSON)
                        .bodyValue(saved));
    }
}
```

---

## 🛣️ 3. AppRouter.java (Router Configuration)

```java
package com.example.webfluxmulti.router;

import com.example.webfluxmulti.handler.CustomerHandler;
import com.example.webfluxmulti.handler.OrderHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.RouterFunctions;
import org.springframework.web.reactive.function.server.ServerResponse;

import static org.springframework.web.reactive.function.server.RequestPredicates.*;

@Configuration
public class AppRouter {

    @Bean
    public RouterFunction<ServerResponse> routes(CustomerHandler customerHandler, OrderHandler orderHandler) {

        return RouterFunctions
                // 👇 Customer routes
                .nest(path("/api/customers"),
                        RouterFunctions.route(GET(""), customerHandler::getAllCustomers)
                                .andRoute(GET("/{id}"), customerHandler::getCustomerById)
                                .andRoute(POST(""), customerHandler::createCustomer)
                )
                // 👇 Order routes
                .andNest(path("/api/orders"),
                        RouterFunctions.route(GET(""), orderHandler::getAllOrders)
                                .andRoute(GET("/{id}"), orderHandler::getOrderById)
                                .andRoute(POST(""), orderHandler::createOrder)
                );
    }
}
```

---

## 🚀 4. WebfluxMultiApplication.java

```java
package com.example.webfluxmulti;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class WebfluxMultiApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebfluxMultiApplication.class, args);
    }
}
```

---

## ✅ Highlights

|MVC Approach|WebFlux Functional Equivalent|
|---|---|
|`@RestController`|`@Component` Handler|
|`@RequestMapping("/api/customers")`|`RouterFunctions.route(path("/api/customers"), …)`|
|`@GetMapping`, `@PostMapping`|`andRoute(GET(""), handler::method)`|
|`ResponseEntity`|`ServerResponse`|
|`@Autowired` Service|Reactive Repositories or Services injected|

---

## 🧩 Why This Design Is Better

- **Decoupled handlers** → Each domain (Customer, Order) has its own logic.
    
- **Reactive & Non-blocking** → Fully async I/O with Project Reactor.
    
- **Testable** → Each `Handler` and `RouterFunction` can be unit tested independently.
    
- **Production-grade scalability** using **Netty event loop model**.
    

---

