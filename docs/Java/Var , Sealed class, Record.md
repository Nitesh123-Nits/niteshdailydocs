Perfect — let’s break this down **conceptually + practically**, covering:
1️⃣ `var` usage
2️⃣ `this` keyword
3️⃣ Inner classes (member, static, local, anonymous)
4️⃣ `record` class

Each with **code snippet + explanation + interview use case** 👇

---

## 🧩 1. `var` (Local Variable Type Inference)

Introduced in **Java 10**, `var` lets the compiler infer the type of a local variable.

### ✅ Example:

```java
public class VarExample {
    public static void main(String[] args) {
        var name = "Nitesh"; // inferred as String
        var age = 28;        // inferred as int
        var list = java.util.List.of("Java", "Spring", "Microservices");

        System.out.println("Name: " + name + ", Age: " + age);
        list.forEach(System.out::println);
    }
}
```

### 💡 Notes:

* Can only be used for **local variables**, **loops**, and **try-with-resources**.
* Type must be **inferrable at compile time**.
* Not for fields, method parameters, or return types.

### 🧠 Interview Tip:

> **Q:** Why was `var` introduced?
> **A:** To improve code readability, reduce verbosity, and encourage cleaner local variable declarations.
> **Q:** Can `var` hold `null`?
> **A:** No, because the compiler can’t infer the type.

---

## 🧩 2. `this` Keyword

`this` refers to the **current instance** of the class — commonly used to:

* Differentiate between instance and local variables
* Invoke current object’s methods
* Pass the current object as a parameter

### ✅ Example:

```java
class Employee {
    private String name;
    private int id;

    public Employee(String name, int id) {
        this.name = name; // differentiates local variable from instance variable
        this.id = id;
    }

    void show() {
        System.out.println("Employee: " + this.name + ", ID: " + this.id);
    }
}

public class ThisKeywordExample {
    public static void main(String[] args) {
        Employee emp = new Employee("Nitesh", 101);
        emp.show();
    }
}
```

### 🧠 Interview Tip:

> **Q:** When do you use `this()`?
> **A:** To call another constructor in the same class.
> **Q:** Can `this` be used in a static context?
> **A:** No — static doesn’t belong to an instance.

---

## 🧩 3. Inner Classes

Inner classes provide **encapsulation** and **logical grouping** of classes that are only relevant to their enclosing class.

### Types:

1. **Non-static (Member Inner Class)**
2. **Static Nested Class**
3. **Local Inner Class (inside method)**
4. **Anonymous Inner Class**

---

### ✅ Example of All:

```java
public class OuterClass {
    private String outerField = "Outer Field";

    // 1️⃣ Member Inner Class
    class InnerClass {
        void display() {
            System.out.println("Accessing: " + outerField); // can access outer instance members
        }
    }

    // 2️⃣ Static Nested Class
    static class StaticNested {
        void show() {
            System.out.println("Static nested class can’t access outer instance fields directly");
        }
    }

    void localAndAnonymousExamples() {
        // 3️⃣ Local Inner Class
        class LocalInner {
            void print() {
                System.out.println("Inside local inner class");
            }
        }
        new LocalInner().print();

        // 4️⃣ Anonymous Inner Class
        Runnable r = new Runnable() {
            public void run() {
                System.out.println("Anonymous inner class running...");
            }
        };
        r.run();
    }

    public static void main(String[] args) {
        OuterClass outer = new OuterClass();

        OuterClass.InnerClass inner = outer.new InnerClass();
        inner.display();

        OuterClass.StaticNested staticNested = new OuterClass.StaticNested();
        staticNested.show();

        outer.localAndAnonymousExamples();
    }
}
```

### 💡 Use Cases:

* **Member Inner Class** → when class logically belongs to outer class (e.g., `Entry` in `Map`).
* **Static Nested Class** → utility/helper inside outer class without holding reference to it.
* **Local Inner Class** → encapsulate behavior used only inside one method.
* **Anonymous Inner Class** → concise instantiation for one-time use (e.g., callbacks, listeners).

### 🧠 Interview Tip:

> **Q:** Difference between static nested and inner class?
> **A:** Static nested doesn’t hold implicit reference to the outer instance.
> **Q:** Common use of anonymous class?
> **A:** In event handling, comparator definitions, etc.

---

## 🧩 4. `record` (Java 16)

Introduced in **Java 14 (preview)** and **finalized in Java 16**,
`record` provides a compact way to create **immutable data carriers**.

### ✅ Example:

```java
public record EmployeeRecord(String name, int id, double salary) {
    // You can still add methods
    public String formattedInfo() {
        return String.format("Employee[name=%s, id=%d, salary=%.2f]", name, id, salary);
    }
}

class RecordExample {
    public static void main(String[] args) {
        EmployeeRecord emp = new EmployeeRecord("Nitesh", 101, 55000.00);
        System.out.println(emp.name());
        System.out.println(emp.formattedInfo());
    }
}
```

### 💡 Notes:

* Automatically generates:

  * `private final` fields
  * Constructor
  * Getters (via component methods)
  * `toString()`, `equals()`, `hashCode()`
* Immutable by design.

### 🧠 Interview Tip:

> **Q:** Why use `record` over class?
> **A:** For **data-transfer**, **DTOs**, **value objects**, reducing boilerplate.
> **Q:** Can a record extend another class?
> **A:** No, but it can **implement interfaces**.

