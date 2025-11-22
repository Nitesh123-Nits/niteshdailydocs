
---

# Low-Level Design of Logger System

A **Logger System** is a fundamental component in any software system. It allows tracking application behavior, errors, and operational events. A well-designed logger should be flexible, extensible, and decoupled.

---

## **1. Requirements**

**Functional Requirements:**

1. Support logging messages with levels: INFO, DEBUG, ERROR, WARN.
    
2. Allow logging to multiple destinations: console, file, network, database.
    
3. Enable formatting: JSON, plain text, XML.
    
4. Support dynamic configuration of log levels.
    
5. Allow thread-safe logging.
    

**Non-Functional Requirements:**

- High performance (non-blocking or asynchronous logging).
    
- Extensible for future output types or formats.
    
- Decoupled design following SOLID principles.
    

---

## **2. High-Level Architecture**

```
Client → Logger → Formatter → Appender → Storage (Console/File/DB/Network)
```

- **Logger**: Entry point for logging requests.
    
- **Formatter**: Converts the message to a specific format.
    
- **Appender/Handler**: Sends the log to the output destination.
    

---

## **3. Design Patterns Used**

|Pattern|Purpose in Logger System|Why Used|
|---|---|---|
|**Singleton**|Ensure a single instance of `Logger`|Avoids multiple log instances writing inconsistently; thread-safe global access|
|**Strategy**|Implement different `Formatter`s (JSON, XML, PlainText)|Allows switching formatting behavior at runtime without changing Logger code|
|**Factory / Abstract Factory**|Create different types of `Appender`s (ConsoleAppender, FileAppender, DBAppender)|Decouples logger from concrete output implementations; allows easy addition of new output types|
|**Observer**|Notify multiple appenders of a log event|Enables broadcasting log messages to multiple destinations without coupling logger to specific appenders|
|**Builder**|Build complex log messages|Flexible and readable creation of structured log entries|
|**Decorator**|Add additional behavior to log messages (e.g., timestamp, thread info)|Allows enhancing log content without changing base logger logic|

---

## **4. Class Design (OOP & SOLID)**

### **Logger Class (Singleton, Open/Closed Principle)**

```java
public class Logger {
    private static volatile Logger instance;
    private List<Appender> appenders = new ArrayList<>();
    private LogLevel currentLevel = LogLevel.INFO;

    private Logger() {}

    public static Logger getInstance() {
        if (instance == null) {
            synchronized (Logger.class) {
                if (instance == null) {
                    instance = new Logger();
                }
            }
        }
        return instance;
    }

    public void addAppender(Appender appender) {
        appenders.add(appender);
    }

    public void log(LogLevel level, String message) {
        if (level.ordinal() >= currentLevel.ordinal()) {
            LogEntry entry = new LogEntry(level, message);
            appenders.forEach(appender -> appender.append(entry));
        }
    }

    public void setLevel(LogLevel level) {
        this.currentLevel = level;
    }
}
```

**SOLID Principles Applied:**

- **S**: Single Responsibility — `Logger` only orchestrates logging.
    
- **O**: Open/Closed — New appenders/formatters can be added without modifying existing code.
    
- **L**: Liskov — Subtypes of `Appender` can replace the base `Appender` without breaking Logger.
    
- **I**: Interface Segregation — Separate interfaces for `Appender` and `Formatter`.
    
- **D**: Dependency Inversion — Logger depends on abstractions (`Appender`, `Formatter`) not concrete implementations.
    

---

### **Appender Interface (Observer / Factory)**

```java
public interface Appender {
    void append(LogEntry entry);
}
```

**Concrete Appenders**

```java
public class ConsoleAppender implements Appender {
    private Formatter formatter;

    public ConsoleAppender(Formatter formatter) {
        this.formatter = formatter;
    }

    @Override
    public void append(LogEntry entry) {
        System.out.println(formatter.format(entry));
    }
}

public class FileAppender implements Appender {
    private Formatter formatter;
    private Path filePath;

    public FileAppender(Formatter formatter, Path filePath) {
        this.formatter = formatter;
        this.filePath = filePath;
    }

    @Override
    public void append(LogEntry entry) {
        try {
            Files.writeString(filePath, formatter.format(entry) + "\n",
                    StandardOpenOption.APPEND, StandardOpenOption.CREATE);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### **Formatter Interface (Strategy Pattern)**

```java
public interface Formatter {
    String format(LogEntry entry);
}

public class JsonFormatter implements Formatter {
    @Override
    public String format(LogEntry entry) {
        return String.format("{\"level\":\"%s\",\"message\":\"%s\",\"time\":\"%s\"}",
                entry.getLevel(), entry.getMessage(), entry.getTimestamp());
    }
}

public class SimpleFormatter implements Formatter {
    @Override
    public String format(LogEntry entry) {
        return entry.getTimestamp() + " [" + entry.getLevel() + "] " + entry.getMessage();
    }
}
```

---

### **LogEntry (Builder + Immutable Design)**

```java
public class LogEntry {
    private final LogLevel level;
    private final String message;
    private final LocalDateTime timestamp;

    public LogEntry(LogLevel level, String message) {
        this.level = level;
        this.message = message;
        this.timestamp = LocalDateTime.now();
    }

    public LogLevel getLevel() { return level; }
    public String getMessage() { return message; }
    public LocalDateTime getTimestamp() { return timestamp; }
}
```

---

## **5. Usage Example**

```java
public class LoggerDemo {
    public static void main(String[] args) {
        Logger logger = Logger.getInstance();

        Formatter jsonFormatter = new JsonFormatter();
        Formatter simpleFormatter = new SimpleFormatter();

        logger.addAppender(new ConsoleAppender(simpleFormatter));
        logger.addAppender(new FileAppender(jsonFormatter, Path.of("app.log")));

        logger.log(LogLevel.INFO, "Application started");
        logger.log(LogLevel.ERROR, "An unexpected error occurred");
    }
}
```

---

## **6. Extensibility**

1. **New Appenders**: Just implement the `Appender` interface (e.g., DatabaseAppender, NetworkAppender).
    
2. **New Formats**: Implement `Formatter` (e.g., XMLFormatter).
    
3. **Dynamic Configuration**: Log levels and appenders can be added or removed at runtime.
    
4. **Thread-Safety / Async Logging**: Can wrap appenders with asynchronous queue or executor service without changing core Logger code.
    
5. **Decorator Support**: Wrap existing appenders to add additional info (thread ID, context, correlation ID) dynamically.
    

---

## **7. Advantages of This Design**

- **Flexible**: Can log to multiple destinations with multiple formats.
    
- **Extensible**: New log destinations or formats do not require core changes.
    
- **Decoupled**: Logger only depends on abstractions (Appender, Formatter).
    
- **Thread-safe**: Singleton + synchronized initialization ensures global consistency.
    
- **SOLID & Clean Code Compliant**: Each class has a single responsibility, dependencies are inverted, open/closed principle followed.
    

---

This Logger system demonstrates production-ready **low-level design**, combining classic design patterns with SOLID principles, clean coding practices, and maximum extensibility—exactly the kind of LLD question you’ll face in interviews.

---
