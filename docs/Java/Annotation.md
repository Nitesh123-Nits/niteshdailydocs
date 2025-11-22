Understanding annotations deeply makes your Java knowledge **modern and framework-ready** (Spring, Hibernate, JPA, etc. are annotation-heavy).

Let’s start our **Annotation Series — Part 1: Introduction to Annotations in Java** 🧠

---

## 🧩 Part 1 — What Are Annotations in Java?

Annotations are **metadata** added to Java code — they **don’t affect program logic directly**, but they **give information to the compiler, runtime, or frameworks** about how to treat a class, method, or field.

In short:

> Annotations are like **labels or tags** attached to code elements.

---

### 🧠 Simple Example

```java
@Override
public String toString() {
    return "Hello";
}
```

Here `@Override` tells the **compiler** that this method overrides a method from a superclass.

If it doesn’t, you’ll get a **compile-time error**.  
So, annotations can **enhance safety**, **reduce boilerplate**, and **guide frameworks**.

---

## 🧩 Types of Annotations in Java

### 1️⃣ **Built-in Annotations**

These are already provided by Java.

|Annotation|Used For|Example|
|---|---|---|
|`@Override`|Ensure overriding a parent method|`@Override public void run()`|
|`@Deprecated`|Mark code as deprecated|`@Deprecated void oldMethod()`|
|`@SuppressWarnings`|Tell compiler to ignore warnings|`@SuppressWarnings("unchecked")`|
|`@FunctionalInterface`|Marks interface with a single abstract method|`@FunctionalInterface public interface Runnable { void run(); }`|

---

### 2️⃣ **Meta-Annotations (Annotations on Annotations)**

These define **how your custom annotations behave**.  
They’re used when you create your own annotation.

|Meta-Annotation|Description|
|---|---|
|`@Retention`|Defines **how long** the annotation is retained (source, class, or runtime)|
|`@Target`|Defines **where** the annotation can be applied (class, method, field, etc.)|
|`@Inherited`|Allows annotation to be **inherited by subclasses**|
|`@Documented`|Indicates that annotation should appear in Javadoc|

---

### 🔍 Example of Meta-Annotations

Let’s say we create our own annotation:

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface Auditable {
    String value() default "default-audit";
}
```

Here’s what each part means:

|Meta-Annotation|Meaning|
|---|---|
|`@Retention(RetentionPolicy.RUNTIME)`|Annotation is available at **runtime** (can be read via reflection)|
|`@Target(ElementType.TYPE)`|Can be applied on **class, interface, or enum**|
|`@Documented`|Will be included in Javadoc|
|`String value()`|Defines a **property** (annotation field) that you can set|

---

## 🧩 Applying the Custom Annotation

```java
@Auditable(value = "TransactionModule")
public class Transaction {
    private double amount;
}
```

Or simply (if you use `value()` as default field):

```java
@Auditable("TransactionModule")
public class Transaction { ... }
```

---

## 🧠 Accessing Annotation at Runtime (Reflection)

```java
public class AnnotationReader {
    public static void main(String[] args) {
        Class<Transaction> clazz = Transaction.class;

        if (clazz.isAnnotationPresent(Auditable.class)) {
            Auditable audit = clazz.getAnnotation(Auditable.class);
            System.out.println("Audit value: " + audit.value());
        }
    }
}
```

**Output:**

```
Audit value: TransactionModule
```

✅ You’ve just:

- Created your own annotation
    
- Controlled where it can be applied
    
- Controlled how long it’s retained
    
- Accessed it dynamically at runtime
    

---

## 🧱 Summary

|Concept|Description|
|---|---|
|**Annotation**|A form of metadata used to provide information to compiler/runtime|
|**Meta-Annotations**|Annotations that define how other annotations behave|
|**RetentionPolicy**|`SOURCE`, `CLASS`, or `RUNTIME`|
|**Target**|Restricts where annotation can be used|
|**Custom Annotations**|Define your own rules for frameworks or code validation|

---

### 🚀 Next in the Series (Part 2 Preview)

In the next part, we’ll cover:

- 🧠 All **RetentionPolicy levels** (SOURCE, CLASS, RUNTIME) with code examples
    
- 🧩 All **Target types** (TYPE, FIELD, METHOD, PARAMETER, etc.)
    
- ⚙️ How frameworks like **Spring** internally use annotations
    

---

# Java Annotation Series — Part 2: Retention & Target Deep Dive

In this part, we’ll focus on the **two most important meta-annotations**:  
👉 `@Retention`  
👉 `@Target`

Both are essential for **defining how and where** your custom annotations behave.

---

## 🧩 What is `@Retention`?

`@Retention` tells the compiler **how long** an annotation should be retained —  
that is, whether it should be available in the **source code**, **compiled .class file**, or **runtime** (for reflection).

---

### 🧠 Syntax

```java
@Retention(RetentionPolicy.RUNTIME)
```

---

### 🧩 RetentionPolicy Levels

|RetentionPolicy|Description|Visible To|Use Case|
|---|---|---|---|
|**SOURCE**|Annotation is discarded during compilation.|Compiler only|`@Override`, `@SuppressWarnings`|
|**CLASS**|Annotation is in `.class` file but not available at runtime.|Compiler + JVM|Default if not specified|
|**RUNTIME**|Annotation is available at runtime (can be accessed via reflection).|Compiler + JVM + Reflection|`@Entity`, `@Autowired`, custom runtime logic|

---

### 🔍 Example 1 — SOURCE Level

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
@interface SourceLevelAnnotation { }

@SourceLevelAnnotation
public class Demo1 { }
```

✅ The compiler **uses** this annotation but removes it after compilation.  
You **can’t read it via reflection**.

---

### 🔍 Example 2 — CLASS Level

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.CLASS)
@Target(ElementType.TYPE)
@interface ClassLevelAnnotation { }

@ClassLevelAnnotation
public class Demo2 { }
```

✅ It is stored in `.class` file but **not loaded into JVM at runtime** — reflection can’t find it.  
Common for **bytecode-processing tools or frameworks** that read `.class` files.

---

### 🔍 Example 3 — RUNTIME Level

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface RuntimeLevelAnnotation {
    String value() default "Active at runtime";
}

@RuntimeLevelAnnotation("DemoClass")
public class Demo3 { }

public class TestRuntime {
    public static void main(String[] args) {
        Class<Demo3> clazz = Demo3.class;

        if (clazz.isAnnotationPresent(RuntimeLevelAnnotation.class)) {
            RuntimeLevelAnnotation ann = clazz.getAnnotation(RuntimeLevelAnnotation.class);
            System.out.println("Annotation value: " + ann.value());
        }
    }
}
```

🟢 **Output:**

```
Annotation value: DemoClass
```

✅ You can access it at runtime through reflection — this is how frameworks like **Spring** and **Hibernate** detect annotations dynamically.

---

## 🧩 What is `@Target`?

`@Target` defines **where** an annotation can be applied.

---

### 🧠 Syntax

```java
@Target(ElementType.METHOD)
```

---

### 🎯 ElementType Options

|ElementType|Used On|Example|
|---|---|---|
|`TYPE`|Class, interface, enum|`@Entity`, `@Configuration`|
|`FIELD`|Field (member variable)|`@Autowired`, `@Id`|
|`METHOD`|Method|`@GetMapping`, `@Bean`|
|`PARAMETER`|Method parameters|`@RequestParam`, `@PathVariable`|
|`CONSTRUCTOR`|Constructors|`@Inject`|
|`LOCAL_VARIABLE`|Local variables|Rarely used|
|`ANNOTATION_TYPE`|Another annotation|Used to create meta-annotations|
|`PACKAGE`|Package declarations|`@PackageAnnotation`|
|`RECORD_COMPONENT`|Java 16+ record components|For record data classes|
|`TYPE_PARAMETER`|Generic type parameters|`class Box<@NotNull T>`|
|`TYPE_USE`|Any use of a type (very general)|Useful in static analysis tools|

---

