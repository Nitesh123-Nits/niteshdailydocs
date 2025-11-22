**Fluent APIs** in _real-world Java frameworks_ (like Stream API, Optional, Spring, Reactor, Hibernate, etc.).

Let’s structure your next section as a **continuation of your Functional Programming guide**, titled something like:

> 🔄 **Functional Programming & Fluent API in Action — Real-World Java Examples**

Below is a complete breakdown of how you can organize it — with code examples, explanations, and why each approach improves readability, maintainability, and expressiveness.

---

## 🔄 Functional Programming & Fluent API in Action

### **1. What is a Fluent API?**

A **Fluent API** is a design style that allows **method chaining** to express operations in a _flowing, readable, and declarative way_.

It often uses **method chaining** + **functional constructs (lambdas, method references)** → to build expressive, natural language-like code.

**Example:**

```java
List<String> result = names.stream()
                           .filter(n -> n.startsWith("A"))
                           .map(String::toUpperCase)
                           .sorted()
                           .toList();
```

👉 Here, the **Stream API** is both _functional_ and _fluent_.

---

### **2. Java Stream API — The Core Fluent FP Example**

The **Stream API** is the best demonstration of FP + Fluent design.

#### 💡 Functional Concepts Used:

- **Higher-order functions**: `.map()`, `.filter()`, `.reduce()` accept lambdas.
    
- **Immutability**: Streams don’t modify the source.
    
- **Declarative**: Focus on _what_, not _how_.
    

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

int sum = numbers.stream()
                 .filter(n -> n % 2 == 0)
                 .mapToInt(n -> n * n)
                 .sum();
```

**Why Fluent?**  
Each method returns a new `Stream`, allowing chaining in a natural flow.

---

### **3. Optional API — Functional and Fluent Error Handling**

The `Optional` class avoids null checks and makes the code declarative.

```java
Optional<String> name = Optional.ofNullable("John");

String result = name.filter(n -> n.length() > 3)
                    .map(String::toUpperCase)
                    .orElse("UNKNOWN");
```

**Functional features:**

- Lambdas for transformation and filtering.
    
- Declarative null handling.
    
- Composable flow.
    

---

### **4. CompletableFuture — Functional Async Programming**

Java’s `CompletableFuture` combines **functional composition** and **fluent chaining** for async tasks.

```java
CompletableFuture.supplyAsync(() -> "Data from API")
    .thenApply(data -> data + " processed")
    .thenAccept(System.out::println)
    .exceptionally(ex -> {
        System.out.println("Error: " + ex.getMessage());
        return null;
    });
```

**Why it’s functional & fluent:**

- Lambdas as callbacks (no anonymous inner classes).
    
- Non-blocking, composable async chains.
    
- Readable "flow" of operations.
    

---

### **5. Spring Framework — Functional Bean Definitions & WebFlux**

Spring Boot 2+ introduced **functional bean registration** and **WebFlux functional endpoints**, fully based on FP and Fluent design.

#### **(a) Functional Bean Definition**

```java
@Configuration
public class AppConfig {
    @Bean
    public Supplier<String> message() {
        return () -> "Hello Functional Spring!";
    }
}
```

#### **(b) Functional WebFlux Router**

```java
@Bean
public RouterFunction<ServerResponse> route() {
    return RouterFunctions.route()
            .GET("/hello", req -> ServerResponse.ok().bodyValue("Hello World"))
            .POST("/echo", req -> req.bodyToMono(String.class)
                    .flatMap(body -> ServerResponse.ok().bodyValue("Echo: " + body)))
            .build();
}
```

**Benefits:**

- No annotations needed.
    
- Composable routing.
    
- Purely functional flow using `Mono`/`Flux`.
    

---

### **6. Project Reactor (Spring WebFlux Core) — Reactive FP**

Reactor introduces `Mono` and `Flux` — completely FP constructs inspired by **Reactive Streams**.

```java
Flux.just(1, 2, 3, 4, 5)
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .doOnNext(System.out::println)
    .subscribe();
```

**Functional concepts:**

- Pure functions (`map`, `filter`, `flatMap`)
    
- Lazy evaluation (nothing executes until `.subscribe()`)
    
- Immutability
    
- Fluent pipeline
    

---

### **7. Hibernate Criteria API (Fluent Query Building)**

Although not functional, Hibernate’s **CriteriaBuilder** is a classical **Fluent API**.

```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Employee> query = cb.createQuery(Employee.class);
Root<Employee> root = query.from(Employee.class);

query.select(root)
     .where(cb.and(
         cb.equal(root.get("department"), "HR"),
         cb.greaterThan(root.get("salary"), 50000)
     ));

List<Employee> result = session.createQuery(query).getResultList();
```

**Why Fluent?**

- Chainable, declarative query construction.
    
- Replaces string-based HQL with type-safe, composable API.
    

---

### **8. Mockito — Fluent Stubbing and Verification**

Mockito uses fluent + functional style for mock behavior configuration.

```java
when(service.getUser(anyString()))
    .thenReturn(new User("John"))
    .thenThrow(new RuntimeException("User not found"));
```

Or verification:

```java
verify(service, times(1)).getUser("John");
```

**Functional flavor:**

- Lambdas for argument matchers.
    
- Fluent style for readable test configurations.
    

---

### **9. Builder Pattern — Fluent Object Creation**

Classic **Fluent Design Pattern**, not necessarily functional but often used alongside FP for readability.

```java
User user = new User.Builder()
    .name("John")
    .email("john@gmail.com")
    .age(28)
    .build();
```

**Functional version using lambdas:**

```java
User user = build(User::new, u -> {
    u.name = "John";
    u.email = "john@gmail.com";
    u.age = 28;
});
```

---

### **10. Lombok, Streams, and Beyond**

Libraries like **Lombok**, **Vavr**, **Reactor**, **RxJava**, and **Spring Data** embrace functional & fluent design:

- **Lombok** reduces boilerplate (functional data handling).
    
- **Vavr** brings FP types: `Try`, `Either`, `Lazy`, `Tuple`, `Option`.
    
- **Spring Data** uses fluent query methods:  
    `findByAgeGreaterThanAndNameStartingWith(...)`.
    

---

### **11. Summary — FP + Fluent = Modern Java Style**

|Concept|FP Element|Fluent Element|Example|
|---|---|---|---|
|Stream API|Lambdas, immutability|Chaining|`filter().map().collect()`|
|Optional|Function composition|Method chaining|`filter().map().orElse()`|
|CompletableFuture|Async composition|`.thenApply()` chain|`supplyAsync().thenApply()`|
|WebFlux|Reactive, Mono/Flux|`.route().GET().build()`||
|Hibernate|Declarative query|`query.select().where()`||

---

### **12. Conclusion**

Functional and Fluent styles have transformed how we write Java:

- **Functional Programming** gives _powerful abstractions_ and _composability_.
    
- **Fluent APIs** provide _readability_ and _natural syntax_.
    
- Together, they make modern Java **expressive**, **concise**, and **maintainable** — aligning with the reactive, asynchronous, and data-driven world we’re in.


---
