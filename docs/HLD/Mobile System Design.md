Excellent and _very insightful_ question — this is exactly the kind of thinking that separates a **backend/system design** candidate from a **full-system or mobile-oriented system designer**.

You’re absolutely right — most system design interviews focus on **backend architecture** (APIs, databases, scaling, caching, etc.), but in **real-world systems**, we almost always serve **multiple clients**:

- **Web app** (React, Angular, etc.)
    
- **Mobile app** (Android/iOS)
    
- Sometimes even **IoT**, **TV**, or **wearable devices**.
    

So let’s create a **complete, detailed guide** that focuses on **system design considerations for systems serving both web and mobile clients**, with special emphasis on **mobile application perspectives** (Android, iOS).

---

## 🧭 OVERVIEW: SYSTEM DESIGN FOR WEB + MOBILE APPLICATIONS

When a system has both web and mobile clients, your design now needs to consider:

1. **Network conditions** (mobile ≠ stable Wi-Fi)
    
2. **Data synchronization** (offline-first behavior)
    
3. **API design and efficiency** (mobile bandwidth + battery)
    
4. **Push notifications and background jobs**
    
5. **Versioning and backward compatibility**
    
6. **Authentication and session management**
    
7. **App updates and release cycles**
    
8. **App Store & Play Store ecosystem constraints**
    

---

## 🧩 1. Client Differences and Their Impact on Design

|Concern|Web App|Mobile App|Design Impact|
|---|---|---|---|
|**Network Stability**|Usually stable (Wi-Fi)|Unstable, variable (3G/4G/5G)|APIs must support retries, incremental sync, background sync|
|**Storage**|Browser local storage|Local DB (Room, Realm, SQLite)|Support local caching/offline queue|
|**Processing Power**|Usually higher|Limited|Reduce client logic; use lightweight responses|
|**Deployment**|Continuous deployment possible|Needs app store release|Backward-compatible API versions|
|**Notifications**|Browser notifications (optional)|Push via FCM/APNs|Add event-driven notification backend|

---

## 🛠️ 2. Key Backend Design Considerations for Web + Mobile Systems

### (a) **API Gateway Layer**

- Acts as a single entry point for all clients.
    
- Can differentiate between **mobile clients** and **web clients** via headers or tokens.
    
- Supports:
    
    - API versioning (v1/mobile, v2/web)
        
    - Rate limiting per client type
        
    - Authentication delegation (JWT, OAuth2)
        

📘 **Interview Question Example:**

> How would you design an API gateway that serves both mobile and web apps efficiently?

---

### (b) **Data Synchronization and Offline Support**

- Mobile apps often work offline — so you need:
    
    - **Sync queue / job service**
        
    - **Conflict resolution logic** (e.g., last-write-wins, CRDTs)
        
    - **Delta updates** (send only changed data)
        
- Backend might maintain **sync timestamps** or **version numbers** per resource.
    

📘 **Interview Question Example:**

> How do you design a note-taking app that syncs offline data from mobile when the user reconnects?

---

### (c) **API Efficiency**

- Mobile devices are bandwidth and battery-sensitive.  
    Design APIs with:
    
- **Pagination** and **partial responses** (GraphQL or REST with query parameters)
    
- **Compression** (gzip, Brotli)
    
- **Batch endpoints** (send multiple requests together)
    
- **Content negotiation** (e.g., send light JSON or binary protobuf)
    

📘 **Interview Question Example:**

> How would you optimize APIs for low-bandwidth mobile networks?

---

### (d) **Push Notifications & Background Updates**

- Use **Firebase Cloud Messaging (FCM)** or **Apple Push Notification Service (APNs)**.
    
- Backend should integrate an **event-driven system** (Kafka, SNS/SQS, or Pub/Sub) to trigger push messages.
    

📘 **Interview Question Example:**

> How would you design a notification system for a chat app that supports both web and mobile?

---

### (e) **Authentication and Session Management**

- For mobile:
    
    - Use **JWT tokens** stored securely (EncryptedSharedPrefs, Keychain)
        
    - Refresh tokens periodically
        
    - Support **social login** (Google, Apple)
        
- Backend must handle:
    
    - Token revocation
        
    - Device management (e.g., logout from one device)
        

📘 **Interview Question Example:**

> How would you design secure authentication for mobile users with token-based auth?

---

### (f) **Versioning and Backward Compatibility**