### 🔍 Example — `@Target` on Fields and Methods

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.METHOD})
@interface Important {
    String value() default "Critical";
}

public class Demo {
    @Important("Primary ID field")
    private int id;

    @Important("Main operation")
    public void process() { }
}
```

You can limit the annotation’s scope so it can’t be misused (e.g., can’t annotate classes or parameters).

---

## 🧱 Combining `@Retention` and `@Target`

Most custom annotations combine both:

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Auditable {
    String module() default "default";
}
```

This means:

- The annotation can only be applied to **classes/interfaces**.
    
- It’s available **at runtime** for reflection or frameworks to use.
    

---

## ⚙️ How Frameworks Use This Internally

**Spring**, **JPA**, and **Jakarta EE** rely heavily on `@Retention(RUNTIME)` annotations.

Examples:

- `@Entity` → tells Hibernate this class maps to a database table.
    
- `@RestController` → Spring scans it at runtime to register HTTP endpoints.
    
- `@Autowired` → Spring finds and injects beans dynamically at runtime.
    

All of these exist because annotations with **RUNTIME retention** can be **read dynamically**.

---

## 🧭 Summary

|Concept|Description|Example|
|---|---|---|
|**RetentionPolicy.SOURCE**|Annotation discarded after compilation|`@Override`|
|**RetentionPolicy.CLASS**|Annotation in `.class`, not at runtime|Bytecode tools|
|**RetentionPolicy.RUNTIME**|Annotation available for reflection|`@Entity`, `@Autowired`|
|**@Target**|Restricts usage locations|`ElementType.TYPE`, `FIELD`, `METHOD`, etc.|

---

### 🚀 Next in the Series (Part 3 Preview)

In the next part, we’ll explore:

- ✳️ Creating **annotations with multiple elements**
    
- 🎛️ Using **default values**
    
- 💥 Handling **arrays and enums** in annotation fields
    
- ⚙️ Real-world examples used in frameworks (Spring-like logic)
    

---


 **Annotation Series for Java and Spring Boot**, designed to cover:  
✅ Core annotations every **Java and Spring Boot developer** must know.  
✅ Real-world examples (how they’re used in real apps).  
✅ Interview-level explanations (what’s happening internally).  
✅ Best practices and gotchas (for production-grade code).

---

## 🧩 **Part 1: Core Java Annotations**

Before diving into Spring Boot, let’s build a strong Java foundation.

### 🔹 1. `@Override`

**Purpose:** Ensures that a method overrides a method from a superclass or implements an interface method.

```java
class Vehicle {
    void start() {
        System.out.println("Vehicle starting...");
    }
}

class Car extends Vehicle {
    @Override
    void start() { // Ensures we're overriding correctly
        System.out.println("Car starting...");
    }
}
```

✅ **Why Important:**

- Prevents accidental overloading instead of overriding.
    
- Compilation error if method signature doesn’t match parent.
    

---

### 🔹 2. `@Deprecated`

**Purpose:** Marks a method/class as obsolete — warns developers not to use it.

```java
@Deprecated
class OldPaymentService {
    void processPayment() { }
}

class NewPaymentService {
    void processPayment() { }
}
```

✅ **Best Practice:**  
Always provide an alternative using `@deprecated` Javadoc tag:

```java
/**
 * @deprecated use {@link NewPaymentService} instead.
 */
```

---

### 🔹 3. `@SuppressWarnings`

**Purpose:** Suppresses compiler warnings (like unchecked, deprecation).

```java
@SuppressWarnings("unchecked")
void loadList() {
    List rawList = new ArrayList();  // Raw types warning suppressed
}
```

✅ **Use Carefully:**  
Never overuse — it can hide genuine problems.

---

### 🔹 4. `@FunctionalInterface` (Java 8+)

**Purpose:** Marks an interface as functional (exactly one abstract method).

```java
@FunctionalInterface
interface Greeting {
    void sayHello(String name);
}
```

✅ **Real-world use:**  
Required for **lambda expressions** and **streams**.

---

### 🔹 5. `@SafeVarargs` (Java 7+)

**Purpose:** Prevents warnings for varargs methods using generics.

```java
@SafeVarargs
static <T> void printAll(T... args) {
    for (T t : args) System.out.println(t);
}
```

---

### 🔹 6. `@Record` (Java 16+ feature)

Technically not an annotation — but **record** uses annotation-like semantics:

```java
public record Employee(String name, int age) {}
```

✅ Auto-generates:

- Constructor
    
- Getters
    
- `equals`, `hashCode`, `toString`
    

---

Now, let’s jump into **Spring Boot annotations**, where real-world development and interview questions come in.

---

## ⚙️ **Part 2: Core Spring Boot Annotations**

### 🔹 1. `@SpringBootApplication`

**Purpose:** Entry point annotation combining three:

- `@Configuration`
    
- `@EnableAutoConfiguration`
    
- `@ComponentScan`
    

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

✅ **Internal Magic:**  
Auto-configures your app based on dependencies (e.g., Tomcat, DataSource, etc.)

---

### 🔹 2. `@Component`, `@Service`, `@Repository`, `@Controller`

**Purpose:** Marks a class as a **Spring Bean** (managed by Spring’s IoC container).

```java
@Component
class EmailValidator { ... }

@Service
class UserService { ... }

@Repository
class UserRepository { ... }

@Controller
class UserController { ... }
```

✅ **Interview Tip:**  
All are **specializations of `@Component`**, but with semantic intent.

---

### 🔹 3. `@Autowired` / `@Inject` / `@Qualifier`

**Purpose:** Dependency Injection (DI).

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    @Autowired
    public OrderService(@Qualifier("stripePayment") PaymentService ps) {
        this.paymentService = ps;
    }
}
```

✅ **Best Practice:**  
Prefer **constructor injection** for immutability and testing.

---

### 🔹 4. `@Configuration` and `@Bean`

**Purpose:** Define Spring beans manually (explicit bean creation).

```java
@Configuration
public class AppConfig {

    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

✅ **Internally:**  
`@Configuration` classes are **proxy-enhanced** to ensure singleton beans.

---

### 🔹 5. `@RestController` + `@RequestMapping`

**Purpose:** Exposes REST APIs.

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(new User(id, "Nitesh"));
    }
}
```

✅ **Combines:**  
`@Controller` + `@ResponseBody`

---

### 🔹 6. `@RequestParam`, `@PathVariable`, `@RequestBody`

**Purpose:** Handle different types of HTTP inputs.

```java
@GetMapping("/greet")
public String greet(@RequestParam String name) { return "Hello " + name; }

@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) { ... }

@PostMapping("/users")
public User addUser(@RequestBody User user) { ... }
```

---

### 🔹 7. `@Transactional`

**Purpose:** Enables declarative transaction management.

```java
@Transactional
public void placeOrder(Order order) {
    orderRepo.save(order);
    paymentService.debit(order);
}
```

✅ **Internally:**  
Uses **AOP proxies** to manage transaction lifecycle (commit/rollback).

---

### 🔹 8. `@Value` and `@ConfigurationProperties`

**Purpose:** Inject values from `application.yml` or environment.

```java
@Value("${app.name}")
private String appName;

@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
}
```

✅ **Best Practice:**  
Prefer `@ConfigurationProperties` for **type safety**.

---

### 🔹 9. `@EnableScheduling`, `@Scheduled`

**Purpose:** Schedule background tasks.

```java
@EnableScheduling
@SpringBootApplication
public class SchedulerApp { }

@Component
class TaskScheduler {
    @Scheduled(fixedRate = 5000)
    void run() { System.out.println("Running every 5 sec"); }
}
```

---

### 🔹 10. `@Conditional`, `@Profile`

**Purpose:** Conditional bean creation.

```java
@Profile("dev")
@Bean
public DataSource devDataSource() { ... }

