
---

# Low-Level Design of a BookMyShow-like Ticket Booking System

A BookMyShow-like platform allows users to search for movies, view showtimes, book tickets, and manage payments. Designing such a system requires careful modeling of **domain objects, services, and interactions**, keeping scalability and flexibility in mind.

---

## 1. **Core Requirements**

**Functional Requirements:**

1. Users can browse movies, shows, and theaters.
    
2. Users can book tickets for specific shows.
    
3. Users can select seats (seat availability, locking mechanism).
    
4. Payments are processed and bookings are confirmed.
    
5. Admins can add/update movies, shows, and theaters.
    
6. Users can cancel bookings and get refunds.
    

**Non-Functional Requirements:**

- High concurrency (multiple users booking same seats).
    
- Extensibility (adding new booking types or payment methods).
    
- Maintainable and modular design.
    
- Easy integration with external services (payment gateways, notifications).
    

---

## 2. **Domain Modeling (Classes & Objects)**

Here are the key domain entities:

```java
class User {
    String userId;
    String name;
    String email;
    List<Booking> bookings;
}

class Movie {
    String movieId;
    String name;
    String genre;
    Duration duration;
}

class Theater {
    String theaterId;
    String name;
    List<Screen> screens;
}

class Screen {
    String screenId;
    int totalSeats;
    List<Seat> seats;
}

class Seat {
    String seatId;
    SeatType type; // REGULAR, PREMIUM, VIP
    boolean isBooked;
}

class Show {
    String showId;
    Movie movie;
    Screen screen;
    LocalDateTime startTime;
    LocalDateTime endTime;
    Map<Seat, Boolean> seatAvailability;
}

class Booking {
    String bookingId;
    User user;
    Show show;
    List<Seat> bookedSeats;
    Payment payment;
    BookingStatus status;
}

class Payment {
    String paymentId;
    double amount;
    PaymentStatus status;
    PaymentMethod method; // CARD, UPI, WALLET
}
```

> This domain model adheres to **OOP principles**:
> 
> - **Encapsulation:** Data and behavior grouped inside classes.
>     
> - **Single Responsibility (SRP):** Each class has one responsibility.
>     
> - **Abstraction:** Exposing only necessary methods to interact with objects.
>     
> - **Open/Closed Principle (OCP):** Easy to extend functionality (e.g., new seat types, payment methods) without modifying existing code.
>     

---

## 3. **Services & Interfaces**

```java
interface BookingService {
    Booking bookTickets(User user, Show show, List<Seat> seats);
    void cancelBooking(String bookingId);
}

interface PaymentService {
    PaymentStatus processPayment(Payment payment);
}

interface NotificationService {
    void notifyUser(User user, String message);
}
```

- Using **interfaces** ensures **Dependency Inversion (DIP)** and **OCP**.
    
- Allows easy swapping of implementations (e.g., different payment gateways or notification channels).
    

---

## 4. **Design Patterns Used**

### a) **Singleton Pattern**

- **Use Case:** Payment gateway handler, Notification service.
    
- **Why:** Ensure only one instance manages external communication and shared resources.
    
- **Advantage:** Resource optimization and global access.
    

```java
public class PaymentGateway {
    private static PaymentGateway instance;

    private PaymentGateway() {}

    public static PaymentGateway getInstance() {
        if(instance == null) instance = new PaymentGateway();
        return instance;
    }
}
```

---

### b) **Factory Pattern / Abstract Factory**

- **Use Case:** Creating Payment objects or Booking objects based on type.
    
- **Why:** Decouples creation logic from usage, supports multiple types of payments or bookings.
    
- **Advantage:** OCP – easy to add new payment types or booking types without changing client code.
    

```java
public interface PaymentFactory {
    Payment createPayment(double amount);
}

public class CardPaymentFactory implements PaymentFactory {
    public Payment createPayment(double amount) {
        return new CardPayment(amount);
    }
}
```

---

### c) **Strategy Pattern**

- **Use Case:** Different payment methods (Card, UPI, Wallet), seat selection algorithms.
    
- **Why:** Encapsulates interchangeable behaviors (e.g., payment processing) and selects at runtime.
    