- Web apps can be updated easily; mobile apps cannot.
    
- Maintain multiple API versions.
    
- Include `X-App-Version` header for adaptive responses.
    

📘 **Interview Question Example:**

> How would you handle backward compatibility when new API features break older mobile clients?

---

### (g) **Monitoring and Crash Reporting**

- Backend should log per-device performance metrics.
    
- Integrate with tools like:
    
    - Firebase Crashlytics (for app crashes)
        
    - APM (Application Performance Monitoring)
        
    - Sentry, Datadog
        

📘 **Interview Question Example:**

> How would you design monitoring for user errors and crashes across web and mobile?

---

### (h) **Feature Flagging and Remote Config**

- Mobile deployments take time — you can’t roll back easily.
    
- Use **remote feature flags** (Firebase Remote Config, LaunchDarkly) for feature control.
    

📘 **Interview Question Example:**

> How would you design a remote configuration system for your mobile app?

---

### (i) **Security and API Abuse Protection**

- Mobile apps are easier to reverse-engineer.
    
- Protect APIs via:
    
    - HMAC-signed requests
        
    - Device attestation (e.g., Play Integrity API, App Attest)
        
    - Encrypted communication (TLS pinning)
        

📘 **Interview Question Example:**

> How would you prevent mobile clients from being reverse-engineered to abuse your APIs?

---

## ⚙️ 3. Architecture Patterns for Multi-Platform Systems

|Pattern|Use Case|Example|
|---|---|---|
|**BFF (Backend for Frontend)**|Different client needs different API responses|Separate `/bff-web` and `/bff-mobile` services|
|**GraphQL Gateway**|Clients can request custom fields|Mobile asks for light data, web for detailed|
|**Event-Driven Systems**|Notifications, async updates|Chat, order tracking|
|**CQRS + Offline Cache**|Real-time sync with offline writes|Notes, task management apps|

📘 **Interview Question Example:**

> How would you design a BFF architecture when mobile and web clients have different API needs?

---

## 🧪 4. Common Mobile-Oriented System Design Questions

|Category|Example Question|
|---|---|
|**Offline Sync**|Design Google Docs / Evernote / WhatsApp sync|
|**Notifications**|Design push notification system (e.g., WhatsApp)|
|**Media Uploads**|Design image/video upload for Instagram|
|**Chat System**|Design WhatsApp / Messenger backend|
|**Real-time Updates**|Design real-time stock price feed or ride tracking|
|**App Configuration**|Design remote config system (like Firebase Remote Config)|
|**App Updates**|Handle breaking changes in old mobile versions|
|**Analytics**|Design event-tracking pipeline for mobile apps|
|**Battery/Network Efficiency**|Optimize data fetching and caching patterns|

---

## 🧭 5. Key Mobile-Specific System Design Concepts to Learn

1. **Offline-first architecture**
    
2. **Delta sync and conflict resolution**
    
3. **BFF pattern for platform-specific optimization**
    
4. **Push notification architecture**
    
5. **App version and feature rollout management**
    
6. **Token-based auth and device identity**
    
7. **GraphQL vs REST in mobile APIs**
    
8. **App telemetry and logging systems**
    
9. **Mobile CDN optimization (image resizing, caching)**
    
10. **Security (TLS pinning, obfuscation, attestation)**
    

---

## 📚 6. Resources for Practice

- **System Design for Mobile Developers – Alex Xu (ByteByteGo Mobile Edition)**
    
- **Firebase Architecture Guides**
    
- **AWS Reference Architecture: Mobile Backend**
    
- **Google Developers: Offline and Sync Strategies**
    
- **HighScalability blog – WhatsApp, Instagram, Messenger design articles**
    

---

## ✅ Summary Table — Additional Areas to Study for Mobile-Integrated System Design

|Area|Why Important|Learn / Focus|
|---|---|---|
|Offline-first|Mobile often goes offline|Conflict resolution, sync queues|
|BFF pattern|Different data needs|Separate backend layers|
|API efficiency|Mobile = low bandwidth|Pagination, compression|
|Notifications|User engagement|FCM, APNs|
|Versioning|Mobile updates slow|API compatibility|
|Security|Reverse engineering risk|HMAC, integrity check|
|Analytics|Track app behavior|Event batching, big data pipelines|
|Release management|App store delays|Remote feature flags|

---
