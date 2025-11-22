Comprehensive list of the most commonly asked LLD (Low-Level Design)** questions for **Software Developer / SDE interviews** — especially in top tech companies like **Amazon, Google, Microsoft, Uber, Swiggy, Zomato, Flipkart, etc.**

I’ll organize them by **category and concept** — so you understand what pattern or design idea each question typically tests.

---
## 🧩 1. **Core Object-Oriented Design Questions**

These are LLD questions that test your ability to design classes, relationships (association, composition, inheritance), and use of OOP principles like encapsulation, abstraction, and polymorphism.

|Question|Concepts Tested|Common Design Patterns Used|
|---|---|---|
|Design a Parking Lot System|OOP, Class Hierarchy, State Management|Strategy, Singleton|
|Design a Library Management System|OOP, Class Diagram, Relationships|Factory, Observer|
|Design a Movie Ticket Booking System (BookMyShow)|Entities, Aggregation, Availability Check|Strategy, Singleton, Observer|
|Design an Online Food Ordering System (Zomato/Swiggy)|Entity Relationships, Order Flow|Strategy, Observer|
|Design a Hotel Management System|Room Reservation, Pricing, Availability|Strategy, Factory|
|Design an Airline Reservation System|Availability, Seat Allocation|Strategy, Singleton|
|Design a Car Rental System (Zoomcar)|Booking Flow, Inventory Management|Factory, Strategy|
|Design a Vending Machine|State Transitions, Coin Handling|State Pattern|
|Design an Elevator System|Scheduling, State Transitions|State, Strategy, Command|
|Design a Coffee Vending Machine|Workflow States|State, Factory|
|Design a Logging Framework|Thread Safety, Extensibility|Singleton, Strategy|
|Design an ATM System|State Transitions, Authentication|State, Strategy, Proxy|

---

## ⚙️ 2. **System Behavior / Interaction Design**

Focuses on **interactions between entities**, state management, and behavior modeling.

|Question|Concepts Tested|Common Design Patterns Used|
|---|---|---|
|Design a Notification System (Email, SMS, Push)|Asynchronous Communication|Observer, Strategy|
|Design a Pub-Sub System|Loose Coupling, Event Handling|Observer, Publisher-Subscriber|
|Design a Rate Limiter|Concurrency, Thread-Safety|Token Bucket, Leaky Bucket|
|Design a Cache System (LRU, LFU)|Data Structures, Eviction Policies|Singleton, Strategy|
|Design a Logger|Thread Safety, Configurability|Singleton, Chain of Responsibility|
|Design a Task Scheduler|Thread Scheduling, Priority|Command, Strategy|
|Design a Message Queue (like Kafka)|Producer-Consumer Model|Observer, Builder|

---

## 💬 3. **Design for Real-World Applications (Product-Based)**

These simulate designing real services or mini versions of popular products.

|Question|Concept Tested|Common Patterns|
|---|---|---|
|Design a TinyURL Service|Hashing, Mapping, Scalability|Singleton|
|Design a Social Media Feed (Instagram, Twitter)|Caching, Ordering|Observer, Strategy|
|Design a Chat Application (WhatsApp)|Message Delivery, Group Chat|Observer, Mediator|
|Design a News Feed System|Ranking, Fan-out on Write/Read|Observer, Strategy|
|Design an Online Payment Gateway|Transaction Workflow|Chain of Responsibility|
|Design an Online Quiz System|Question Bank, Evaluation|Strategy, Factory|
|Design a Ride-Sharing System (Uber/Ola)|Matching Algorithm|Strategy, Observer, Factory|
|Design a Meeting Scheduler|Conflict Resolution|Strategy|

---

## 🧠 4. **Design Pattern Focused Questions**

These questions test **recognition and application** of specific design patterns.

|Design Pattern|Example Question|
|---|---|
|Singleton|Design a Configuration Manager for an App|
|Factory|Design a Shape or Notification Factory|
|Strategy|Design Payment Methods or Sorting Algorithms|
|Observer|Design a Notification System or Event Framework|
|Decorator|Design a Pizza Ordering System with Add-ons|
|State|Design a Vending Machine or ATM|
|Command|Design an Undo/Redo System|
|Proxy|Design a Caching Proxy or Access Control System|
|Adapter|Design an API Integrator between Two Systems|
|Composite|Design a File System (Folders/Files)|
|Builder|Design a Query Builder or Object Creator|
|Chain of Responsibility|Design a Logging or Authentication Pipeline|