---

## 🔍 Summary Table

| Feature       | Version          | Main Use Case                    | Key Points                                 |
| ------------- | ---------------- | -------------------------------- | ------------------------------------------ |
| `var`         | Java 10          | Type inference                   | Local vars only                            |
| `this`        | Always           | Current instance reference       | Used for disambiguation                    |
| Inner Classes | Since early Java | Logical grouping & encapsulation | 4 types (member, static, local, anonymous) |
| `record`      | Java 16          | Immutable data carrier           | Auto-generates boilerplate                 |

---

# Sealed Class

Excellent addition — **sealed classes** (introduced in **Java 17**) are one of the most important modern Java features for **controlled inheritance** and **pattern matching readiness**.

Let’s go step-by-step just like before 👇

---

## 🧩 5. `sealed` Classes (Java 17)

### 🧠 Concept:

A **sealed class** restricts which other classes can **extend or implement it**.
It enforces a **fixed inheritance hierarchy**, improving **domain modeling**, **type safety**, and **exhaustive pattern matching**.

---

### ✅ Example:

```java
sealed class Shape permits Circle, Rectangle, Square {
    abstract double area();
}

final class Circle extends Shape {
    private final double radius;
    Circle(double radius) { this.radius = radius; }
    @Override
    double area() { return Math.PI * radius * radius; }
}

non-sealed class Rectangle extends Shape {  // can be extended further
    private final double length, breadth;
    Rectangle(double l, double b) { this.length = l; this.breadth = b; }
    @Override
    double area() { return length * breadth; }
}

final class Square extends Shape {
    private final double side;
    Square(double side) { this.side = side; }
    @Override
    double area() { return side * side; }
}

public class SealedClassExample {
    public static void main(String[] args) {
        Shape c = new Circle(5);
        Shape r = new Rectangle(4, 6);
        Shape s = new Square(3);

        System.out.println("Circle area: " + c.area());
        System.out.println("Rectangle area: " + r.area());
        System.out.println("Square area: " + s.area());
    }
}
```

---

### 🔍 Keyword Meanings:

| Modifier     | Meaning                                            |
| ------------ | -------------------------------------------------- |
| `sealed`     | restricts which classes can extend or implement it |
| `permits`    | explicitly lists the allowed subclasses            |
| `final`      | subclass cannot be further extended                |
| `non-sealed` | subclass can be freely extended by others          |

---

### 💡 Why It Exists:

Before Java 17, inheritance was **open-ended** — any class could extend a non-final class, leading to:

* Fragile hierarchies
* Uncontrolled subclassing
* Type-safety issues in pattern matching

Sealed classes solve this by **making inheritance explicit**.

---

### 🧱 Real-World Use Case:

* Modeling **finite domain hierarchies**, e.g. `Payment → CreditCardPayment, UpiPayment, CashPayment`.
* Used with **pattern matching switch** for exhaustive checks (from Java 21 onward):

```java
static String describeShape(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with area " + c.area();
        case Rectangle r -> "Rectangle with area " + r.area();
        case Square s -> "Square with area " + s.area();
    };
}
```

The compiler ensures all permitted subclasses are handled — no `default` case required. ✅

---

### 🧠 Interview Insights:

| Question                                                    | Strong Answer                                                                                                             |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Why sealed classes?**                                     | To restrict inheritance hierarchies for domain safety and predictable modeling. Helps compiler optimize pattern matching. |
| **Difference between `sealed`, `final`, and `non-sealed`?** | `final`: no subclassing allowed. `non-sealed`: open again for inheritance. `sealed`: restricted to known subclasses.      |
| **Where are sealed classes used?**                          | Financial transaction types, event modeling, protocol messages, compiler AST nodes, etc.                                  |
| **Benefits in pattern matching?**                           | Compiler can verify all subtypes are covered → safer, more expressive switches.                                           |

---

### ✅ Summary Table

| Feature            | Introduced In | Used For             | Key Advantage                                |
| ------------------ | ------------- | -------------------- | -------------------------------------------- |
| `sealed` class     | Java 17       | Restrict inheritance | Controlled hierarchy, safer pattern matching |
| `final` class      | Old           | Prevent subclassing  | Immutable type hierarchy                     |
| `non-sealed` class | Java 17       | Reopen inheritance   | Flexible hybrid modeling                     |

---

### 🧱 Example in Enterprise Modeling

```java
sealed interface Payment permits CreditCardPayment, UpiPayment, CashPayment {}

record CreditCardPayment(String cardNumber, double amount) implements Payment {}
record UpiPayment(String upiId, double amount) implements Payment {}
record CashPayment(double amount) implements Payment {}

class PaymentProcessor {
    static void process(Payment payment) {
        switch (payment) {
            case CreditCardPayment c -> System.out.println("Processing card: " + c.cardNumber());
            case UpiPayment u -> System.out.println("Processing UPI: " + u.upiId());
            case CashPayment cash -> System.out.println("Processing cash of ₹" + cash.amount());
        }
    }

    public static void main(String[] args) {
        process(new UpiPayment("nitesh@upi", 500.00));
    }
}
```

This is an **idiomatic** modern Java approach — sealed interfaces + records = **immutable, exhaustive, and clean domain modeling**.

---