@ConditionalOnMissingBean
@Bean
public DefaultEmailService emailService() { ... }
```

✅ **Used internally in Spring Boot’s auto-configuration.**

---

## 🚀 Next in the Series (Part 2 Preview)

We’ll go deeper into:

- `@ControllerAdvice`, `@ExceptionHandler`
    
- `@Aspect`, `@Around`, `@Before`
    
- `@PostConstruct`, `@PreDestroy`
    
- Custom Annotations (`@LogExecutionTime`, etc.)
    
- Meta-annotations and annotation processing
    


---

# 🚀 **All Important Caching Annotations in Spring Boot**

Spring Boot’s caching abstraction (Spring Cache) makes caching extremely simple using annotations. These annotations work with:

- **Caffeine**
    
- **Redis**
    
- **Ehcache**
    
- **Hazelcast**
    
- **Simple in-memory concurrent map (default)**
    

Once you add:

```java
@EnableCaching
```

…you instantly get caching capabilities across your services.

Let’s break down all caching annotations you'll encounter in real projects and interviews.

---

# 1️⃣ **@EnableCaching**

### **Purpose:**

Turns on Spring’s caching mechanism.

### **Where it goes:**

On the main application class.

### **Example:**

```java
@SpringBootApplication
@EnableCaching
public class ProductServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}
```

### **What it does:**

- Activates cache interceptors
    
- Allows Spring to wrap methods annotated with caching annotations
    
- Enables cache providers (Redis, Caffeine, etc.)
    

---

# 2️⃣ **@Cacheable**

### **Purpose:**

Caches the **method return value**.  
Next call with the same key returns cached data _without executing the method_.

### **Most used caching annotation.**

### **Example:**

```java
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(String id) {
        simulateSlowDB();
        return new Product(id, "Macbook Pro");
    }
}
```

### **What happens:**

- First call → method executes
    
- Next calls → response comes from cache (super fast!)
    

### **Options:**

- `condition = "#id > 10"`
    
- `unless = "#result == null"`
    
- `sync = true` (avoid race condition)
    

---

# 3️⃣ **@CachePut**

### **Purpose:**

- Executes the method **always**
    
- Updates the cache **with the return value**
    
- Used for **create/update operations**
    

### **Example:**

```java
@CachePut(value = "products", key = "#product.id")
public Product update(Product product) {
    return repository.save(product);
}
```

### Use Case:

- After updating product in DB, we want the latest product in cache.
    

---

# 4️⃣ **@CacheEvict**

### **Purpose:**

Evicts (deletes) items from cache.

### **Example:**

```java
@CacheEvict(value = "products", key = "#id")
public void deleteProduct(String id) {
    repository.deleteById(id);
}
```

### **Useful Options:**

📌 **1. Remove entire cache**

```java
@CacheEvict(value = "products", allEntries = true)
public void deleteAllProducts() {}
```

📌 **2. Remove cache after method success only**

```java
@CacheEvict(value = "products", key = "#id", beforeInvocation = false)
```

📌 **3. Remove cache even if method fails**

```java
@CacheEvict(value = "products", key = "#id", beforeInvocation = true)
```

---

# 5️⃣ **@Caching**

### **Purpose:**

Allows multiple caching rules on the same method.

### **Example:**

```java
@Caching(
    put = {
        @CachePut(value = "products", key = "#product.id")
    },
    evict = {
        @CacheEvict(value = "inventory", key = "#product.id")
    }
)
public Product save(Product product) {
    return repository.save(product);
}
```

### **Use Case:**

- Update product cache
    
- Remove stale inventory cache  
    …in **one single method**
    

---

# 6️⃣ **@CacheConfig**

### **Purpose:**

Used at class level to define **shared cache settings**.

### **Example:**

```java
@Service
@CacheConfig(cacheNames = "products")
public class ProductService {

    @Cacheable(key = "#id")
    public Product getProduct(String id) { ... }

    @CacheEvict(key = "#id")
    public void deleteProduct(String id) { ... }
}
```

### **What it does:**

Removes the need to repeatedly specify:

```java
value = "products"
```

---

# 7️⃣ **@EnableSpringConfigured**

(rare, but shows up in interviews)

Used with **AspectJ** when caching needs to apply to non-Spring-managed beans.  
Not common for microservices, but good to know.

---

# 8️⃣ **@CacheResult, @CacheRemove, @CacheRemoveAll** (JSR-107 / JCache)

If using **JCache API (JSR-107)** instead of Spring Cache:

- `@CacheResult` → equivalent to `@Cacheable`
    
- `@CacheRemove` → equivalent to `@CacheEvict`
    
- `@CacheRemoveAll` → clears entire cache
    

### Example (JSR-107):

```java
@CacheResult(cacheName = "products")
public Product find(String id) { ... }
```

Spring automatically maps these JSR annotations.

---

# 🔥 Real-World Caching Flow Example

Let’s combine them like in a real microservice:

```java
@Service
@CacheConfig(cacheNames = "products")
public class ProductService {

    @Cacheable(key = "#id")
    public Product findById(String id) {
        return repo.findById(id).orElseThrow();
    }

    @CachePut(key = "#product.id")
    public Product update(Product product) {
        return repo.save(product);
    }

    @CacheEvict(key = "#id")
    public void delete(String id) {
        repo.deleteById(id);
    }
}
```

Perfect pattern:

- `GET` → `@Cacheable`
    
- `PUT/POST` → `@CachePut`
    
- `DELETE` → `@CacheEvict`
    

---

# 🧠 Interview-Level Summary

|Annotation|Purpose|
|---|---|
|`@EnableCaching`|Enable caching system|
|`@Cacheable`|Cache method result (most used)|
|`@CachePut`|Update cache after modifying data|
|`@CacheEvict`|Remove cache entries|
|`@Caching`|Combine multiple cache rules|
|`@CacheConfig`|Share cache config at class level|
|`@CacheResult` (JCache)|Cache result (JSR-107)|
|`@CacheRemove` (JCache)|Remove entry|
|`@CacheRemoveAll` (JCache)|Clear full cache|

---

# Want to go deeper?

I can give you next:

### ✔ **Caching in Microservices with Redis**

### ✔ **Cache + Resilience4j integration (Cache First, Fallback Later)**

### ✔ **@Cacheable with Feign Client**

### ✔ **Caffeine Cache + Expire policy + Size limits**


---

# ✅ **Spring AOP — Core Annotations (With Use Cases & Snippets)**

Spring AOP is built around **cross-cutting concerns** like logging, metrics, transactions, auditing, security, caching, retry, tracing, etc.

Below are the core annotations you must know.

---

# **1️⃣ `@Aspect`**

Marks a class as an Aspect.

### ✔ What it does

Defines a component containing cross-cutting logic (advice + pointcuts).

### ✔ Example

```java
@Aspect
@Component
public class LoggingAspect {
}
```

### ✔ Real-world

Commonly used to implement:

- Request logging
    
- Performance monitoring
    
- Method validation
    
- Security checks
    
- Multi-service tracing
    

---

# **2️⃣ `@Pointcut`**

Defines the _where_ — the join points.

### ✔ Example

```java
@Pointcut("execution(* com.nitesh.service.*.*(..))")
public void serviceLayer() {}
```

### ✔ Use Cases

- Target package/class/method patterns
    
- Annotated methods (`@Transactional`, `@Cacheable`, custom annotations)
    
- Exclude noisy methods
    

---

# **3️⃣ `@Before`**

Runs logic **before** the method executes.

### ✔ Example

```java
@Before("serviceLayer()")
public void beforeServiceCall(JoinPoint jp) {
    System.out.println("Before: " + jp.getSignature());
}
```

### ✔ Use Cases

- Logging request params
    
- Authentication/authorization pre-check
    
- API input validation
    
- Tracing correlation ID creation
    

---

# **4️⃣ `@After`**

Runs **always after** method execution (success or failure).

### ✔ Example

```java
@After("serviceLayer()")
public void cleanup() {
    System.out.println("Cleanup logic executed");
}
```

### ✔ Use Cases

- Audit trails
    
- MDC cleanup
    
- ThreadLocal cleanup in multithreaded apps
    

---

# **5️⃣ `@AfterReturning`**

Runs **after method returns successfully**.

### ✔ Example

```java
@AfterReturning(
        pointcut = "serviceLayer()",
        returning = "result")
