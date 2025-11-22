![[Pasted image 20251122171427.png]]When choosing a **shard key** (in a sharded database like MongoDB, Cassandra, or any distributed data system), these **three principles** are the most critical. Let’s break them down clearly:

---

### 🔑 1. **High Cardinality**

**Meaning:**

- The shard key should have **many unique possible values**.
    
- This ensures that data can be distributed across all shards rather than clustering around a few.
    

**Why it matters:**

- Low-cardinality fields (like “country” or “status”) cause **data skew** — one shard gets most of the writes, leading to performance bottlenecks.
    

**Example:**

|Field|Cardinality|Good Shard Key?|Why|
|---|---|---|---|
|`gender`|Low (2 values)|❌ No|All data goes to only 2 shards|
|`userId`|High (millions of users)|✅ Yes|Even spread across shards|

---

### ⚖️ 2. **Even Data Distribution**

**Meaning:**

- The shard key should spread data **uniformly across shards** to prevent **hotspots** (shards getting uneven traffic).
    

**How to achieve:**

- Use a **hashed** shard key if values are sequential (e.g., MongoDB’s `hashed(userId)`).
    
- Combine multiple fields (compound shard key) like `region + userId` if you need some locality but still want good distribution.
    

**Example:**

- `timestamp` → ❌ bad (monotonic, all new writes go to the latest shard)
    
- `hashed(userId)` → ✅ good (randomized, balanced write load)
    

---

### 🧭 3. **Align With Query Patterns**

**Meaning:**

- The shard key must **match your most common query filters** to enable **targeted queries** (querying one shard instead of all).
    

**Why it matters:**

- If your shard key doesn’t align with query filters, every query becomes a **scatter-gather** — sent to all shards → poor performance.
    

**Example:**  
If your common queries are:

```js
db.orders.find({ userId: 123 })
```

Then `userId` (or `hashed(userId)`) is a perfect shard key.

If queries are like:

```js
db.orders.find({ region: 'US', orderDate: { $gt: ... } })
```

Then a **compound shard key** like `{ region, orderDate }` might be ideal.

---

### 🧩 Summary: Ideal Shard Key Checklist

|Principle|Description|Example|
|---|---|---|
|**High Cardinality**|Many unique values|`userId`, `email`, `UUID`|
|**Even Distribution**|Avoid hotspots or uneven load|`hashed(userId)`, `region + userId`|
|**Aligned with Queries**|Matches most frequent filters|If queries often use `userId`, use it as shard key|

---

### 🚫 Common Mistakes

- Sharding on **timestamp** → causes all writes to newest shard (hotspot).
    
- Sharding on **low-cardinality** field → uneven data split.
    
- Shard key not used in queries → scatter-gather overhead.
    

---

Would you like me to show a **few practical shard key examples** for systems like MongoDB, Cassandra, and Elasticsearch side by side?  
That way you can see how these principles apply in real design scenarios.

## Question
Let’s go through **real-world shard key design examples** across three popular distributed databases — **MongoDB**, **Cassandra**, and **Elasticsearch** — to see how _high cardinality_, _even distribution_, and _query alignment_ work together.

---

## 🧱 1. **MongoDB**

### ✅ Good Example: `hashed(userId)`

**Use Case:** Large multi-tenant app (e.g., e-commerce, social media).

|Aspect|Explanation|
|---|---|
|**High Cardinality**|Every userId is unique — millions of possible values.|
|**Even Distribution**|`hashed` key randomizes userIds → avoids hotspots.|
|**Aligns with Queries**|Most queries filter by userId (e.g., `find({ userId: 123 })`).|

**Result:**  
✔ Balanced writes and queries target only one shard.  
⚠ Downside — you lose ordering by userId (hash breaks order).

---

### ⚠ Bad Example: `createdAt` (timestamp)

|Aspect|Issue|
|---|---|
|**Cardinality**|High — timestamps are unique.|
|**Distribution**|❌ Bad — writes always go to the most recent shard.|
|**Query Alignment**|❌ Often causes hotspot writes.|

**Result:**  
Uneven write load, one shard overloaded (the latest).

---

### ✅ Alternative: Compound Shard Key `{ region, hashed(userId) }`

