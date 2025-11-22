Here’s a detailed low-level design (LLD) interview-style blog for a **Cache System**, structured to reflect a professional interview explanation. It covers OOP principles, SOLID design, design patterns, clean coding practices, extensibility, and reasoning behind choices.

---

# **LLD of a Cache System – Interview-Style Blog**

## **1. Introduction**

Caching is critical in software systems for improving performance, reducing latency, and decreasing load on databases or external services. A **Cache System** stores frequently accessed data in memory, making it quickly retrievable.

Typical interview questions around cache LLD focus on:

- Designing a thread-safe cache.
    
- Supporting eviction policies (LRU, LFU, FIFO).
    
- Scalability and distributed cache considerations.
    
- Extensibility for future policies or persistence.
    

---

## **2. Requirements**

**Functional Requirements:**

1. Store key-value pairs.
    
2. Retrieve value by key.
    
3. Eviction policies: LRU, LFU.
    
4. Optional TTL (time-to-live) support.
    
5. Maximum cache size limit.
    

**Non-Functional Requirements:**

- High performance (O(1) retrieval and insertion ideally).
    
- Thread-safe operations.
    
- Easy extensibility for adding new eviction policies or persistence layers.
    

---

## **3. High-Level Design**

We can design the cache using the following components:

1. **Cache Interface** – Defines basic operations (`get`, `put`, `delete`).
    
2. **EvictionPolicy Interface** – Strategy for eviction (`LRU`, `LFU`).
    
3. **InMemoryCache** – Core implementation of cache using a combination of `HashMap` + `DoublyLinkedList` for LRU.
    
4. **Persistence Layer (Optional)** – For write-through or write-back strategies.
    
5. **TTL Manager** – Handles expiry of cache entries.
    
6. **Factory Class** – To instantiate cache with specific eviction policies.
    

---

## **4. Class Diagram (Simplified)**

```
+----------------+        +-------------------+
|    Cache<K,V>  |<-------|  EvictionPolicy   |
+----------------+        +-------------------+
| +get(key)      |        | +keyAccessed(key) |
| +put(key, val) |        | +evict()          |
| +delete(key)   |        +-------------------+
+----------------+
        ^
        |
+--------------------+
|   InMemoryCache    |
+--------------------+
| -map: HashMap      |
| -policy: EvictionPolicy |
+--------------------+
| +get()             |
| +put()             |
| +delete()          |
+--------------------+
```

---

## **5. Design Patterns Used**

|Design Pattern|Where Used|Why Used / Advantage|
|---|---|---|
|**Strategy Pattern**|Eviction policies (LRU, LFU)|Allows runtime switching of eviction policies without modifying core cache logic.|
|**Singleton Pattern**|Cache manager (optional for global cache)|Ensures a single instance of cache across application.|
|**Factory Pattern**|Creating cache with specific eviction policy|Decouples object creation from implementation, easy to extend for new policies.|
|**Observer Pattern**|TTL expiry notifications|Allows automatic removal of expired keys from cache.|
|**Decorator Pattern**|Optional persistence layer or metrics|Enhances cache with additional behavior (logging, metrics, persistence) without modifying core logic.|

---

## **6. Applying OOP Principles and SOLID**

1. **Single Responsibility Principle (SRP)**
    
    - `Cache` handles storage and retrieval.
        
    - `EvictionPolicy` handles eviction.
        
    - `TTLManager` handles expiry.
        
2. **Open/Closed Principle (OCP)**
    
    - Cache is open for extension: new eviction policies can be added without modifying `InMemoryCache`.
        
3. **Liskov Substitution Principle (LSP)**
    
    - Any `EvictionPolicy` implementation can be substituted without affecting cache behavior.
        
4. **Interface Segregation Principle (ISP)**
    
    - Separate interfaces for cache operations and eviction logic (`Cache` and `EvictionPolicy`).
        
5. **Dependency Inversion Principle (DIP)**
    
    - `InMemoryCache` depends on `EvictionPolicy` abstraction, not concrete implementations.
        

---

## **7. Sample Code Snippet**

```java
// EvictionPolicy.java
public interface EvictionPolicy<K> {
    void keyAccessed(K key);
    K evictKey();
}

// LRU implementation
public class LRUEvictionPolicy<K> implements EvictionPolicy<K> {
    private final Deque<K> deque = new LinkedList<>();
    private final Set<K> set = new HashSet<>();

    @Override
    public void keyAccessed(K key) {
        if (set.contains(key)) {
            deque.remove(key);
        } else {
            set.add(key);
        }
        deque.addFirst(key);
    }

    @Override
    public K evictKey() {
        K evict = deque.removeLast();
        set.remove(evict);
        return evict;
    }
}

// Cache.java
public interface Cache<K, V> {
    V get(K key);
    void put(K key, V value);
    void delete(K key);
}

// InMemoryCache.java
public class InMemoryCache<K, V> implements Cache<K, V> {
    private final Map<K, V> map = new HashMap<>();
    private final EvictionPolicy<K> evictionPolicy;
    private final int capacity;

    public InMemoryCache(int capacity, EvictionPolicy<K> policy) {
        this.capacity = capacity;
        this.evictionPolicy = policy;
    }

    @Override
    public V get(K key) {
        if (!map.containsKey(key)) return null;
        evictionPolicy.keyAccessed(key);
        return map.get(key);
    }

    @Override
    public void put(K key, V value) {
        if (map.size() >= capacity) {
            K evictKey = evictionPolicy.evictKey();
            map.remove(evictKey);
        }
        map.put(key, value);
        evictionPolicy.keyAccessed(key);
    }

    @Override
    public void delete(K key) {
        map.remove(key);
    }
}
```

---

## **8. Clean Coding Practices Applied**

- Meaningful class, method, and variable names.
    
- Separation of concerns.
    
- Encapsulation of cache and eviction logic.
    
- Interface-driven design.
    
- Minimal coupling, high cohesion.
    

---

## **9. Extensibility**

- **New eviction policy:** Implement `EvictionPolicy` and inject via factory.
    
- **Distributed caching:** Wrap `InMemoryCache` with a decorator for Redis/Memcached.
    
- **Persistence:** Add a decorator for write-through/write-back to DB.
    
- **TTL support:** Observer pattern to remove expired entries without blocking cache operations.
    

---

## **10. Advantages of This Design**

- **Flexible:** Can switch eviction policies at runtime.
    
- **Thread-safe implementation possible:** Using `ConcurrentHashMap` and `ReentrantLock`.
    
- **Maintainable:** Follows SOLID and clean coding principles.
    
- **Extensible:** Easy to add TTL, metrics, or persistence features.
    

---

## **11. Conclusion**

Designing a cache system in LLD interviews tests your understanding of:

- **OOP and SOLID principles.**
    
- **Core design patterns**: Strategy, Singleton, Factory, Observer, Decorator.
    
- **Clean code** and separation of concerns.
    
- **Extensibility** for future features.
    

The key is to **decouple responsibilities**, **abstract policies**, and **ensure high performance** while maintaining maintainable and scalable code.

---