public void afterReturn(Object result) {
    System.out.println("Method Result → " + result);
}
```

### ✔ Use Cases

- Mask PII in logs
    
- Response enrichment
    
- Post-processing response
    
- Cache warm-up
    

---

# **6️⃣ `@AfterThrowing`**

Runs **when method throws an exception**.

### ✔ Example

```java
@AfterThrowing(
        pointcut = "serviceLayer()",
        throwing = "ex")
public void logError(Exception ex) {
    ex.printStackTrace();
}
```

### ✔ Use Cases

- Centralized error logging
    
- Alerting & monitoring
    
- Exception transformation
    
- Security anomaly detection
    

---

# **7️⃣ `@Around`** (MOST IMPORTANT)

Allows full control — runs **before & after** method execution.

### ✔ Example

```java
@Around("serviceLayer()")
public Object measureExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = pjp.proceed();
    long end = System.currentTimeMillis();

    System.out.println("Exec Time: " + (end - start) + "ms");

    return result;
}
```

### ✔ Use Cases

- Performance monitoring
    
- Distributed tracing
    
- Retry + fallback logic
    
- Rate limiting
    
- Method-level transaction wrapping
    
- Correlation ID propagation
    

This is the backbone of major frameworks like **Resilience4j**, **Spring Transactions**, **Caching**, etc.

---

# **8️⃣ `@DeclareParents`** (Advanced, rarely asked but high-value)

Introduces new interfaces to existing beans — for mixin support.

### ✔ Example

```java
@Aspect
public class MixinAspect {
    @DeclareParents(
        value = "com.nitesh.service.*+",
        defaultImpl = DefaultAuditable.class)
    public static Auditable auditable;
}
```

### ✔ Use Case

Add functionality **without changing code**, e.g. soft-delete capability.

---

# **9️⃣ Custom Annotation for AOP-driven Aspects**

This is how real-world AOP is used.

### ✔ Example Custom Annotation

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TrackExecution {
}
```

### ✔ Aspect

```java
@Around("@annotation(com.nitesh.TrackExecution)")
public Object track(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.nanoTime();
    Object result = pjp.proceed();
    long end = System.nanoTime();

    System.out.println("Time : " + (end - start));
    return result;
}
```

### ✔ Use Cases

- @Audit
    
- @Trace
    
- @Secure
    
- @Retry
    
- @RateLimit
    
- @MaskSensitiveData
    
- @DecryptData / @EncryptData
    

---

# 🎯 **Typical Interview Questions You’ll Face**

1. Difference between `@Before` and `@Around`?
    
2. Why is `@Around` most powerful?
    
3. How Spring AOP works under the hood (Proxies, JDK/CGLIB)?
    
4. How to apply AOP to only annotated methods?
    
5. How to use custom annotations to implement cross-cutting behavior?
    
6. What is the difference between AspectJ and Spring AOP?
    
7. Limitations of Spring AOP (method-level, proxy-based).
    

If you want, I can explain **all interview questions with full answers** too.

---

# Want to go to the next set?

Options:

✔ Spring Boot Core Annotations  
✔ Spring Security Annotations  
✔ Spring Cloud / Resilience4j Annotations  
✔ Spring WebFlux-specific Annotations  
✔ Jackson / JSON annotations  
✔ Lombok annotations  
✔ Testing annotations (JUnit + Mockito)


---

# ✅ **Lombok Annotations — The Only Ones That Matter (With Examples & Use Cases)**

Lombok reduces boilerplate — getters, setters, constructors, builders, logging, equals/hashCode, and more.  
But in interviews, you must show you understand both **functionality and hidden behaviors** (equals, immutability, builder cost, unsafe usages).

---

# **1️⃣ @Getter / @Setter**

### ✔ Use

Generates getter/setter methods automatically.

```java
@Getter
@Setter
public class User {
    private String name;
    private int age;
}
```

### ✔ Real-World Use

POJOs, request/response DTOs, config classes, entity models.

### ⚠ Avoid

Using `@Setter` on `@Entity` unless intentional — it breaks immutability and domain integrity.

---

# **2️⃣ @ToString**

### ✔ Use

Generates a `toString()` implementation.

```java
@ToString
public class Order {
    private String id;
    private double amount;
}
```

### ✔ Use Case

Logging, debugging.

### ⚠ Danger

Never log sensitive fields (password, tokens, PII).  
Use:

```java
@ToString.Exclude
private String password;
```

---

# **3️⃣ @EqualsAndHashCode**

### ✔ Use

Generates `equals()` and `hashCode()`.

```java
@EqualsAndHashCode
public class Employee {
    private Long id;
    private String name;
}
```

### ⚠ Danger

Avoid using on entities with mutable fields → can break Sets/Maps & Hibernate identity.

Use:

```java
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
```

---

# **4️⃣ @NoArgsConstructor / @AllArgsConstructor / @RequiredArgsConstructor**

Used for autowiring, JPA entities, test data creation.

```java
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
public class User {
    @NonNull private String name;
}
```

### ✔ Use Case

- JPA requires a `@NoArgsConstructor`
    
- Spring requires a constructor for DI (`@RequiredArgsConstructor` is perfect)
    

### ⭐ Best Practice

In Spring Boot, always prefer:

```java
@RequiredArgsConstructor
class Service {
    private final UserRepository repo;
}
```

---

# **5️⃣ @Builder** (One of the most used)

### ✔ Use

Creates a builder pattern for complex object construction.

```java
@Builder
public class Book {
    private String title;
    private double price;
}
```

Usage:

```java
Book b = Book.builder()
             .title("Java")
             .price(500)
             .build();
```

### ✔ Real-World Use

- Config classes
    
- Immutable DTO creation
    
- Large request bodies
    
- Kafka message building
    
- Domain model creation in clean architecture
    

### ⚠ Beware

- Builder is **slower** than constructors for high-performance or small objects.
    
- Never use it for **hot loops** or **millions of object creations**.
    

---

# **6️⃣ @Data** (VERY powerful but dangerous)

Equivalent to:

```
@Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor
```

```java
@Data
public class User {
    private String name;
    private String email;
}
```

### ✔ Use Case

DTOs, value objects.

### ⚠ Avoid `@Data` on:

- JPA entities → equals/hashCode → infinite loops, identity issues
    
- Aggregates containing sensitive fields
    
- High-security models
    

Use `@Data` only when safe.

---

# **7️⃣ @Value** (Immutable DTO)

Immutable version of `@Data` (all fields `private final`).

```java
@Value
public class ImmutableConfig {
    String url;
    int timeout;
}
```

### ✔ Use Case

- Thread-safe configuration
    
- Messages
    
- Kafka events
    
- Cache keys
    

---

# **8️⃣ @Slf4j / @Log4j2 / @CommonsLog**

```java
@Slf4j
@Service
public class PaymentService {
    
    public void pay() {
        log.info("Payment started...");
    }
}
```

### ✔ Use Case

Logging without manual logger creation.

---

# **9️⃣ @SneakyThrows**

Allows throwing checked exceptions without declaring them.

```java
@SneakyThrows
public void risky() {
    throw new Exception("Boom");
}
```

### ⚠ Not recommended in production

Hard to read  
Breaks contract  
Confuses teammates

Use only for tests or temporary code.

---

# **🔟 @Cleanup**

Auto-closes resources (like try-with-resources).

```java
public void read() {
    @Cleanup FileInputStream fis = new FileInputStream("data.txt");
}
```

Not commonly used now due to `try-with-resources`.

---

# **1️⃣1️⃣ @NonNull**

Adds null-check to constructor/setter parameters.

```java
public class User {
    @NonNull private String name;
}
```

### ✔ Use Case

Fail fast on illegal input.

---

