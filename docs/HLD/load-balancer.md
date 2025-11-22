![[Pasted image 20251122171613.png]]
---

# ✅ 1. HIGH-LEVEL CLASSIFICATION OF LOAD-BALANCING ALGORITHMS

Load-balancing algorithms fall into four major categories:

---

## **A. Static Algorithms**

Algorithms that **do NOT consider runtime load** or real-time server metrics.

### 1. **Round Robin**

### 2. **Weighted Round Robin**

### 3. **IP Hash**

### 4. **URL Hash / Request Hash**

---

## **B. Dynamic Algorithms**

Algorithms that use **real-time server load** or connection metrics.

### 5. **Least Connections**

### 6. **Weighted Least Connections**

### 7. **Least Response Time**

### 8. **Resource-Based / Utilization-Based (CPU/Memory/Latency)**

---

## **C. Consistent Hashing / Distributed Hashing Family**

### 9. **Consistent Hashing**

### 10. **Ring Hashing / Ketama Hashing**

### 11. **Maglev Hashing (Google)**

---

## **D. Advanced / Intelligent Algorithms**

### 12. **Random with Two Choices**

### 13. **Adaptive Load Balancing (Feedback-based)**

### 14. **Latency-Based Routing (AWS Route 53 style)**

### 15. **Geo-Based Routing (GeoDNS, CDN LB)**

---

# ✅ 2. ALGORITHMS WITH DEFINITIONS + WHEN TO USE + PITFALLS

---

## **1. Round Robin**

**How it works:**  
Each request goes to the next server in cyclic order.

**Use When:**

- All servers are identical.
    
- Traffic is uniform.
    
- Stateless applications.
    

**Avoid When:**

- Servers have different capacities.
    
- Connections are long-lived (WebSockets).
    

---

## **2. Weighted Round Robin**

**How it works:**  
Servers with higher weights get more requests.

**Use When:**

- Servers have different CPU/RAM capacities.
    
- Mixed hardware in clusters.
    

**Avoid When:**

- You need dynamic metrics (weights are static).
    

---

## **3. Least Connections**

**How it works:**  
Sends new requests to the server with the **least active connections**.

**Use When:**

- Variable request duration (heterogeneous load).
    
- Good for long-lived connections (HTTP/1.1 keep-alive).
    

**Avoid When:**

- Connections do not accurately reflect load (e.g., event-loop servers like Node.js).
    

---

## **4. Weighted Least Connections**

**How it works:**  
Same as least connections but considers server capacity.

**Use When:**

- Cluster contains both large and small nodes.
    
- Traffic is uneven.
    

---

## **5. Least Response Time**

**How it works:**  
Sends traffic to the server with the **fastest average response time** + **fewest connections**.

**Use When:**

- APIs where latency is the main concern.
    
- Real-time systems.
    

**Avoid When:**

- Backend metrics are unstable (noisy signals).
    

---

## **6. IP Hash**

**How it works:**  
`server = hash(client_ip) % number_of_servers`

**Use When:**

- Need **sticky session** (stateful sessions without Redis).
    
- Gaming servers, session-bound users.
    

**Avoid When:**

- Cluster resizing → hash changes and session breaks.
    

---

## **7. URL Hash / Request Hash**

**How it works:**  
`server = hash(url/path)`

**Use When:**

- Caching systems (CDN)
    
- Content-based routing
    

**Avoid When:**

- Low hit rates → low cache efficiency.
    

---

## **8. Consistent Hashing (Ring-Based)**

**How it works:**  
Maps servers and keys on a logical ring so minimal remapping happens when a server joins/leaves.

**Use When:**

- Need stable routing when scaling (add/remove node).
    
- Distributed systems like:
    
    - Redis Cluster
        
    - Cassandra
        
    - Kafka Partitions
        
    - Microservices needing sticky routing
        

**Avoid When:**

- Need full randomness across nodes.
    

---

## **9. Random with Two Choices (Power of 2 Choices)**

**How it works:**  
Pick two random servers → choose the one with fewer connections.

**Use When:**

- Very large clusters (hundreds/thousands of nodes).
    
- Distributed LB systems.
    

**Why it’s great:**  
Nearly matches performance of full least-connections with minimal overhead.

---

## **10. Adaptive / Resource-Based**

**How it works:**  
Routes traffic based on real-time metrics:

- CPU
    
- Memory
    
- Latency
    
- Queue length
    

**Use When:**

