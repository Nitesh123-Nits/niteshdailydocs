

---

## 🧭 1. Introduction

Modern distributed systems rely on **intermediate network components** to manage scalability, availability, routing, and security.  
Three of the most commonly discussed components are:

- **Load Balancer** – Distributes incoming traffic across multiple servers.
    
- **Reverse Proxy** – Sits in front of servers and routes requests on their behalf.
    
- **API Gateway** – Acts as an entry point for microservices, handling API-specific concerns.
    

Though they may seem similar, their **goals and scopes** differ fundamentally.

---

## ⚙️ 2. Load Balancer

### **Definition**

A **Load Balancer** evenly distributes client requests across multiple backend servers to ensure **availability, reliability, and scalability**.

### **Primary Goal**

➡️ Optimize **resource utilization** and **minimize latency** by balancing the incoming load.

### **How it works**

- Client sends a request to a single endpoint (the Load Balancer).
    
- Load Balancer selects a backend server using an **algorithm** (like Round Robin, Least Connections, IP Hash, etc.).
    
- It forwards the request to that server and relays the response back to the client.
    

### **Common Algorithms**

- **Round Robin:** Requests distributed sequentially.
    
- **Least Connections:** Sends to the server with the fewest connections.
    
- **Weighted Round Robin / Least Connection:** Prioritize more powerful servers.
    
- **Consistent Hashing:** Sticky sessions.
    

### **Examples**

- **Hardware:** F5, Citrix NetScaler
    
- **Software / Cloud:** Nginx, HAProxy, AWS Elastic Load Balancer (ALB/NLB), Google Cloud LB, Azure Load Balancer
    

### **Key Features**

- Health checks for servers
    
- Failover and redundancy
    
- SSL termination (optional)
    
- Layer 4 (TCP) and Layer 7 (HTTP) load balancing
    

### **Use Case Example**

E-commerce website with millions of users → distribute traffic to multiple web servers running behind a load balancer.

---

## 🧩 3. Reverse Proxy

### **Definition**

A **Reverse Proxy** sits in front of one or more backend servers and forwards client requests to them.  
Unlike a **forward proxy** (which acts on behalf of clients), a reverse proxy acts **on behalf of servers**.

### **Primary Goal**

➡️ Provide an **abstraction and control layer** for backend servers (security, caching, SSL termination, routing).

### **How it works**

- Client thinks it’s talking to the main server (the reverse proxy).
    
- The proxy decides **which backend** to route to and **forwards** the request.
    
- The backend’s response is sent back through the proxy to the client.
    

### **Common Capabilities**

- SSL termination / HTTPS offloading
    
- Static content caching
    
- Gzip compression
    
- Authentication and access control
    
- Request routing and rewriting
    
- Hiding backend details (server IP, topology)
    

### **Examples**

- Nginx
    
- Apache HTTP Server (mod_proxy)
    
- HAProxy
    
- Traefik
    

### **Use Case Example**

CDN edge servers (like Cloudflare) act as reverse proxies → cache and protect backend origin servers.

---

## 🌐 4. API Gateway

### **Definition**

An **API Gateway** is a **specialized reverse proxy** designed for **microservices architectures**.  
It’s the **entry point** for all API calls from clients.

### **Primary Goal**

➡️ Provide **API-level management**, such as request routing, authentication, rate limiting, monitoring, and service aggregation.

### **How it works**

- Clients make calls to the API Gateway.
    
- The gateway **authenticates, authorizes**, applies **policies (e.g. rate limits)**, and **routes requests** to the right microservice.
    
- It may **aggregate** multiple backend calls into one response.
    

### **Key Features**

- Authentication & Authorization (JWT, OAuth2)
    
- Rate limiting & throttling
    
- Caching responses
    
- Circuit breaker / retry patterns
    
- Request/response transformation (REST ↔ gRPC ↔ SOAP, etc.)
    
- Logging and metrics
    
- Versioning and service discovery integration
    

### **Examples**

- **Open Source:** Kong, Tyk, KrakenD, Ambassador
    