# **1️⃣2️⃣ @SuperBuilder**

Builder support for inheritance.

```java
@SuperBuilder
public class User { ... }

@SuperBuilder
public class Admin extends User { ... }
```

### ✔ Use Case

Hierarchical DTOs, inheritance-heavy models.

---

# **1️⃣3️⃣ @Accessors(chain = true)**

Fluent API style setters.

```java
@Accessors(chain = true)
@Getter @Setter
public class User {
    private String name;
    private int age;
}
```

Usage:

```java
user.setName("Ram").setAge(28);
```

---

# 🚀 Interview-Level Best Practices

### ✔ When to avoid Lombok?

- For shared public APIs
    
- For deeply nested inheritance
    
- When reflection/bytecode tools dislike Lombok
    

### ✔ Why some companies ban `@Data`?

- Equals/hashCode break entity identity
    
- Setter generation breaks immutability
    
- Harder debugging
    
- Security risk (toString leaks fields)
    

### ✔ Most common Lombok annotations in Spring Boot microservices

- `@Slf4j`
    
- `@RequiredArgsConstructor`
    
- `@Getter`
    
- `@Builder`
    
- `@Value`
    
- `@Data` (DTO only)
    

---

# Want next in the series?

Choose your next category:

✔ **Jackson JSON annotations**  
✔ **Spring Security annotations**  
✔ **Spring Cloud + Resilience4j annotations**  
✔ **Testing annotations (JUnit, Mockito)**  
✔ **Controller & REST annotations**



Here is a **clean, production-grade, interview-focused guide** on the most important **JUnit 5 + Mockito annotations** that every Spring Boot / microservices engineer must know.

This is the set of annotations you actually use in enterprise codebases, along with pitfalls, best practices, and real project examples.

---

# 🚀 **JUnit 5 Annotations (Core + Spring Boot Testing)**

---

# **1️⃣ @Test**

Marks a method as a test case.

```java
@Test
void shouldCalculateTotal() {
    assertEquals(10, service.calculate());
}
```

### ✔ Use Case

Basic unit tests.

---

# **2️⃣ @BeforeEach / @AfterEach**

Runs before/after **each test method**.

```java
@BeforeEach
void init() {
    service = new OrderService();
}

@AfterEach
void cleanup() {
    System.out.println("Test completed");
}
```

### ✔ Use Case

- Creating fresh test objects
    
- Resetting mocks
    
- Cleanup of temp data
    

---

# **3️⃣ @BeforeAll / @AfterAll**

Runs once per class (static unless using @TestInstance).

```java
@BeforeAll
static void setupAll() { ... }

@AfterAll
static void tearDownAll() { ... }
```

### ✔ Use

- Connecting to test DB
    
- Loading configuration
    
- Expensive setup shared across tests
    

---

# **4️⃣ @DisplayName**

Human-friendly test names.

```java
@DisplayName("Should apply discount when user is premium")
@Test
void discountTest() { ... }
```

---

# **5️⃣ @Nested**

For grouping test cases logically.

```java
@Nested
class WhenUserIsPremium {
    @Test void shouldApplyDiscount() {}
}
```

---

# **6️⃣ @RepeatedTest**

Runs a test multiple times.

```java
@RepeatedTest(5)
void stressTest() {}
```

---

# **7️⃣ @ParameterizedTest** (Advanced + Interview Favorite)

Allows testing with multiple inputs.

```java
@ParameterizedTest
@ValueSource(strings = {"abc", "xyz"})
void testValues(String input) {
    assertTrue(input.length() == 3);
}
```

Also supports:

- @CsvSource
    
- @CsvFileSource
    
- @MethodSource
    

---

# **8️⃣ @TestInstance**

Defines lifecycle of test instance.

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class ConfigTest { ... }
```

---

# **Spring Boot Testing Annotations**

---

# **9️⃣ @SpringBootTest**

Loads the full Spring context.

```java
@SpringBootTest
class OrderServiceTest {}
```

### ✔ Use Case

- Integration tests
    
- Testing full wiring
    
- Hitting repositories + services
    

### ⚠ Avoid

Using it for simple unit tests → slow.

---

# **🔟 @WebMvcTest**

Loads only controller layer + MVC beans.

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {}
```

### ✔ Use Case

- Testing Controllers in isolation
    
- Mocking service layer
    
- Fast HTTP layer tests
    

---

# **1️⃣1️⃣ @DataJpaTest**

Loads only JPA + Hibernate + DB-related beans.

```java
@DataJpaTest
class UserRepositoryTest { ... }
```

### ✔ Use Case

- Testing repositories
    
- Verifying DB queries
    
- Using in-memory DB (H2)
    

---

# **1️⃣2️⃣ @MockBean**

Creates a mock in Spring context.

```java
@MockBean
private PaymentService paymentService;
```

### ✔ Use Case

- Replace real beans with mocks in @SpringBootTest / @WebMvcTest
    
- Control behavior of dependencies
    

---

# **1️⃣3️⃣ @Autowired (in tests)**

Inject test subject or mocked dependencies.

```java
@Autowired
private OrderController controller;
```

---

# ⚡ Mockito Annotations (Core Unit Testing)

---

# **1️⃣4️⃣ @Mock**

Creates a pure Mockito mock.

```java
@Mock
private PaymentGateway gateway;
```

### ✔ Use inside

- Unit tests
    
- Without Spring context
    
- Fastest possible tests
    

---

# **1️⃣5️⃣ @InjectMocks**

Injects mocks into the class-under-test.

```java
@InjectMocks
private PaymentService paymentService;
```

### ✔ Use Case

Classic unit testing:

- No Spring
    
- No context loading
    
- Pure business logic
    

---

# **1️⃣6️⃣ @Captor**

Captures arguments passed to mocked methods.

```java
@Captor
ArgumentCaptor<Order> orderCaptor;
```

Usage:

```java
verify(repo).save(orderCaptor.capture());
assertEquals("abc", orderCaptor.getValue().getId());
```

### ✔ Use Case

- Verify arguments
    
- Deep test validation
    
- Verify transformations
    

---

# **1️⃣7️⃣ @Spy**

Partial mocks (real methods + stubbing).

```java
@Spy
private InvoiceCalculator calculator;
```

### ✔ Use

- When you want real behavior but override some specific parts
    
- Decorator pattern testing
    

### ⚠ Avoid

Overusing @Spy — it often signals design issues.

---

# **1️⃣8️⃣ @ExtendWith(MockitoExtension.class)**

Enables Mockito for JUnit 5.

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest { ... }
```

---

# **1️⃣9️⃣ @MockStatic**

For mocking static methods (Mockito 3.4+).

```java
try (MockedStatic<UUID> mocked = mockStatic(UUID.class)) {
    mocked.when(UUID::randomUUID).thenReturn(fixedUuid);
}
```

### ✔ Use Case

- Static factory utilities
    
- UUID generation
    
- Time/date static calls
    

### ⚠ Use carefully

Static mocking is powerful but costs isolation.

---

# 🎯 Real-World Spring Boot Testing Examples

---

## **Unit Test (Fastest) — Mockito + JUnit**

```java
@ExtendWith(MockitoExtension.class)
class PaymentServiceTest {

    @Mock PaymentGateway gateway;

    @InjectMocks PaymentService service;

    @Test
    void shouldPaySuccessfully() {
        when(gateway.charge(anyDouble())).thenReturn(true);

        boolean result = service.pay(100);

        assertTrue(result);
        verify(gateway).charge(100);
    }
}
```

---

## **Controller Test — WebMvcTest**

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired MockMvc mvc;

    @MockBean OrderService service;

    @Test
    void shouldPlaceOrder() throws Exception {
        when(service.placeOrder()).thenReturn("OK");

        mvc.perform(get("/orders"))
           .andExpect(status().isOk())
           .andExpect(content().string("OK"));
    }
}
```

---