- Autoscaling environments
    
- Dynamic workloads
    

**Avoid When:**

- Metric collection overhead is high.
    

---

## **11. Latency-Based Routing**

**Examples:** AWS Route 53, Google Global Load Balancer.

**How it works:**  
Send user to the region/server with lowest latency.

**Use When:**

- Global systems
    
- CDNs
    
- Multi-region microservices
    

---

## **12. Geo-Based Routing**

**Use When:**

- Compliance (GDPR, region restrictions)
    
- Edge routing for low-latency
    

---

# ✅ 3. WHICH ALGORITHM TO USE WHEN (CHEAT SHEET)

|Scenario|Best Algorithm|Why|
|---|---|---|
|All nodes identical & stateless|Round Robin|Simple, efficient|
|Nodes with different capacity|Weighted Round Robin|Respects capacity|
|Long-lived connections|Least Connections|Tracks actual load|
|Highly variable traffic load|Least Response Time|Picks fastest server|
|Sticky sessions required|IP Hash / Consistent Hashing|Same user → same server|
|CDNs / caching|URL Hash|Improves cache hit ratios|
|Dynamic scaling (add/remove servers)|Consistent Hashing|Minimal rebalancing|
|Global users across regions|Latency-based routing|Lowest-latency edge|
|Extremely large clusters|Random with Two Choices|Efficient with minimal overhead|
|Real-time autoscaling|Resource-based|Cluster adapts to load|

---

# ✅ 4. INTERVIEW-ORIENTED QUESTIONS (With expected answers)

---

## **A. Conceptual Questions**

### **1. How does Round Robin differ from Least Connections?**

- RR blindly distributes requests.
    
- LC considers active connections to avoid overloading busy nodes.
    

---

### **2. Why is Consistent Hashing used in distributed systems?**

- It prevents massive reshuffling when servers are added/removed (only a small % of keys move).
    
- Critical for scalability and cache stability.
    

---

### **3. Why use Weighted algorithms?**

Because all nodes are not equal — different CPU, memory, or hardware.

---

## **B. Scenario-Based Questions**

---

### **4. Your system uses sticky sessions; which algorithm will you choose?**

- IP Hash (simple)
    
- Consistent Hashing (better scalability)
    

---

### **5. You have long-running requests (file uploads, streaming). Which algorithm fits?**

- Least Connections  
    Because number of open connections reflects true load.
    

---

### **6. You have a global system deployed in 5 regions; which LB algorithm to use?**

- Latency-based routing
    
- Geo-based DNS  
    Because it directs traffic to the nearest region.
    

---

### **7. If one server becomes slow but has fewer connections, what happens in Least Connections?**

It may still get traffic → **unfair load**.  
Solution: **Least Response Time** or **Health-based routing**.

---

## **C. Deep-Dive / Senior-Level Questions**

---

### **8. What’s the problem with IP Hash when scaling up or down?**

Adding/removing a server changes the modulus →  
All traffic gets remapped → sessions break → hot partitions.

Solution: **Consistent Hashing**.

---

### **9. How does “Power of Two Choices” outperform Round Robin?**

- It avoids overloaded servers more often.
    
- Achieves performance close to perfect LC with far less complexity.
    

---

### **10. How do CDNs use load-balancing internally?**

- Layer 1 → GeoDNS
    
- Layer 2 → Latency-based
    
- Layer 3 → URL Hash + Consistent Hashing for cache affinity
    
- Layer 4 → Weighted RR + LC inside datacenters
    

---

## **D. System-Design Questions**

---

### **11. If you have 1M requests/sec and 30 servers, which algorithm ensures smooth distribution?**

- Random with Two Choices
    
- Weighted Round Robin
    
- Nginx’s Least Time algorithm
    

---

### **12. How does an LB avoid hotspotting?**

- Distributes load using:
    
    - least-connections
        
    - consistent hashing
        
    - random+2
        
    - locality-aware hashing
        

---

# ✅ 5. SHORT SUMMARY (Interview-Friendly)

- Use **Round Robin/Weighted RR** → for simple, stateless distribution.
    
- Use **Least Connections/Least Response Time** → for long-lived or varying workloads.
    
- Use **IP Hash** → for sticky sessions.
    
- Use **Consistent Hashing** → for scalable, stable routing.
    
- Use **Latency/Geo-based routing** → for global traffic.
    
- Use **Random with Two Choices** → for massive clusters.
    

---

