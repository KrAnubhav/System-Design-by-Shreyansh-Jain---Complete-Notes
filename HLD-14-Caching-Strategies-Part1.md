# HLD-14: Caching Strategies (Part 1)

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What is Caching](#what-is-caching)
3. [Types of Caching](#types-of-caching)
4. [Distributed Caching](#distributed-caching)
5. [Caching Strategies](#caching-strategies)
6. [Summary](#summary)
7. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Caching Strategies (Part 1)

**Coverage:**
- ✅ What is caching & benefits
- ✅ Types of caching (layers)
- ✅ Distributed caching
- ✅ 5 Caching strategies with sequence diagrams
  - Cache-Aside
  - Read-Through
  - Write-Around
  - Write-Through
  - Write-Back (Write-Behind)

**Part 2 will cover:**
- Cache eviction policies

---

## What is Caching

### Definition

```
Caching = Technique to store frequently used data
          in fast-access memory
          instead of slow-access memory
```

**Memory Speed Hierarchy:**

```
Fast Access Memory:
├─ CPU Cache (L1, L2, L3)
├─ RAM
└─ In-memory stores (Redis, Memcached)

Slow Access Memory:
├─ Hard Disk (HDD)
├─ SSD
└─ Database
```

**Example:**

```
WITHOUT Cache:
Request → App → Database (100ms)
Total: 100ms

WITH Cache:
Request → App → Cache (1ms)
Total: 1ms

100x faster! ✓
```

---

### Benefits

#### 1. Makes System Fast

```
Reduced Latency:
- Cache read: 1-5ms
- DB read: 50-100ms

Result: Faster response times
```

#### 2. Fault Tolerance

```
Scenario: DB goes down

WITHOUT Cache:
All requests fail ✗

WITH Cache (Write-Back strategy):
Requests served from cache ✓
System remains available
```

**Note:** Fault tolerance depends on caching strategy (covered in Write-Back section)

---

## Types of Caching

### Caching at Different Layers

```
┌─────────────────────────────────────┐
│         Client Side                 │
│  ┌──────────────────────────────┐   │
│  │   Browser Cache              │   │
│  │   - HTML, CSS, JS, Images    │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│         CDN Layer                   │
│  ┌──────────────────────────────┐   │
│  │   CDN Cache                  │   │
│  │   - Static content           │   │
│  │   - Geo-distributed          │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│      Load Balancer Layer            │
│  ┌──────────────────────────────┐   │
│  │   Load Balancer Cache        │   │
│  │   - L7 load balancers        │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│      Application Layer              │
│  ┌──────────────────────────────┐   │
│  │   Server-Side Cache          │   │
│  │   - Redis, Memcached         │   │
│  │   - Application cache        │   │ ← Focus of this lecture
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│       Database Layer                │
│  ┌──────────────────────────────┐   │
│  │   Database Cache             │   │
│  │   - Query cache              │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

---

### Server-Side Application Cache

**Architecture:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│    Load     │
│  Balancer   │
└─────────────┘
    │
    ├──────────┬──────────┬──────────┐
    ▼          ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ App    │ │ App    │ │ App    │ │ App    │
│Server 1│ │Server 2│ │Server 3│ │Server N│
└────────┘ └────────┘ └────────┘ └────────┘
    │          │          │          │
    └──────────┴──────────┴──────────┘
               │
               ▼
         ┌─────────┐
         │  Cache  │ ← Redis, Memcached
         │ (Redis) │
         └─────────┘
               │
               ▼
         ┌─────────┐
         │   DB    │
         └─────────┘
```

**Flow:**

```
1. Client → Load Balancer
2. Load Balancer → App Server
3. App Server → Cache (check first)
4. If cache miss → DB
5. Update cache
6. Return to client
```

---

## Distributed Caching

### Problem: Single Cache Server

```
┌────────┐  ┌────────┐  ┌────────┐
│ App 1  │  │ App 2  │  │ App 3  │
└────────┘  └────────┘  └────────┘
    │           │           │
    └───────────┼───────────┘
                ▼
          ┌─────────┐
          │  Cache  │ ← Single point of failure!
          │ Server  │
          └─────────┘

Problems:
❌ Limited scalability
❌ Single point of failure
❌ Limited resources
```

---

### Solution: Distributed Cache

```
┌────────┐  ┌────────┐  ┌────────┐
│ App 1  │  │ App 2  │  │ App 3  │
│Cache   │  │Cache   │  │Cache   │
│Client  │  │Client  │  │Client  │
└────────┘  └────────┘  └────────┘
    │           │           │
    └───────────┼───────────┘
                ▼
        ┌───────────────┐
        │  Cache Pool   │
        └───────────────┘
                │
    ┌───────────┼───────────┬───────────┐
    ▼           ▼           ▼           ▼
┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐
│Cache 1 │  │Cache 2 │  │Cache 3 │  │Cache N │
└────────┘  └────────┘  └────────┘  └────────┘

Benefits:
✓ Scalable (add more servers)
✓ No single point of failure
✓ Distributed load
```

---

### Consistent Hashing

**How Cache Server is Selected:**

```
Consistent Hashing Ring:

                    Cache 2
                       ●
                   ╱       ╲
              ╱               ╲
         ╱                       ╲
    ●                               ●
Cache 1                         Cache 3
    │                               │
    │                               │
    │       Request (hash)          │
    │            ●                  │
    │            │                  │
    │            └─► Clockwise      │
    │               rotation        │
    │               First server    │
    │               = Cache 3       │
    ●                               ●
Cache 5                         Cache 4
         ╲                       ╱
              ╲               ╱
                   ╲       ╱
                       ●

Process:
1. Hash the request key
2. Find position on ring
3. Move clockwise
4. First cache server = Selected
```

**Example:**

```
Request: GET user:123

1. Hash("user:123") → Position on ring
2. Rotate clockwise
3. First cache server: Cache 3
4. Store/Retrieve from Cache 3
```

**Note:** For in-depth consistent hashing, see HLD-06 video

---

## Caching Strategies

### Overview

```
5 Caching Strategies:

READ Strategies:
1. Cache-Aside (Lazy Loading)
2. Read-Through

WRITE Strategies:
3. Write-Around
4. Write-Through
5. Write-Back (Write-Behind)
```

---

### 1. Cache-Aside (Lazy Loading)

#### How It Works

```
READ Flow:

Client                App               Cache              DB
  │                    │                  │                 │
  │──── GET ──────────►│                  │                 │
  │                    │                  │                 │
  │                    │──── Check ──────►│                 │
  │                    │                  │                 │
  │                    │◄──── Hit/Miss ───│                 │
  │                    │                  │                 │
  │                    │                                    │
  │                    │ If HIT:                            │
  │                    │◄──── Data ───────│                 │
  │◄──── Response ─────│                  │                 │
  │                    │                                    │
  │                    │ If MISS:                           │
  │                    │──── Fetch ──────────────────────►  │
  │                    │                  │                 │
  │                    │◄──── Data ───────────────────────  │
  │                    │                  │                 │
  │                    │──── Update ─────►│                 │
  │                    │                  │                 │
  │◄──── Response ─────│                  │                 │
  │                    │                  │                 │
```

**Step-by-Step:**

```
1. Client sends GET request
2. App checks cache
3. Cache Hit:
   - Return data from cache
   - Fast! ✓
4. Cache Miss:
   - Fetch from DB
   - Update cache
   - Return data
```

---

#### Sequence Diagram

```
┌────────┐    ┌─────────┐    ┌───────┐    ┌──────┐
│ Client │    │   App   │    │ Cache │    │  DB  │
└────────┘    └─────────┘    └───────┘    └──────┘
     │             │              │            │
     │─── GET ────►│              │            │
     │             │              │            │
     │             │─── Read ────►│            │
     │             │              │            │
     │             │◄─── Data ────│            │
     │             │   (Cache Hit)│            │
     │             │              │            │
     │◄── Data ────│              │            │
     │             │              │            │
     
     
     │─── GET ────►│              │            │
     │             │              │            │
     │             │─── Read ────►│            │
     │             │              │            │
     │             │◄─── Miss ────│            │
     │             │              │            │
     │             │─── Fetch ───────────────►│
     │             │              │            │
     │             │◄─── Data ────────────────│
     │             │              │            │
     │             │─── Write ───►│            │
     │             │              │            │
     │◄── Data ────│              │            │
     │             │              │            │
```

---

#### Advantages

**1. Good for Read-Heavy Applications**

```
Scenario: News website

Reads: 10,000 requests/sec
Writes: 10 requests/sec

Cache-Aside:
- Most reads from cache (fast)
- Few writes to DB
- Perfect fit ✓
```

**2. Cache Failure Doesn't Break System**

```
Cache goes down:

Request → App → Cache ✗ (Down)
                │
                └─► Treated as cache miss
                    Fetch from DB ✓
                    System still works!

Resilient ✓
```

**3. Independent Cache Data Model**

```
DB Structure:
Table: Employee
├─ id
├─ name
├─ address
├─ dept_id

Table: Salary
├─ emp_id
├─ amount
├─ currency

Cache Structure:
Key: employee:123
Value: {
  "id": 123,
  "name": "John",
  "address": "NYC",
  "salary": {
    "amount": 50000,
    "currency": "USD"
  }
}

App controls cache structure ✓
Can denormalize, combine tables
Optimized for reads
```

---

#### Disadvantages

**1. New Data Always Cache Miss**

```
Timeline:

T0: POST /users (create user:456)
    → Writes to DB only
    → Cache not updated

T1: GET /users/456
    → Check cache: Miss ✗
    → Fetch from DB
    → Update cache
    → Return data

First read always slow
```

**2. Inconsistency Risk**

```
Problem: Stale data in cache

Timeline:

T0: GET user:123
    Cache: Miss
    DB: value = 10
    Cache updated: value = 10 ✓

T1: PUT user:123 (update to 11)
    DB updated: value = 11 ✓
    Cache NOT updated ✗

T2: GET user:123
    Cache: Hit (value = 10) ✗ STALE!
    DB: value = 11
    
Inconsistency! Cache has old data
```

**Visual:**

```
┌───────┐         ┌──────┐
│ Cache │         │  DB  │
│       │         │      │
│ v=10  │         │ v=10 │
└───────┘         └──────┘
                     ↓
                  PUT v=11
                     ↓
┌───────┐         ┌──────┐
│ Cache │         │  DB  │
│       │         │      │
│ v=10  │ ✗       │ v=11 │ ✓
│(Stale)│         │(Fresh)│
└───────┘         └──────┘

Solution: Use with Write-Around or Write-Through
```

---

### 2. Read-Through Cache

#### How It Works

```
Similar to Cache-Aside, but:
- Cache library handles DB fetch
- App doesn't manage cache updates
```

**Sequence Diagram:**

```
┌────────┐    ┌─────────┐    ┌───────────┐    ┌──────┐
│ Client │    │   App   │    │Cache Lib  │    │  DB  │
└────────┘    └─────────┘    └───────────┘    └──────┘
     │             │                │              │
     │─── GET ────►│                │              │
     │             │                │              │
     │             │─── Read ──────►│              │
     │             │                │              │
     │             │                │              │
     │             │   Cache Hit:   │              │
     │             │◄─── Data ──────│              │
     │             │                │              │
     │◄── Data ────│                │              │
     │             │                │              │
     
     │─── GET ────►│                │              │
     │             │                │              │
     │             │─── Read ──────►│              │
     │             │                │              │
     │             │   Cache Miss:  │              │
     │             │                │─── Fetch ───►│
     │             │                │              │
     │             │                │◄─── Data ────│
     │             │                │              │
     │             │                │ (Updates     │
     │             │                │  itself)     │
     │             │                │              │
     │             │◄─── Data ──────│              │
     │             │                │              │
     │◄── Data ────│                │              │
     │             │                │              │
```

**Key Difference:**

```
Cache-Aside:
App handles:
- Check cache
- Fetch from DB (if miss)
- Update cache

Read-Through:
Cache library handles:
- Check cache
- Fetch from DB (if miss)
- Update cache

App just calls cache.get(key)
```

---

#### Advantages

**1. Good for Read-Heavy Applications**

```
Same as Cache-Aside
Optimized for reads
```

**2. Separation of Concerns**

```
App code:
data = cache.get("user:123")
// That's it!

Cache library handles:
- Cache hit/miss logic
- DB fetch
- Cache update

Cleaner code ✓
```

---

#### Disadvantages

**1. New Data Always Cache Miss**

```
Same as Cache-Aside
First read always slow
```

**2. Inconsistency Risk**

```
Same as Cache-Aside
Needs write strategy
```

**3. Cache Structure = DB Structure**

```
DB Table:
Employee (id, name, address, dept, salary, ...)
20 fields

Cache:
Key: employee:123
Value: {
  "id": 123,
  "name": "John",
  "address": "NYC",
  "dept": "Engineering",
  "salary": 50000,
  ... (all 20 fields)
}

1:1 mapping with DB ✗
Cannot customize structure
Cannot denormalize
```

---

### 3. Write-Around Cache

#### How It Works

```
Write Strategy:
- Write directly to DB
- Don't update cache
- Invalidate cache entry (mark dirty)
```

**Sequence Diagram:**

```
┌────────┐    ┌─────────┐    ┌───────┐    ┌──────┐
│ Client │    │   App   │    │ Cache │    │  DB  │
└────────┘    └─────────┘    └───────┘    └──────┘
     │             │              │            │
     │─── PUT ────►│              │            │
     │             │              │            │
     │             │─── Write ───────────────►│
     │             │              │            │
     │             │◄─── OK ──────────────────│
     │             │              │            │
     │             │─── Invalidate│            │
     │             │   (mark dirty)           │
     │             │              │            │
     │◄── OK ─────│              │            │
     │             │              │            │
     
Next GET:
     │─── GET ────►│              │            │
     │             │              │            │
     │             │─── Read ────►│            │
     │             │              │            │
     │             │◄─── Dirty ───│            │
     │             │   (Cache Miss)           │
     │             │              │            │
     │             │─── Fetch ───────────────►│
     │             │              │            │
     │             │◄─── Data ────────────────│
     │             │              │            │
     │             │─── Update ──►│            │
     │             │   (fresh data)           │
     │             │              │            │
     │◄── Data ────│              │            │
     │             │              │            │
```

**Example:**

```
Initial State:
Cache: user:123 = {value: 10}
DB: user:123 = {value: 10}

PUT user:123 (value: 11):
1. Write to DB: value = 11 ✓
2. Invalidate cache: dirty = true

State After PUT:
Cache: user:123 = {value: 10, dirty: true}
DB: user:123 = {value: 11}

GET user:123:
1. Check cache: dirty = true → Cache Miss
2. Fetch from DB: value = 11
3. Update cache: value = 11, dirty = false
4. Return: 11 ✓
```

---

#### Advantages

**1. Good for Read-Heavy Applications**

```
Used WITH Cache-Aside or Read-Through
Solves inconsistency problem
```

**2. Resolves Inconsistency**

```
Write-Around ensures:
- Writes go to DB
- Cache invalidated
- Next read gets fresh data

No stale data ✓
```

---

#### Disadvantages

**1. New Data Always Cache Miss**

```
POST /users (new user)
→ Writes to DB only
→ Cache not touched

GET /users/new
→ Cache miss
→ Fetch from DB
```

**2. Not Fault Tolerant**

```
DB goes down:

PUT request → DB ✗ (Down)
              Write fails ✗

System unavailable for writes ✗
```

**Visual:**

```
┌─────────┐
│   App   │
└─────────┘
     │
     │ PUT request
     ▼
┌─────────┐
│   DB    │
│  ✗ DOWN │
└─────────┘

Write fails ✗
No fault tolerance
```

---

### 4. Write-Through Cache

#### How It Works

```
Write Strategy:
1. Write to cache first
2. Write to DB synchronously
3. Both must succeed (2-phase commit)
```

**Sequence Diagram:**

```
┌────────┐    ┌─────────┐    ┌───────┐    ┌──────┐
│ Client │    │   App   │    │ Cache │    │  DB  │
└────────┘    └─────────┘    └───────┘    └──────┘
     │             │              │            │
     │─── POST ───►│              │            │
     │             │              │            │
     │             │─── Write ───►│            │
     │             │              │            │
     │             │◄─── OK ──────│            │
     │             │              │            │
     │             │─── Write ───────────────►│
     │             │   (Synchronous)          │
     │             │              │            │
     │             │◄─── OK ──────────────────│
     │             │              │            │
     │◄── OK ─────│              │            │
     │             │              │            │
     
If DB fails:
     │─── POST ───►│              │            │
     │             │              │            │
     │             │─── Write ───►│            │
     │             │              │            │
     │             │◄─── OK ──────│            │
     │             │              │            │
     │             │─── Write ───────────────►│
     │             │              │            │
     │             │◄─── FAIL ────────────────│
     │             │              │            │
     │             │─── Rollback►│            │
     │             │              │            │
     │◄── FAIL ────│              │            │
     │             │              │            │
```

**Example:**

```
POST user:123 (value: 10):

1. Write to cache: value = 10 ✓
2. Write to DB: value = 10 ✓
3. Both succeed → Return success

State:
Cache: user:123 = {value: 10}
DB: user:123 = {value: 10}
Consistent! ✓


POST user:456 (value: 20):

1. Write to cache: value = 20 ✓
2. Write to DB: FAIL ✗
3. Rollback cache
4. Return failure

State:
Cache: user:456 = (not present)
DB: user:456 = (not present)
Consistent! ✓
```

---

#### Advantages

**1. Cache and DB Always Consistent**

```
2-Phase Commit:
- Both succeed → Consistent
- Either fails → Rollback → Consistent

No stale data ✓
```

**2. Increased Cache Hits**

```
POST user:123 (new user)
→ Written to cache ✓
→ Written to DB ✓

GET user:123
→ Cache hit! ✓
→ Fast response

Even new data in cache
```

---

#### Disadvantages

**1. Cannot Use Alone**

```
Write-Through alone:
- Writes to cache + DB
- But who reads from cache?

Need read strategy:
- Cache-Aside OR
- Read-Through

Otherwise just added latency ✗
```

**2. Two-Phase Commit Required**

```
Complexity:
- Transaction management
- Rollback logic
- Error handling

Implementation overhead
```

**3. Not Fully Fault Tolerant**

```
Cache goes down:
POST → Cache ✗ (Down)
       Write fails ✗

DB goes down:
POST → Cache ✓
       DB ✗ (Down)
       Rollback cache
       Write fails ✗

Either down → Writes fail ✗
```

---

### 5. Write-Back (Write-Behind) Cache

#### How It Works

```
Write Strategy:
1. Write to cache
2. Queue write to DB (asynchronous)
3. Return success immediately
4. DB write happens later
```

**Sequence Diagram:**

```
┌────────┐  ┌─────────┐  ┌───────┐  ┌───────┐  ┌──────┐
│ Client │  │   App   │  │ Cache │  │ Queue │  │  DB  │
└────────┘  └─────────┘  └───────┘  └───────┘  └──────┘
     │           │            │          │          │
     │─ POST ───►│            │          │          │
     │           │            │          │          │
     │           │─ Write ───►│          │          │
     │           │            │          │          │
     │           │◄─ OK ──────│          │          │
     │           │            │          │          │
     │           │─ Publish ─────────►  │          │
     │           │            │          │          │
     │◄─ OK ─────│            │          │          │
     │           │            │          │          │
     
     (Async - later)
                              │          │          │
                              │          │─ Write ─►│
                              │          │          │
                              │          │◄─ OK ────│
                              │          │          │
```

**Example:**

```
POST user:123 (value: 10):

1. Write to cache: value = 10 ✓
2. Publish to queue: {write user:123, value: 10}
3. Return success immediately ✓

State (immediately):
Cache: user:123 = {value: 10}
Queue: [write user:123 = 10]
DB: (not updated yet)

State (after async processing):
Cache: user:123 = {value: 10}
Queue: []
DB: user:123 = {value: 10}
```

---

#### Advantages

**1. Good for Write-Heavy Applications**

```
Scenario: Analytics logging

Writes: 10,000 events/sec
Reads: 100 queries/sec

Write-Back:
- All writes to cache (fast)
- Async DB writes (batched)
- No write bottleneck ✓
```

**2. Reduced Write Latency**

```
Write-Through:
POST → Cache (1ms) + DB (100ms) = 101ms

Write-Back:
POST → Cache (1ms) + Queue (1ms) = 2ms

50x faster! ✓
```

**3. Increased Cache Hits**

```
POST user:123
→ Cache updated immediately

GET user:123
→ Cache hit (latest data) ✓

Always fresh data in cache
```

**4. Fault Tolerant**

```
DB goes down for 2 hours:

Writes:
→ Cache ✓ (works)
→ Queue ✓ (accumulates)
→ System available ✓

Reads:
→ Cache ✓ (has latest data)
→ System available ✓

When DB comes back:
→ Queue processes backlog
→ DB catches up

Fault tolerant! ✓
```

---

#### Disadvantages

**1. Data Loss Risk**

```
Problem: Cache TTL < DB downtime

Timeline:

T0: POST user:123 (value: 10)
    Cache: value = 10 (TTL: 3 hours)
    Queue: [write user:123 = 10]
    DB: (down)

T1 (1 hour): DB still down
             Queue trying to write

T2 (3 hours): Cache TTL expires
              Cache: (evicted)
              DB: still down
              Queue: still trying

T3 (5 hours): DB comes back
              Queue writes: value = 10
              But cache empty!

T4: GET user:123
    Cache: Miss
    DB: value = 10
    Cache updated

Risk period: T2 to T4
If app had cached value in memory → Lost!
```

**Visual:**

```
Timeline:
T0        T1        T2        T3        T4
│         │         │         │         │
POST      │    Cache│    DB   │    GET  │
          │    TTL  │   comes │         │
          │   expires│   back │         │
          │         │         │         │
Cache: 10 │    10   │   empty │   empty │   10
Queue: [10]   [10]     [10]      []       []
DB:    down   down     down      10       10
                      ↑
                Data loss risk!
```

**2. Best Used With Read Strategy**

```
Write-Back alone:
- Fast writes ✓
- But reads?

With Cache-Aside or Read-Through:
- Fast writes ✓
- Fast reads ✓
- Complete solution ✓
```

---

## Summary

### Strategy Comparison

```
┌──────────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│   Strategy   │  Cache-  │  Read-   │  Write-  │  Write-  │  Write-  │
│              │  Aside   │ Through  │  Around  │ Through  │   Back   │
├──────────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ Type         │   Read   │   Read   │  Write   │  Write   │  Write   │
│              │          │          │          │          │          │
│ App manages  │   Yes    │    No    │   Yes    │   Yes    │   Yes    │
│ cache        │          │(library) │          │          │          │
│              │          │          │          │          │          │
│ Consistency  │   Risk   │   Risk   │   Good   │Excellent │   Good   │
│              │          │          │          │          │          │
│ Cache hits   │  Medium  │  Medium  │  Medium  │   High   │   High   │
│ for new data │          │          │          │          │          │
│              │          │          │          │          │          │
│ Write        │   N/A    │   N/A    │  Medium  │   Slow   │   Fast   │
│ latency      │          │          │          │          │          │
│              │          │          │          │          │          │
│ Fault        │   Good   │   Good   │   Poor   │   Poor   │Excellent │
│ tolerance    │          │          │          │          │          │
│              │          │          │          │          │          │
│ Use alone    │   Yes    │   Yes    │    No    │    No    │    No    │
│              │          │          │          │(need read│(need read│
│              │          │          │          │strategy) │strategy) │
└──────────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

---

### Common Combinations

```
1. Cache-Aside + Write-Around
   - Read-heavy apps
   - Consistency important
   - Simple implementation

2. Read-Through + Write-Through
   - Consistency critical
   - Library-managed cache
   - Transactional systems

3. Cache-Aside + Write-Back
   - Write-heavy apps
   - High performance needed
   - Fault tolerance required

4. Read-Through + Write-Back
   - Best performance
   - Library-managed
   - Complex implementation
```

---

### Decision Matrix

```
Choose based on:

Read-Heavy + Simple:
→ Cache-Aside + Write-Around

Read-Heavy + Library-managed:
→ Read-Through + Write-Around

Write-Heavy + Performance:
→ Cache-Aside + Write-Back

Consistency Critical:
→ Read-Through + Write-Through

Fault Tolerance Critical:
→ Cache-Aside + Write-Back
```

---

## Interview Tips

### Common Questions

**Q1: "Explain Cache-Aside strategy"**

```
Answer:
"Cache-Aside is a lazy loading strategy.

Read flow:
1. App checks cache
2. Cache hit → Return data
3. Cache miss → Fetch from DB, update cache, return

Advantages:
✓ Good for read-heavy apps
✓ Cache failure doesn't break system
✓ Custom cache structure

Disadvantages:
✗ New data always cache miss
✗ Inconsistency risk (needs write strategy)

Example: News website, product catalog"
```

**Q2: "Cache-Aside vs Read-Through?"**

```
Answer:
"Both are read strategies, key difference is who manages cache:

Cache-Aside:
- App handles cache logic
- App fetches from DB on miss
- App updates cache
- More control, more code

Read-Through:
- Cache library handles logic
- Library fetches from DB on miss
- Library updates cache
- Less code, less control

Trade-off:
Cache-Aside: Flexibility (custom cache structure)
Read-Through: Simplicity (1:1 DB mapping)

Choose Cache-Aside when need custom cache structure
Choose Read-Through when want simpler code"
```

**Q3: "How does Write-Back provide fault tolerance?"**

```
Answer:
"Write-Back is fault tolerant because writes don't depend on DB.

Normal flow:
1. Write to cache ✓
2. Queue DB write (async)
3. Return success immediately

DB goes down:
1. Write to cache ✓ (still works)
2. Queue accumulates writes
3. Reads from cache ✓ (has latest data)
4. System remains available

When DB recovers:
- Queue processes backlog
- DB catches up

Tolerance window = Cache TTL
If cache TTL = 24 hours, can tolerate 24hr DB outage

Risk: If DB down > Cache TTL → Data loss

Example: Analytics, logging systems"
```

**Q4: "Why can't Write-Through be used alone?"**

```
Answer:
"Write-Through only handles writes, not reads.

Write-Through alone:
POST → Cache + DB ✓
GET → Where to read from? ✗

Without read strategy:
- Cache updated but not read
- Just added latency
- No benefit

Must combine with:
- Cache-Aside OR
- Read-Through

Complete solution:
Writes: Write-Through (consistency)
Reads: Cache-Aside/Read-Through (performance)

Example:
Read-Through + Write-Through = Full consistency"
```

**Q5: "Explain inconsistency problem in Cache-Aside"**

```
Answer:
"Cache-Aside only handles reads, writes go directly to DB.

Problem scenario:

T0: GET user:123
    Cache miss → Fetch from DB (value=10)
    Update cache (value=10)

T1: PUT user:123 (value=11)
    Update DB (value=11) ✓
    Cache NOT updated ✗

T2: GET user:123
    Cache hit (value=10) ✗ STALE!
    DB has value=11

Inconsistency: Cache stale, DB fresh

Solutions:
1. Write-Around: Invalidate cache on write
2. Write-Through: Update cache on write
3. TTL: Cache expires eventually
4. Manual invalidation: App invalidates on write

Best: Combine Cache-Aside with Write-Around or Write-Through"
```

### Key Points to Remember

```
1. Caching = Fast memory for frequent data

2. Benefits:
   - Faster response (low latency)
   - Fault tolerance (depends on strategy)
   - Reduced DB load

3. Distributed caching = Consistent hashing

4. 5 Strategies:
   READ: Cache-Aside, Read-Through
   WRITE: Write-Around, Write-Through, Write-Back

5. Cache-Aside:
   - App manages cache
   - Custom structure
   - Inconsistency risk

6. Read-Through:
   - Library manages cache
   - 1:1 DB mapping
   - Simpler code

7. Write-Around:
   - Write to DB, invalidate cache
   - Solves inconsistency
   - Not fault tolerant

8. Write-Through:
   - Write to cache + DB (sync)
   - Always consistent
   - Slower writes

9. Write-Back:
   - Write to cache, queue DB (async)
   - Fast writes
   - Fault tolerant
   - Data loss risk

10. Combine strategies:
    - Cache-Aside + Write-Around
    - Read-Through + Write-Through
    - Cache-Aside + Write-Back
```

### Do's ✅

**1. Explain with Sequence Diagrams**
```
"Let me draw the flow:
Client → App → Cache → DB
This helps visualize the strategy"
```

**2. Mention Trade-offs**
```
"Cache-Aside gives flexibility but requires more code.
Read-Through is simpler but less flexible."
```

**3. Give Real Examples**
```
"Cache-Aside: Product catalog (read-heavy)
Write-Back: Analytics logging (write-heavy)"
```

### Don'ts ❌

**1. Don't Confuse Read and Write Strategies**
```
❌ "Cache-Aside handles writes"
✓ "Cache-Aside is a read strategy"
```

**2. Don't Forget Combinations**
```
❌ "Use Write-Through alone"
✓ "Combine Write-Through with read strategy"
```

**3. Don't Ignore Consistency**
```
❌ "Cache-Aside is perfect"
✓ "Cache-Aside has inconsistency risk, needs write strategy"
```

---

**End of Lecture (Part 1)**

Caching is critical for system performance. Understanding the 5 strategies (Cache-Aside, Read-Through, Write-Around, Write-Through, Write-Back) and their trade-offs is essential. Remember: read strategies (Cache-Aside, Read-Through) handle reads, write strategies (Write-Around, Write-Through, Write-Back) handle writes. Combine them based on requirements: consistency, performance, or fault tolerance.

**Key Takeaway:** Cache-Aside for flexibility, Read-Through for simplicity, Write-Around for consistency, Write-Through for strong consistency, Write-Back for performance and fault tolerance. Always combine read + write strategies!

**Part 2 Preview:** Cache eviction policies (LRU, LFU, FIFO, etc.)