## **Repository Test — DataJpaTest**

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired UserRepository repo;

    @Test
    void shouldSaveUser() {
        User user = repo.save(new User(null, "Ram"));
        assertNotNull(user.getId());
    }
}
```

---

# 🎤 Top Interview Questions on JUnit + Mockito

You will 100% face these:

### ✔ Difference between @Mock, @MockBean, and @Spy?

### ✔ Why use @ExtendWith(MockitoExtension.class)?

### ✔ What is the difference between @SpringBootTest and @WebMvcTest?

### ✔ Why use @DataJpaTest for repository testing?

### ✔ Why avoid @SpringBootTest for unit tests?

### ✔ How does @InjectMocks work internally?

### ✔ When to use @Captor?

### ✔ How to mock static functions?

### ✔ How to test exceptions?

If you want, I can give **complete answers for each question** like interview responses.

---

# Next Topics Available

Choose your next topic:

✔ Spring Security annotations  
✔ Jackson JSON annotations  
✔ Spring Cloud + Resilience4j annotations  
✔ REST Controller annotations  
✔ Microservices-specific annotations  
✔ Testcontainers annotations

Here is the **ultimate, production-grade, interview-oriented guide** to the **most important Spring Security annotations** used in real microservices and enterprise Spring Boot applications.

This covers:  
✔ Core annotations  
✔ Method-level security  
✔ Web security  
✔ OAuth2/JWT-related annotations  
✔ Real-world usage patterns  
✔ Common interview questions  
✔ Best practices and pitfalls

---

# 🚀 **SPRING SECURITY ANNOTATIONS — CORE + ADVANCED**

---

# **1️⃣ @EnableWebSecurity**

Enables Spring Security’s web security support.

```java
@EnableWebSecurity
public class SecurityConfig { }
```

### ✔ Use Case

- Base annotation for enabling Spring Security
    
- Activates filter chain (SecurityFilterChain)
    

### ⚠ Note

Since Spring Boot 3, you configure security using a `SecurityFilterChain` bean, not WebSecurityConfigurerAdapter.

---

# **2️⃣ @Configuration (Security Configuration)**

You always combine:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig { ... }
```

---

# **3️⃣ @EnableMethodSecurity** (Spring Boot 3+ Recommended)

Used for **@PreAuthorize**, **@PostAuthorize**, **@Secured**, etc.

```java
@EnableMethodSecurity
public class SecurityConfig { }
```

### ✔ Use Case

- Method-level authorization
    
- Fine-grained access control
    

---

# **4️⃣ @PreAuthorize**

Runs _before_ method execution → Expression-based.

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { }
```

### ✔ Supported Expressions

- `hasRole('ADMIN')`
    
- `hasAuthority('READ_PRIVILEGE')`
    
- `authentication.principal.username == #username`
    
- `#id == authentication.principal.id`
    

### ✔ Real Uses

- Service-layer restrictions
    
- Controller endpoint permissions
    
- Multi-tenant access checks
    

### ⭐ Powerful example:

```java
@PreAuthorize("#userId == authentication.principal.userId")
public UserProfile getProfile(Long userId) { ... }
```

---

# **5️⃣ @PostAuthorize**

Runs _after_ method execution → Can validate returned object.

```java
@PostAuthorize("returnObject.owner == authentication.name")
public Document getDocument(Long id) { ... }
```

### ✔ Use Case

Object-level access control.

---

# **6️⃣ @Secured**

Older annotation, role-based only.

```java
@Secured("ROLE_ADMIN")
public void manage() { }
```

### ✔ Use Case

Legacy compatibility.

---

# **7️⃣ @RolesAllowed** (JSR-250)

Alternate role-based access.

```java
@RolesAllowed({"ADMIN", "MANAGER"})
public void approveOrder() { }
```

---

# **8️⃣ @WithMockUser (Testing)**

Used in **Spring Security test** framework.

```java
@Test
@WithMockUser(username = "john", roles = "ADMIN")
void testAdminAccess() throws Exception { }
```

---

# **9️⃣ @AuthenticationPrincipal**

Injects the currently authenticated user.

```java
@GetMapping("/me")
public UserResponse me(@AuthenticationPrincipal CustomUser user) {
    return new UserResponse(user.getId(), user.getEmail());
}
```

### ✔ Use Case

- Get logged-in user
    
- Resolve claims from JWT token
    
- Custom UserDetails
    

---

# **🔟 @EnableGlobalAuthentication**

Enables global authentication providers (rarely needed explicitly).

---

# ⭐ JWT / OAuth2 Specific Annotations

---

# **1️⃣1️⃣ @EnableOAuth2Client** (Legacy)

Used for OAuth2 Client applications.

---

# **1️⃣2️⃣ @EnableResourceServer** (Legacy OAuth2)

Used in OAuth2 resource servers (pre Spring Security 5.5).

---

# **1️⃣3️⃣ @RegisteredOAuth2AuthorizedClient**

Injected OAuth2 Authorized Client.

```java
@GetMapping("/profile")
public String profile(@RegisteredOAuth2AuthorizedClient OAuth2AuthorizedClient client) { }
```

---

# **1️⃣4️⃣ @CurrentSecurityContext**

Access security context directly.

```java
@GetMapping("/info")
public String info(@CurrentSecurityContext SecurityContext context) {
    return context.getAuthentication().getName();
}
```

---

# ⭐ Spring Security + JWT (Modern Stack)

In JWT-based microservices, you often use:

### **@AuthenticationPrincipal Jwt jwt**

```java
@GetMapping("/user")
public Map<String, Object> user(@AuthenticationPrincipal Jwt jwt) {
    return jwt.getClaims();
}
```

### ✔ Use Case

Injecting claims directly from JWT (OAuth2 Resource Server).

---

# ⭐ Spring Security for Microservices (Real-World Patterns)

---

# **1️⃣5️⃣ @PermitAll**

Allows anonymous access.

```java
@PermitAll
@GetMapping("/public")
public String publicApi() { return "ok"; }
```

---

# **1️⃣6️⃣ @DenyAll**

Blocks access.

```java
@DenyAll
public void removeEverything() {}
```

---

# **1️⃣7️⃣ @CrossOrigin**

Enables CORS (not strictly security, but part of web security config).

```java
@CrossOrigin(origins = "https://frontend.com")
@GetMapping("/orders")
public List<Order> orders() { ... }
```

---

# ⚙ **Most Important Spring Boot Security Test Annotations**

---

## @WithMockUser

## @WithAnonymousUser

## @WithSecurityContext

## @SpringBootTest + @AutoConfigureMockMvc

Example:

```java
@Test
@WithMockUser(roles = "USER")
void shouldAccessUserEndpoint() throws Exception {
   mockMvc.perform(get("/user"))
          .andExpect(status().isOk());
}
```

---

# 🚨 Common Interview Questions (with crisp answers)

---

### **1️⃣ What is the difference between @PreAuthorize and @Secured?**

✔ `@PreAuthorize` → expression-based, more powerful  
✔ `@Secured` → role-based only

---

### **2️⃣ Should we use method-level or endpoint-level security?**

**Best Practice:** Both, but method-level ensures business logic is fully protected even if endpoint mappings change.

---

### **3️⃣ How does @AuthenticationPrincipal work?**

Injects the `UserDetails`, `Jwt`, or custom principal extracted from the security context.

---

### **4️⃣ What is @EnableMethodSecurity used for?**

Enables annotations like:

- @PreAuthorize
    
- @PostAuthorize
    
- @Secured
    
- @RolesAllowed
    

---

### **5️⃣ Can we use @PreAuthorize inside service layer?**

Absolutely — this is the **best practice** for business-security.

---

### **6️⃣ How to secure a REST endpoint for specific roles?**

```java
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/admin")
public String admin() { return "secured"; }
```

---