- **Advantage:** Extensible; easy to add new payment strategies without modifying existing logic.
    

```java
interface PaymentStrategy {
    PaymentStatus pay(Payment payment);
}

class CardPaymentStrategy implements PaymentStrategy {
    public PaymentStatus pay(Payment payment) { /* logic */ }
}

class UpiPaymentStrategy implements PaymentStrategy {
    public PaymentStatus pay(Payment payment) { /* logic */ }
}
```

---

### d) **Observer Pattern**

- **Use Case:** Notify users about booking confirmation, seat availability.
    
- **Why:** Promotes **loose coupling** between booking service and notification service.
    
- **Advantage:** Adding new notification channels is easy.
    

```java
interface Observer { void update(String message); }
class EmailNotification implements Observer { /* implement update */ }
class BookingServiceSubject {
    List<Observer> observers;
    void addObserver(Observer o) { observers.add(o); }
    void notifyAll(String message) { for(Observer o : observers) o.update(message); }
}
```

---

### e) **Decorator Pattern**

- **Use Case:** Adding extra services to bookings (e.g., snacks, premium seating).
    
- **Why:** Enhances functionality dynamically without altering core Booking class.
    
- **Advantage:** Flexible feature addition, adheres to OCP.
    

```java
abstract class BookingDecorator extends Booking {
    protected Booking booking;
}

class SnackBookingDecorator extends BookingDecorator {
    public double getCost() { return booking.getCost() + 100; }
}
```

---

### f) **Command Pattern**

- **Use Case:** Encapsulating booking/cancellation actions, supporting undo operations.
    
- **Why:** Decouples invoker from execution logic.
    
- **Advantage:** Useful for audit, retry, or rollback operations.
    

---

### g) **Facade Pattern**

- **Use Case:** BookingFacade to handle all booking-related operations like seat selection, payment, and notification.
    
- **Why:** Simplifies complex subsystem interactions.
    
- **Advantage:** Hides internal complexity from client.
    

```java
class BookingFacade {
    private BookingService bookingService;
    private PaymentService paymentService;
    private NotificationService notificationService;

    public Booking makeBooking(User user, Show show, List<Seat> seats) {
        Booking booking = bookingService.bookTickets(user, show, seats);
        paymentService.processPayment(booking.getPayment());
        notificationService.notifyUser(user, "Booking confirmed");
        return booking;
    }
}
```

---

## 5. **Concurrency & Scalability**

- **Seat Locking:** Use **Optimistic Locking** or **Pessimistic Locking** in DB to prevent double booking.
    
- **Caching:** Frequently accessed data like movie schedules can be cached.
    
- **Microservices Consideration:** Booking, Payment, and Notification can be separate services for horizontal scaling.
    

---

## 6. **Extensibility & Maintainability**

- **Open/Closed Principle:** Add new payment methods, seat types, or notification channels without modifying existing code.
    
- **Liskov Substitution Principle:** PaymentStrategy or BookingDecorator can be extended without breaking client usage.
    
- **Interface Segregation:** Separate interfaces for PaymentService, NotificationService, BookingService ensure clients only depend on what they use.
    
- **Dependency Injection:** Allows easy swapping of strategies and services.
    

---

## 7. **Advantages of this Design**

1. **Modular:** Each responsibility is encapsulated in a single class or service.
    
2. **Flexible & Extensible:** Easy to add new features (new payment methods, booking types, promotions).
    
3. **Maintainable:** Clean separation of concerns.
    
4. **Testable:** Interfaces and strategy pattern make unit testing easy.
    
5. **Scalable:** Services can be independently scaled in microservices architecture.
    

---

## 8. **Conclusion**

This design combines **OOP principles, SOLID principles, and multiple design patterns** to build a robust, extensible BookMyShow-like platform.  
Key takeaways for an LLD interview:

- Identify **core entities and their relationships**.
    
- Define **services and interfaces** for decoupling.
    
- Apply **design patterns strategically** to solve specific problems.
    
- Ensure **extensibility, maintainability, and testability**.
    
- Think of **real-world constraints** like concurrency, caching, and microservices.
    

---