|Aspect|Benefit|
|---|---|
|**High Cardinality**|Region + hashed userId gives large unique key space.|
|**Even Distribution**|Hashing ensures balance even within each region.|
|**Query Alignment**|Matches queries filtering by region and user.|

**Result:**  
Perfect for region-based apps like food delivery or social platforms.

---

## 🪴 2. **Cassandra**

Cassandra partitions data based on the **partition key** (acts like shard key).

### ✅ Good Example: `(user_id)`

**Use Case:** Accessing user data or timeline.

|Aspect|Explanation|
|---|---|
|**High Cardinality**|Every user_id is unique.|
|**Even Distribution**|Partition key is hashed internally → auto-balanced.|
|**Query Alignment**|Queries like `SELECT * FROM posts WHERE user_id = ?` hit one partition.|

**Result:**  
Fast lookups, evenly distributed partitions.

---

### ⚠ Bad Example: `(country)`

|Aspect|Issue|
|---|---|
|**Cardinality**|Low — few countries.|
|**Distribution**|❌ Skewed — US/India shards get overloaded.|
|**Query Alignment**|Queries not selective — read many partitions.|

**Result:**  
Hot partitions and unbalanced storage.

---

### ✅ Better: Compound Partition Key `(country, user_id)`

|Aspect|Benefit|
|---|---|
|**High Cardinality**|Combination increases uniqueness.|
|**Even Distribution**|User_id spreads data inside each country.|
|**Query Alignment**|Enables country-specific queries too.|

---

## 🔍 3. **Elasticsearch**

Elasticsearch distributes data using **index shards** and routing keys.

### ✅ Good Example: `userId` (as routing key)

**Use Case:** Querying logs or posts per user.

|Aspect|Explanation|
|---|---|
|**High Cardinality**|Many users → many routing values.|
|**Even Distribution**|User IDs distributed evenly across shards.|
|**Query Alignment**|You can route queries directly to that user’s shard (`routing=userId`).|

**Result:**  
Low latency, reduced cluster load.

---

### ⚠ Bad Example: `region`

|Aspect|Issue|
|---|---|
|**Cardinality**|Low.|
|**Distribution**|Some shards overloaded with popular regions.|
|**Query Alignment**|If your queries aren’t by region, routing fails.|

---

## ⚙️ Comparison Table

|Database|Concept|Good Example|Bad Example|Key Balancing Method|
|---|---|---|---|---|
|**MongoDB**|Shard Key|`hashed(userId)`|`createdAt`|Hash-based sharding|
|**Cassandra**|Partition Key|`(user_id)`|`(country)`|Murmur3 hashing|
|**Elasticsearch**|Routing Key|`userId`|`region`|Routing hash function|

---

### 🧭 General Rule of Thumb

|Goal|Guideline|
|---|---|
|**Avoid hotspots**|Don’t use monotonically increasing keys (timestamps, IDs).|
|**Keep queries targeted**|Choose shard key that appears in your most common query filter.|
|**Enable scalability**|Prefer high-cardinality fields or compound keys.|
|**Balance writes and reads**|Use hashing when ordering isn’t important.|

---
## Question

**Different types of sharding** is fundamental for system design interviews and real-world architecture.

Let’s break this down **clearly, deeply, and visually structured** 👇

---

## 🧩 **What Is Sharding?**

**Sharding** = breaking a large dataset into smaller, faster, more manageable parts called **shards**, which are distributed across multiple servers (nodes).

It’s mainly used to:

- Scale horizontally
    
- Improve performance and availability
    
- Distribute load and storage
    

---

## 🧱 **Types of Sharding**

There are **four major types** of sharding you should know:

1. **Range-based Sharding**
    
2. **Hash-based Sharding**
    
3. **Directory / Lookup-based Sharding**
    
4. **Composite / Hybrid Sharding**
    

Let’s explore each.

---

### 1️⃣ **Range-Based Sharding**

**How it works:**  
Data is divided into shards based on value **ranges** of the shard key.

**Example:**

|Range|Shard|
|---|---|
|`userId 1–1,000,000`|Shard A|
|`userId 1,000,001–2,000,000`|Shard B|
|`userId 2,000,001–3,000,000`|Shard C|

**✅ Pros**

- Simple to implement and understand
    