# 🧠 Real Project Example (Modern SecurityFilterChain)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/public/**").permitAll()
                        .anyRequest().authenticated()
                )
                .oauth2ResourceServer(oauth -> oauth.jwt())
                .build();
    }
}
```

---

# 🎉 Done — The complete Spring Security annotation guide

If you want, next we can continue the annotation series with:

✔ Jackson JSON Annotations  
✔ Resilience4j + Spring Cloud annotations  
✔ REST Controller annotations  
✔ JPA/Hibernate annotations  
✔ Microservices-specific annotations

Here are **microservices-specific annotations** you frequently use in Spring Boot + Spring Cloud–based distributed systems.  
I’ll group them by **Service Discovery**, **Configuration**, **API Gateway**, **Client-side Load Balancing**, **Fault Tolerance**, **Distributed Tracing**, **Messaging**, and **Cloud-Native Patterns**.

---

# ✅ **Microservices-Specific Annotations**

---

# 🔵 **1. Service Discovery (Eureka, Consul, Zookeeper)**

### **@EnableEurekaServer**

Turns a Spring Boot application into a Eureka Server.

```java
@EnableEurekaServer
@SpringBootApplication
public class DiscoveryServerApplication {}
```

### **@EnableDiscoveryClient**

Registers the service with Eureka/Consul/Zookeeper.

```java
@EnableDiscoveryClient
@SpringBootApplication
public class ProductServiceApplication {}
```

### **Use Case**

- Service registration & lookup
    
- Dynamic scaling without hardcoded URLs
    

---

# 🔵 **2. Spring Cloud API Gateway**

### **@EnableGateway**

(For Spring Cloud Gateway – older versions require it)

```java
@EnableGateway
@SpringBootApplication
public class ApiGatewayApplication {}
```

### **Global Filters often use @Component**

```java
@Component
public class AuthFilter implements GlobalFilter {}
```

---

# 🔵 **3. OpenFeign – Declarative HTTP Client**

### **@EnableFeignClients**

Enables Feign client scanning.

```java
@EnableFeignClients
@SpringBootApplication
public class OrderService {}
```

### **@FeignClient**

Declarative REST client.

```java
@FeignClient(name = "payment-service", path = "/payments")
public interface PaymentClient {

    @GetMapping("/{id}")
    PaymentResponse getPayment(@PathVariable String id);
}
```

### **Use Case**

- Replaces RestTemplate/WebClient for inter-service communication
    
- Built-in load balancing with Spring Cloud LoadBalancer
    

---

# 🔵 **4. Load Balancing (Spring Cloud LoadBalancer)**

### **@LoadBalanced**

Injects client-side load balancer for RestTemplate/WebClient.

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

---

# 🔵 **5. Resilience4j (Circuit Breaker, Retry, Bulkhead, RateLimiter)**

### **@CircuitBreaker**

```java
@CircuitBreaker(name = "inventoryCircuitBreaker", fallbackMethod = "fallback")
public Inventory getInventory(String id) { ... }
```

### **@Retry**

```java
@Retry(name = "inventoryRetry")
public String callApi() { ... }
```

### **@RateLimiter**

```java
@RateLimiter(name = "productRateLimiter")
public Product getProduct() { ... }
```

### **@Bulkhead**

```java
@Bulkhead(name = "productBulkhead", type = Bulkhead.Type.SEMAPHORE)
public Product fetchProduct() { ... }
```

---

# 🔵 **6. Distributed Configuration (Spring Cloud Config)**

### **@EnableConfigServer**

```java
@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {}
```

### **@RefreshScope**

Re-fetches config without restart.

```java
@RefreshScope
@RestController
public class MessageController {

    @Value("${service.message}")
    private String msg;
}
```

---

# 🔵 **7. Distributed Tracing (Sleuth / Micrometer Tracing)**

### **@NewSpan**

Manually create a tracing span.

```java
@NewSpan
public void processOrder() {}
```

---

# 🔵 **8. Messaging (Kafka, RabbitMQ, Event-Driven Microservices)**

### **Kafka @KafkaListener**

```java
@KafkaListener(topics = "order-events")
public void consume(String message) { ... }
```

### **RabbitMQ @RabbitListener**

```java
@RabbitListener(queues = "payment-queue")
public void receive(String msg) { ... }
```

---

# 🔵 **9. Validation in Microservice DTOs (For API Contracts)**

### Common:

- `@Valid`
    
- `@Validated`
    
- `@NotNull`, `@NotEmpty`, `@Positive`, etc.
    

```java
@PostMapping
public Response create(@Valid @RequestBody CreateOrderRequest req) {}
```

---

# 🔵 **10. Serialization / REST API Response Tuning (Jackson)**

- `@JsonIgnore`
    
- `@JsonProperty`
    
- `@JsonInclude`
    
- `@JsonFormat`
    

Used heavily in DTOs across services.

---

# 🔵 **11. Spring Data JPA for Microservice Persistence**

Though not microservice-specific, commonly used:

- `@Entity`
    
- `@Id`
    
- `@GeneratedValue`
    
- `@Repository`
    

---

# 🔵 **12. Spring Boot Admin (Microservice Monitoring)**

### **@EnableAdminServer**

```java
@EnableAdminServer
@SpringBootApplication
public class AdminServer {}
```

---

# 🔵 **13. Distributed Locking (Redis, Hazelcast)**

### **@SchedulerLock (ShedLock)**

Used to avoid duplicate job execution across instances.

```java
@SchedulerLock(name = "inventorySyncJob")
@Scheduled(fixedDelay = 10000)
public void sync() { ... }
```

---

# 📌 **If You Want, I Can Provide:**

✔ Full real-world microservice diagrams  
✔ Sample project combining Feign + Resilience4j + Eureka + Gateway  
✔ Production-level code examples  
✔ Low-level design & HLD for microservices architecture

# Full microservice sample — **Feign + Resilience4j + Eureka + Gateway**

Below is a **complete, production-minded blueprint**: project structure, HLD, LLD, key code/config snippets, resilience & caching integration, CI/infra notes, and testing suggestions. You can use this as a template to implement a resilient microservice ecosystem.

---

# 1 — Executive summary (what you get)

A small microservice ecosystem containing:

- **Discovery**: Eureka Server
    
- **Config**: Spring Cloud Config Server (centralized properties)
    
- **API Gateway**: Spring Cloud Gateway (routing, auth filter, rate-limiting hooks)
    
- **Services**: `order-service`, `inventory-service`, `product-service` (example)
    
- **Client**: Feign clients for inter-service calls
    
- **Resilience**: Resilience4j annotations (`@CircuitBreaker`, `@Retry`, `@RateLimiter`, `@TimeLimiter`, `@Bulkhead`)
    
- **Caching**: Redis-backed caching via Spring Cache (`@Cacheable`, `@CachePut`, `@CacheEvict`)
    
- **Observability**: Micrometer metrics, tracing hooks (instrumentation-ready)
    
- **Dev ops**: Docker + Kubernetes-ready manifest notes, health checks, readiness/liveness
    

---

# 2 — High-Level Architecture (HLD)

Clients → **API Gateway** → (service discovery) → **order-service** ↔ **inventory-service** & **product-service**  
Shared infra:

- **Eureka** (service registry)
    
- **Config Server** (properties)
    
- **Redis** (cache)
    
- **Prometheus + Grafana** (metrics)
    
- **Jaeger** (tracing)
    
- **Kafka** (optional event bus)
    

Key flows:

- Client calls Gateway `/api/orders` → gateway route to `order-service`.
    
- `order-service` calls `inventory-service` via Feign. Feign client is decorated with Resilience4j policies.
    
- `product-service` provides product details and is cached in Redis with `@Cacheable`.
    
- Failures: `order-service` fallback path uses cached data or returns graceful error.
    

Non-functional requirements:

- Resiliency: circuit breakers, retries with backoff, bulkheads.
    
- Scalability: stateless services, sticky caches where needed (Redis).
    
- Observability: metrics per endpoint, distributed tracing.
    
- Security: OAuth2/JWT at gateway, method-level `@PreAuthorize` in services.
    

---

# 3 — Project layout (multi-module Maven)

```
microservices-root/
├─ config-server/
├─ eureka-server/
├─ gateway/
├─ common/
│  ├─ libs: dto, exceptions, feign-config, resilience-config
├─ order-service/
├─ inventory-service/
├─ product-service/
├─ docker-compose.yml
├─ k8s/ (manifests)
└─ infra/ (prometheus, grafana, jaeger)
```

`common` is a dependency for DTOs, constants, and centralized configs.

---

# 4 — Key dependencies (Maven snippets)

Add to each service `pom.xml`:

```xml
<!-- Spring Boot + Cloud -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<!-- Resilience4j -->
<dependency>
  <groupId>io.github.resilience4j</groupId>
  <artifactId>resilience4j-spring-boot2</artifactId>