- **Cloud Managed:** AWS API Gateway, Azure API Management, Apigee, Spring Cloud Gateway
    

### **Use Case Example**

Microservices-based application (e.g., Netflix, Uber, Amazon) where each request must be authenticated, routed, and monitored.

---

## 🧮 5. Comparison Table

|Feature / Aspect|**Load Balancer**|**Reverse Proxy**|**API Gateway**|
|---|---|---|---|
|**Primary Role**|Distribute traffic across servers|Forward client requests to internal servers|Manage and route API calls in microservice systems|
|**Focus Layer**|L4 / L7 (Network + HTTP)|L7 (HTTP)|L7 (HTTP / REST / gRPC / GraphQL)|
|**Scope**|Performance, scalability|Security, caching, routing|Authentication, orchestration, policies|
|**Traffic Source**|End-user clients|End-user clients|API consumers / mobile / web apps|
|**Awareness of APIs**|No|Minimal|Deep (API structure, versioning, protocols)|
|**Security Features**|Basic (SSL termination)|SSL, access control|Auth, throttling, WAF, token verification|
|**Request Transformation**|No|Basic (URL rewrite)|Yes (JSON/XML ↔ transformation)|
|**Service Discovery Integration**|Sometimes|Rarely|Often (via registry or config)|
|**Used in**|Monolithic or distributed apps|Web servers / edge servers|Microservices-based systems|

---

## 🧠 6. Real-World Example (Netflix-Style System)

|Component|Purpose|
|---|---|
|**AWS Elastic Load Balancer (ELB)**|Balances traffic across multiple instances of the API Gateway|
|**Nginx Reverse Proxy**|Handles SSL termination and caching|
|**Netflix Zuul / Spring Cloud Gateway**|Routes requests to microservices, handles auth, rate limits, logging|

This layered approach ensures **scalability (Load Balancer)**, **security and caching (Reverse Proxy)**, and **API-level orchestration (Gateway)**.

---

## 🧩 7. Typical Architecture Diagram (Conceptually)

```
         ┌────────────┐
         │   Client   │
         └──────┬─────┘
                │
        ┌───────▼────────┐
        │ Load Balancer  │
        └───────┬────────┘
                │
       ┌────────▼────────┐
       │ Reverse Proxy / │
       │   API Gateway   │
       └────────┬────────┘
                │
   ┌────────────┼─────────────┐
   │    Service A (Auth)      │
   │    Service B (Orders)    │
   │    Service C (Payments)  │
   └──────────────────────────┘
```

---

## 💬 8. Common Interview Questions

### ✅ **Basic**

1. What is the difference between a Load Balancer and a Reverse Proxy?
    
2. Is a Load Balancer a type of Reverse Proxy?  
    → Yes, at Layer 7, many load balancers act as reverse proxies.
    
3. Why use a Reverse Proxy in front of your servers?
    

### ✅ **Intermediate**

4. How does an API Gateway differ from a Reverse Proxy?
    
5. Can a Load Balancer and API Gateway coexist?  
    → Yes. Load balancer distributes traffic to multiple gateway instances.
    
6. What are some patterns handled by API Gateways (e.g., aggregation, routing, fallback)?
    

### ✅ **Advanced**

7. What are the performance trade-offs of adding an API Gateway layer?
    
8. How to make an API Gateway fault-tolerant (circuit breaker, retries, fallback)?
    
9. How does SSL termination differ at LB vs Reverse Proxy vs Gateway?
    
10. What’s the difference between L4 and L7 load balancing?
    

---

## 🧱 9. Key Takeaways

- **Load Balancer** = distributes **traffic** for scalability.
    
- **Reverse Proxy** = protects and controls **servers**.
    
- **API Gateway** = manages and orchestrates **APIs**.
    
- **Hierarchy:**
    
    ```
    API Gateway ⊂ Reverse Proxy ⊂ Load Balancer
    ```
    
    (Each builds on the capabilities of the previous one.)
    

---

[[Load Balancer]]