---

## 📦 5. **Concurrency and Thread-Safe Design Questions**

Used in high-level interviews to test system-level thinking.

|Question|Concepts Tested|
|---|---|
|Design a Thread-Safe Singleton|Double-checked locking, Volatile|
|Design a Thread Pool|Executor Pattern|
|Design a Blocking Queue|Concurrency Control|
|Design a Producer-Consumer System|Synchronization, Wait/Notify|
|Design a Rate Limiter|Token Bucket, Sliding Window|
|Design a Cache with Expiry|Synchronization, Eviction Policy|

---

## 🧰 6. **Utility / Infrastructure Design**

|Question|Concepts Tested|Common Design Patterns Used|
|---|---|---|
|Design a File System|Hierarchical Structure|Composite|
|Design a Key-Value Store|Indexing, Storage|Strategy|
|Design a URL Shortener|Mapping, Hashing|Singleton|
|Design a Metrics Collector|Aggregation, Observability|Observer|
|Design a Configuration Management System|Thread Safety, Immutability|Singleton, Builder|

---

## 🧩 7. **Coding + LLD Hybrid Questions**

These combine **data structure + design** expectations.

|Question|Focus|
|---|---|
|Implement an LRU Cache|Map + Doubly Linked List|
|Implement a Hit Counter|Queue / Sliding Window|
|Design a Stack using Queues|Data Structure + Abstraction|
|Implement a Thread-Safe Logger|Synchronization + Design|
|Design a Tic-Tac-Toe Game|OOP + Game State|

---

## 🧭 8. **Interview Tip — How to Approach LLD Questions**

When asked an LLD question:

1. **Clarify Requirements** – Functional and Non-functional.
    
2. **Identify Core Entities** – Classes, attributes, relationships.
    
3. **Draw Class Diagram** – Show hierarchy and interactions.
    
4. **Discuss Design Patterns** – Explain why you chose them.
    
5. **Handle Concurrency / Scale** – Optional but valuable.
    
6. **Write Clean Pseudocode / Class Skeletons** – If asked.
    
7. **Explain Trade-offs** – Extensibility, Maintainability, and SRP.
    

---

If you’d like, I can next give you a **structured table** of

> 🔹 “Top 20 LLD Interview Questions + The Design Pattern They Map To + The Core Problem They Solve”

**Structured master table** of the **Top 20 Most Frequently Asked LLD Questions** in software developer interviews, along with:

- 🎯 **Design Pattern(s)** used
    
- 🧩 **Problem Type / Concept Tested**
    
- 💡 **What Problem the Pattern Solves (in context)**
    

---

## 🧠 **Top 20 LLD Interview Questions — with Patterns & Problem Solved**