</dependency>

<!-- Caching (Redis) -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- Metrics/Tracing -->
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
<dependency>
  <groupId>io.opentelemetry.instrumentation</groupId>
  <artifactId>opentelemetry-spring-boot-starter</artifactId>
</dependency>
```

Gateway additionally:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

Eureka server:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

Config server:

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

---

# 5 — Example configurations (application.yml)

## 5.1 Eureka server (eureka-server/src/main/resources/application.yml)

```yaml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    wait-time-in-ms-when-sync-empty: 0
```

## 5.2 Config server (config-server)

```yaml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/micro-configs
```

## 5.3 Gateway (gateway/application.yml)

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

spring:
  cloud:
    gateway:
      routes:
        - id: order
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - RequestRateLimiter=redis-rate-limiter[10,20]  # optional
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
```

## 5.4 Order service (order-service/application.yml)

```yaml
server:
  port: 8090
spring:
  application:
    name: order-service

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

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
        slidingWindowSize: 20
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
  retry:
    instances:
      inventoryService:
        maxAttempts: 3
        waitDuration: 2s
management:
  metrics:
    export:
      prometheus:
        enabled: true
```

## 5.5 Redis (application.yml)

```yaml
spring:
  redis:
    host: redis
    port: 6379
```

---

# 6 — Key code snippets

## 6.1 Eureka server main

```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
  public static void main(String[] args) {
    SpringApplication.run(DiscoveryServerApplication.class, args);
  }
}
```

## 6.2 Config server main

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication { ... }
```

## 6.3 Gateway - global auth filter (sample)

```java
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // Validate JWT token; reject if unauthorized
        return chain.filter(exchange);
    }
    @Override public int getOrder() { return -1; }
}
```

## 6.4 Feign client (inventory)

`common` module -> `dto` share types

```java
@FeignClient(name = "inventory-service", path = "/api/inventory")
public interface InventoryClient {
    @GetMapping("/{productId}")
    InventoryDto getInventory(@PathVariable("productId") String productId);
}
```

## 6.5 OrderService using Resilience4j + Feign

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final InventoryClient inventoryClient;

    @CircuitBreaker(name = "inventoryService", fallbackMethod = "inventoryFallback")
    @Retry(name = "inventoryService")
    public OrderConfirmation placeOrder(String productId, int qty) {
        InventoryDto inv = inventoryClient.getInventory(productId);
        if (inv.getAvailable() < qty) throw new InsufficientStockException();
        // create order...
        return new OrderConfirmation(...);
    }

    private OrderConfirmation inventoryFallback(String productId, int qty, Throwable ex) {
        // Try cache or return user-friendly response
        return new OrderConfirmation("fallback", ...);
    }
}
```

## 6.6 Product service caching (Redis)

```java
@Service
@CacheConfig(cacheNames = "products")
@RequiredArgsConstructor
public class ProductService {
    private final ProductRepository repo;

    @Cacheable(key = "#id")
    public ProductDto getProduct(String id) {
        simulateSlowDB();
        return repo.findById(id).orElseThrow();
    }

    @CachePut(key = "#product.id")
    public ProductDto update(ProductDto product) {
        return repo.save(product);
    }
}
```

---

# 7 — Resilience4j config (example YAML)

```yaml
resilience4j:
  circuitbreaker:
    instances:
      inventoryService:
        slidingWindowSize: 50
        minimumNumberOfCalls: 10
        failureRateThreshold: 60
        waitDurationInOpenState: 30s
  retry:
    instances:
      inventoryService:
        maxAttempts: 4
        waitDuration: 2s
  timelimiter:
    instances:
      inventoryService:
        timeoutDuration: 2s
```

---

# 8 — Caching strategy (recommendation)

- Use Redis as central cache for distributed services.
    
- Cache read-heavy product catalog with `@Cacheable`.
    
- Cache invalidation: `@CacheEvict` on update/delete paths in product-service and publish an event to notify other services if needed.
    
- Cache warm-up: populate on startup if cold start is a concern.
    
- Set TTLs per business rule (e.g., product details 15m, inventory 30s).
    

---

# 9 — Observability & Metrics

- Expose `/actuator/prometheus` on services for Prometheus scraping.
    
- Instrument methods with Micrometer `@Timed` or `Timer` code.
    
- Add tracing by propagating HTTP headers (W3C traceparent) via Feign config and Gateway forwards.
    
- Use Resilience4j metrics integration — circuit breaker open/close counts visible in Prometheus.
    

---

# 10 — Security (recommended)

- Token validation at Gateway: API Gateway validates JWT and forwards claims to services.
    
- Use `@EnableMethodSecurity` and `@PreAuthorize("hasRole('ORDER_CREATE')")` on service methods for fine-grained control.
    
- Protect actuator endpoints with appropriate roles.
    

---

# 11 — Dev / Infra (containers, k8s)

**docker-compose.yml** quick stacks:

- config-server, eureka-server, redis, prometheus, grafana, jaeger, gateway, services
    

**Kubernetes**:

- Each service: Deployment + Service + HorizontalPodAutoscaler
    
- Use readinessProbe (HTTP `/actuator/health`) and livenessProbe.
    
- Use ClusterIP for Redis, Headless service for stateful, Ingress for external access (gateway via Ingress).
    

---

# 12 — CI / CD & Testing

- Unit tests: pure unit (Mockito) for service logic. Test resilience fallback logic.
    
- Contract tests: Pact or Spring Cloud Contract between `order` and `inventory`.
    
- Integration tests: `@SpringBootTest` with `@Testcontainers` (Redis, Eureka replaced by local testcontainers or WireMock for feign).
    
- E2E tests: run against docker-compose or dev k8s namespace.
    
- CI: build, run unit tests, build images, push to registry; CD: helm charts deploy to staging.
    

---

# 13 — Example of integration test (Testcontainers + Eureka stub)

Use Testcontainers for Redis + WireMock for feign target:

```java
@SpringBootTest
@Testcontainers
public class OrderServiceIntegrationTest {

  @Container
  static GenericContainer redis = new GenericContainer("redis:6.2").withExposedPorts(6379);

  @BeforeAll
  static void setup() {
    System.setProperty("spring.redis.host", redis.getHost());
    System.setProperty("spring.redis.port", redis.getFirstMappedPort().toString());
  }

  @MockBean
  InventoryClient inventoryClient;

  @Autowired OrderService orderService;

  @Test
  void placeOrderFallback_onInventoryDown() {
    when(inventoryClient.getInventory(anyString()))
       .thenThrow(new RuntimeException("down"));

    OrderConfirmation result = orderService.placeOrder("p1", 1);
    assertEquals("fallback", result.getStatus());
  }
}
```

---

# 14 — Operational runbook (quick points)

- Monitor circuit breakers; configure alerts when a CB opens repeatedly.
    
- Monitor error rates & latencies.
    
- Use health endpoints to detect dependency failures (e.g., Redis down).
    
- For cache incoherence, use versioned keys or events to invalidate.
    

---

# 15 — Next steps (pick one)

- I can **generate the full Maven multi-module skeleton** with sample code for each module.
    
- Or produce **Kubernetes manifests + Helm chart** for this stack.
    
- Or produce a **complete Order → Inventory Feign + resilience integrated codebase** (full files ready to paste).
    
[[Resilience4j]]