- Good for **range queries** (e.g., date ranges, sequential IDs)
    
- Easy to predict where data lives
    

**❌ Cons**

- **Hotspot risk:** all new data (e.g., new users or latest timestamps) go to one shard
    
- Rebalancing can be costly if ranges are uneven
    

**Used in:**

- MySQL (manual range sharding)
    
- MongoDB (range shard keys)
    
- Elasticsearch (time-based indices)
    

---

### 2️⃣ **Hash-Based Sharding**

**How it works:**  
Apply a **hash function** to the shard key, then assign the data to shards based on the hash output.

**Example:**  
`hash(userId) % num_shards`

|userId|Hash|Shard|
|---|---|---|
|101|1|Shard A|
|102|2|Shard B|
|103|0|Shard C|

**✅ Pros**

- **Even data distribution** (reduces hotspots)
    
- Great for **random access patterns**
    
- Ideal when queries are key-based (e.g., by userId, productId)
    

**❌ Cons**

- **No range queries** (hash breaks order)
    
- Harder to rebalance if shard count changes (consistent hashing helps)
    

**Used in:**

- MongoDB (`hashed` shard key)
    
- Cassandra (partition key hashing)
    
- DynamoDB (partition hash key)
    

---

### 3️⃣ **Directory-Based (Lookup-Based) Sharding**

**How it works:**  
A **mapping service (directory)** stores which shard each key belongs to.

**Example:**

|userId|Shard|
|---|---|
|101|Shard A|
|102|Shard B|
|103|Shard C|

When a query arrives:

- Directory lookup → find correct shard → forward the query.
    

**✅ Pros**

- Very **flexible** — any sharding rule possible
    
- Easy to **reshard or rebalance** dynamically
    
- Can mix rules (region, user type, etc.)
    

**❌ Cons**

- **Directory is a single point of failure** (unless replicated)
    
- Extra lookup step adds latency
    
- Directory must stay consistent
    

**Used in:**

- Facebook’s TAO (social graph store)
    
- Custom enterprise sharding routers
    

---

### 4️⃣ **Composite / Hybrid Sharding**

**How it works:**  
Combines **multiple sharding strategies** — for example:

- **Hash + Range**
    
- **Range + Directory**
    
- **Region + Hash**
    

**Example:**  
Shard first by `region`, then within region by `hashed(userId)`:

|Region|Hash(userId)|Shard|
|---|---|---|
|US|0–4|Shard A|
|US|5–9|Shard B|
|IN|0–4|Shard C|
|IN|5–9|Shard D|

**✅ Pros**

- Very **flexible** and **balanced**
    
- Supports both **regional locality** and **load distribution**
    
- Minimizes cross-region latency
    

**❌ Cons**

- More complex to design and maintain
    
- Rebalancing logic gets complicated
    

**Used in:**

- Global-scale systems (Netflix, Uber, Amazon)
    
- MongoDB compound shard keys
    

---

## ⚙️ **Comparison Table**

|Type|How It Works|Pros|Cons|Use Case|
|---|---|---|---|---|
|**Range-Based**|Split by value ranges|Great for range queries|Hotspots on high/low ends|Time-series, ordered data|
|**Hash-Based**|Hash of key → shard|Even distribution|No range queries|User-based systems|
|**Directory-Based**|Lookup table maps keys to shards|Highly flexible|Directory overhead|Dynamic routing systems|
|**Composite (Hybrid)**|Combine strategies|Balanced + flexible|Complex logic|Multi-region, large-scale apps|

---

## 🧠 Bonus: **Consistent Hashing**

A **variant of hash sharding** used in systems like **Cassandra**, **DynamoDB**, **Kafka**, and **Redis Cluster**.

**Idea:**  
Instead of `hash(key) % N`, use a **hash ring** — when adding/removing a shard, only a small subset of keys move.

**✅ Benefit:**  
Highly scalable and avoids full data reshuffling.

---

## 🧭 When to Use Which

|Scenario|Best Sharding Type|
|---|---|
|Time-series or logs|Range-based|
|User-based systems|Hash-based|
|Complex routing or dynamic workloads|Directory-based|
|Global apps (e.g., region + user)|Composite/Hybrid|
|Streaming systems (Kafka, Redis)|Consistent hashing|

---