|#|LLD Question|Design Pattern(s) Used|Concept Tested|Problem Solved|
|---|---|---|---|---|
|**1**|🅿️ **Design a Parking Lot System**|Singleton, Strategy|OOP Design, State Handling|Manage multiple vehicle types, slots, and pricing using flexible policies.|
|**2**|🎬 **Design a Movie Ticket Booking System (BookMyShow)**|Observer, Strategy, Factory|Entity Modeling, Availability, Notifications|Handle dynamic seat booking, pricing, and event notifications.|
|**3**|🍔 **Design a Food Ordering System (Zomato/Swiggy)**|Strategy, Observer|Entity Relationship, Order Lifecycle|Handle restaurant menus, order tracking, and delivery updates.|
|**4**|🚕 **Design a Cab Booking System (Uber/Ola)**|Strategy, Observer, Factory|Matching Algorithm, Geo-based Assignment|Efficient driver-customer match and live tracking.|
|**5**|☕ **Design a Coffee Vending Machine**|State, Factory|State Transitions, Workflow|Manage machine states (Idle, Ready, Dispense, OutOfStock).|
|**6**|🏧 **Design an ATM System**|State, Strategy, Proxy|Authentication, State Management|Handle ATM states and user sessions securely.|
|**7**|🏢 **Design a Hotel Management System**|Factory, Strategy|Room Booking, Pricing Policies|Flexible room pricing and easy reservation flow.|
|**8**|🪙 **Design a Vending Machine**|State, Factory|Workflow, Error Handling|Manage item selection, payment, and dispense flow.|
|**9**|📚 **Design a Library Management System**|Factory, Observer|Inventory, Lending Flow|Handle book lending, due dates, and notifications.|
|**10**|🔗 **Design a URL Shortener (TinyURL)**|Singleton, Factory|Mapping, Caching|Efficient unique ID generation and mapping short → long URL.|
|**11**|💬 **Design a Chat Application (WhatsApp/Slack)**|Observer, Mediator|Real-time Communication|Efficient message delivery, group updates, and status tracking.|
|**12**|🪶 **Design a Logging Framework**|Singleton, Chain of Responsibility|Thread Safety, Extensibility|Enable log level filtering and multi-sink output.|
|**13**|⚙️ **Design a Notification System (Email/SMS/Push)**|Observer, Strategy|Loose Coupling, Extensibility|Easily add new notification channels dynamically.|
|**14**|🔁 **Design a Rate Limiter**|Token Bucket / Sliding Window (Strategy)|Concurrency Control|Prevent overload by controlling request rates.|
|**15**|🧱 **Design an Online Payment System**|Strategy, Chain of Responsibility|Workflow, Transaction Management|Handle multiple payment gateways with validation chains.|
|**16**|🧩 **Design an LRU Cache**|Singleton, Strategy|Memory Management|Efficient caching with least recently used eviction policy.|
|**17**|🪵 **Design a File System**|Composite|Hierarchical Structure|Represent files and folders uniformly.|
|**18**|🕹 **Design a Tic-Tac-Toe / Chess Game**|State, Strategy|Game Logic, Turn Handling|Manage turn-based state transitions and rules.|
|**19**|🧰 **Design a Thread Pool / Executor Service**|Command, Singleton|Concurrency, Resource Management|Efficient reuse of threads and queued task execution.|
|**20**|🏗 **Design a Builder Framework (Query Builder / Object Creator)**|Builder, Fluent Interface|Object Construction|Simplify creation of complex immutable objects.|

---

## 🔍 **Patterns & Their Real Use in These LLDs**

|Pattern|Used In|Problem Solved|
|---|---|---|
|**Singleton**|Logger, Cache, Config Manager, Parking Lot|Ensure only one global instance exists.|
|**Factory / Abstract Factory**|Booking Systems, Payment, Vending Machine|Create related objects without exposing concrete classes.|
|**Strategy**|Payment, Pricing, Matching Algorithms|Define a family of algorithms and make them interchangeable.|
|**Observer**|Notification, Feed, Chat|Automatically notify dependent objects about changes.|
|**State**|ATM, Vending Machine, Elevator|Manage object behavior when internal state changes.|
|**Builder**|Query Builder, Object Creator|Simplify building complex objects with multiple fields.|
|**Composite**|File System, UI Components|Treat individual and composite objects uniformly.|
|**Chain of Responsibility**|Logger, Authentication, Payment Workflow|Pass requests through a chain of handlers.|
|**Command**|Undo/Redo System, Thread Pool|Encapsulate actions as objects to allow undo/redo or queueing.|
|**Proxy**|Caching Proxy, Access Control|Control access or add caching around an object.|

---

## 💬 Interviewer’s Evaluation Criteria

|Skill Area|What They Evaluate|
|---|---|
|**OOP Fundamentals**|Class design, inheritance, abstraction, encapsulation|
|**Pattern Recognition**|Choosing the right pattern & explaining why|
|**Scalability / Extensibility**|How easily system can adapt to new features|
|**Trade-offs & Reasoning**|Explain design choices (e.g., Singleton vs Factory)|
|**Communication & Diagramming**|How clearly you explain relationships & data flow|

---

## ⚡ Bonus Tip

> If you’re aiming for **FAANG / Product-based companies**, focus on these **5 LLDs first:**
> 
> 1. Parking Lot
>     
> 2. Vending Machine
>     
> 3. BookMyShow / Food Ordering
>     
> 4. Notification System
>     
> 5. Logger / Rate Limiter
>     

Each touches on **core design patterns + concurrency + extensibility**, which interviewers love.

---
