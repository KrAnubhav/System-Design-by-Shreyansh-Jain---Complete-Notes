# HLD-09: Idempotency Handling

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Idempotency vs Concurrency](#idempotency-vs-concurrency)
3. [What is Idempotency?](#what-is-idempotency)
4. [HTTP Methods and Idempotency](#http-methods-and-idempotency)
   - [GET, PUT, DELETE (Already Idempotent)](#get-put-delete-already-idempotent)
   - [POST (Not Idempotent)](#post-not-idempotent)
5. [Types of Duplicate Requests](#types-of-duplicate-requests)
   - [Sequential Duplicates](#sequential-duplicates)
   - [Parallel Duplicates](#parallel-duplicates)
6. [Solution: Idempotency Key](#solution-idempotency-key)
   - [What is Idempotency Key?](#what-is-idempotency-key)
   - [Client-Server Agreement](#client-server-agreement)
7. [Idempotency Flow](#idempotency-flow)
   - [Original Request Flow](#original-request-flow)
   - [Duplicate Request Flow](#duplicate-request-flow)
8. [Handling Parallel Requests](#handling-parallel-requests)
9. [Multi-Cluster/Geo-Distributed Systems](#multi-clustergeo-distributed-systems)
10. [Summary](#summary)
11. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Idempotency Handling

**Importance:**
- Very important interview question
- Recently asked in top Singapore-based product companies
- Critical for production systems

**What We'll Cover:**
- ✅ Idempotency handling
- ❌ NOT covering concurrency (separate topic for next video)

---

## Idempotency vs Concurrency

### Concurrency

**Definition:** Multiple users trying to access the same resource

```
         Resource
            │
    ┌───────┼───────┐
    │       │       │
    ▼       ▼       ▼
 User 1  User 2  User 3

All trying to access same resource
```

**Example: Movie Ticket Booking**

```
Seat A1 (Resource)
    │
    ├─ User 1 trying to book
    ├─ User 2 trying to book
    └─ User 3 trying to book

Problem: Who gets the seat?
Solution: Concurrency control (locks, etc.)
```

---

### Idempotency

**Definition:** Handling duplicate requests safely

```
Client → Server (Request 1)
Client → Server (Request 2 - Duplicate)
Client → Server (Request 3 - Duplicate)

All requests should have same effect as single request
```

**Key Difference:**

```
┌─────────────────┬──────────────────────────────┐
│   Concurrency   │        Idempotency           │
├─────────────────┼──────────────────────────────┤
│ Multiple users  │ Same user, duplicate request │
│ Same resource   │ Same operation               │
│ Who gets it?    │ Don't repeat side effects    │
└─────────────────┴──────────────────────────────┘
```

**Important:** Don't confuse these two concepts!

---

## What is Idempotency?

### Definition

**Idempotency** = Enables client to safely retry an operation without worrying about side effects

```
Operation can be retried multiple times
→ Same result as executing once
→ No duplicate side effects
```

### Example: Add to Cart

**Without Idempotency:**

```
Request 1: Add item to cart
→ DB: 1 item added ✓

Request 2 (Duplicate): Add item to cart
→ DB: 1 more item added ✗ (Wrong!)

Request 3 (Duplicate): Add item to cart
→ DB: 1 more item added ✗ (Wrong!)

Result: 3 items in cart (Should be 1)
```

**With Idempotency:**

```
Request 1: Add item to cart
→ DB: 1 item added ✓

Request 2 (Duplicate): Add item to cart
→ DB: No change (Duplicate detected) ✓

Request 3 (Duplicate): Add item to cart
→ DB: No change (Duplicate detected) ✓

Result: 1 item in cart (Correct!)
```

### Key Principle

```
Client can retry as many times as needed
→ DB will have only ONE record
→ Safe to retry without side effects
```

---

## HTTP Methods and Idempotency

### GET, PUT, DELETE (Already Idempotent)

#### GET Request

```
Client → Server: GET /user/123
Server → Client: { name: "Shreyansh", ... }

Retry:
Client → Server: GET /user/123
Server → Client: { name: "Shreyansh", ... }

No side effects on DB ✓
Data remains same ✓
Idempotent by nature ✓
```

**Characteristics:**
- No DB changes
- Just returns data
- Safe to retry

---

#### PUT Request

```
Request 1:
Client → Server: PUT /user/123
                  { name: "Shreyansh" }
Server: Updates name from "SJ" to "Shreyansh"

Retry (Duplicate):
Client → Server: PUT /user/123
                  { name: "Shreyansh" }
Server: Name already "Shreyansh", no change

No side effects ✓
Idempotent by nature ✓
```

**Characteristics:**
- Updates to same value
- No additional changes
- Safe to retry

---

#### DELETE Request

```
Request 1:
Client → Server: DELETE /user/123
Server: Deletes user 123

Retry (Duplicate):
Client → Server: DELETE /user/123
Server: User 123 already deleted, no change

No side effects ✓
Idempotent by nature ✓
```

**Characteristics:**
- Already deleted
- No additional changes
- Safe to retry

---

### POST (Not Idempotent)

**Problem:** POST creates new resources

#### Example: Payment

```
Request 1: POST /payment
           { from: "Shreyansh", to: "Hardik", amount: 10 }
DB: 
  Shreyansh: -10
  Hardik: +10

Request 2 (Duplicate): POST /payment
                       { from: "Shreyansh", to: "Hardik", amount: 10 }
DB:
  Shreyansh: -10 (again!) ✗
  Hardik: +10 (again!) ✗

Request 3 (Duplicate): POST /payment
                       { from: "Shreyansh", to: "Hardik", amount: 10 }
DB:
  Shreyansh: -10 (again!) ✗
  Hardik: +10 (again!) ✗

Result: Shreyansh lost 30, Hardik gained 30
        (Should be 10 each)
```

**Problem:**
```
POST creates new row each time
→ Duplicate requests = Multiple rows
→ Side effects repeated
→ NOT idempotent by nature
```

**Solution:** We must make POST idempotent!

---

## Types of Duplicate Requests

### Sequential Duplicates

**Scenario:** Timeout causes retry

```
Step 1: Client sends request
┌────────┐         ┌────────┐
│ Client │────────►│ Server │
└────────┘         └────────┘
                       │
                       ▼
                   Processing...

Step 2: Timeout occurs (but server still processing)
┌────────┐         ┌────────┐
│ Client │   ✗     │ Server │
└────────┘ Timeout └────────┘
                       │
                       ▼
                   Still processing...
                       │
                       ▼
                   Processed ✓

Step 3: Client retries (duplicate request)
┌────────┐         ┌────────┐
│ Client │────────►│ Server │
└────────┘ Retry   └────────┘
                       │
                       ▼
                   Duplicate!
```

**Timeline:**

```
T0: Request sent
T1: Server processing
T2: Timeout (client side)
T3: Server completes processing
T4: Client retries (duplicate)
```

**Problem:**
- Server already processed original request
- Client doesn't know (timeout)
- Retry creates duplicate

---

### Parallel Duplicates

**Scenario:** Multiple requests at same time

```
┌────────┐         ┌────────┐
│ Client │────────►│Server 1│
└────────┘ Request └────────┘
    │
    │
    └─────────────►┌────────┐
           Request │Server 2│
                   └────────┘

Both requests at same time
Same operation
Different servers (maybe)
```

**Causes:**
- Different browser tabs
- Network issues
- Different servers in load balancer

**Problem:**
- Both requests processed simultaneously
- Both create resources
- Duplicate side effects

---

## Solution: Idempotency Key

### What is Idempotency Key?

**Idempotency Key** = Unique identifier for each operation

```
Characteristics:
✓ Unique (UUID - Universal Unique ID)
✓ Generated by client
✓ Sent with request
✓ Used to identify duplicates
```

**Example:**

```
Idempotency Key: "abc123-def456-ghi789"

Properties:
- Universally unique
- Cannot be duplicated
- Identifies specific operation
```

---

### Client-Server Agreement

**Two Key Agreements:**

#### Agreement 1: Client Generates Key

```
Client responsibility:
- Generate idempotency key
- Use UUID or similar
- Ensure uniqueness
```

**Generation Methods:**

```
Option 1: UUID
Key = UUID.generate()
Example: "550e8400-e29b-41d4-a716-446655440000"

Option 2: UUID + Operation
Key = UUID.generate() + operation_name
Example: "550e8400_add_to_cart"

Option 3: UUID + Timestamp
Key = UUID.generate() + timestamp
Example: "550e8400_1234567890"
```

#### Agreement 2: New Key Per Operation

```
Operation 1: Add item A to cart
Key: "key-001"

Operation 2: Add item B to cart
Key: "key-002" (Different!)

Same operation retried:
Key: "key-001" (Same!)
```

**Rule:**
```
Different operation → New key
Same operation retry → Same key
```

---

## Idempotency Flow

### Original Request Flow

**Step 1: User Initiates Operation**

```
User: "Add item to cart"
    ↓
Client Application
```

**Step 2: Client Generates Idempotency Key**

```
Client:
- Generate UUID
- Key = "IK-001"
```

**Step 3: Client Sets Key in Header**

```
POST /cart/add
Headers:
  Idempotency-Key: IK-001
Body:
  { item_id: 123, quantity: 1 }
```

**Step 4: Server Validates Key Presence**

```
Server receives request
    ↓
Check: Is Idempotency-Key in header?
    ├─ No → Return HTTP 400 (Validation Error)
    └─ Yes → Continue
```

**Step 5: Server Reads Key from DB**

```
Server: Query DB
SELECT * FROM idempotency_keys WHERE key = 'IK-001'

Result: Not found (first time)
```

**Step 6: Server Creates DB Entry**

```
INSERT INTO idempotency_keys
VALUES ('IK-001', 'CREATED')

┌─────────┬──────────┐
│   Key   │  Status  │
├─────────┼──────────┤
│ IK-001  │ CREATED  │
└─────────┴──────────┘
```

**Key Lifecycle:**

```
CREATED → Operation in progress
    ↓
CONSUMED → Operation completed
```

**Step 7: Execute Operation**

```
Server: Add item to cart
    ↓
Success!
```

**Step 8: Update Key Status**

```
UPDATE idempotency_keys
SET status = 'CONSUMED'
WHERE key = 'IK-001'

┌─────────┬──────────┐
│   Key   │  Status  │
├─────────┼──────────┤
│ IK-001  │ CONSUMED │
└─────────┴──────────┘
```

**Step 9: Return Success**

```
Server → Client: HTTP 201 (Created)

201 = Resource created successfully
```

**Complete Flow Diagram:**

```
User → Client → Generate Key (IK-001)
                    ↓
                Set in Header
                    ↓
                POST /cart/add
                    ↓
              Server Validates
                    ↓
              Read from DB (Not found)
                    ↓
              Create Entry (CREATED)
                    ↓
              Execute Operation
                    ↓
              Update Status (CONSUMED)
                    ↓
              Return 201
```

---

### Duplicate Request Flow

**Scenario:** Timeout occurred, client retries

**What Happened:**

```
Original Request:
T0: Client sends request
T1: Server processing
T2: Client timeout ✗
T3: Server completes (but client gone)
T4: Client retries (duplicate)
```

**Duplicate Request Steps:**

**Step 1-3: Same as Original**

```
Client retries with SAME key: IK-001
POST /cart/add
Headers:
  Idempotency-Key: IK-001
```

**Step 4: Server Validates**

```
Check: Is Idempotency-Key in header?
Yes → Continue
```

**Step 5: Server Reads Key from DB**

```
Server: Query DB
SELECT * FROM idempotency_keys WHERE key = 'IK-001'

Result: Found!
┌─────────┬──────────┐
│   Key   │  Status  │
├─────────┼──────────┤
│ IK-001  │ CONSUMED │
└─────────┴──────────┘
```

**Step 6: Check Status**

```
Key exists: Yes
Status: CONSUMED or CREATED?
```

**Case 1: Status = CONSUMED**

```
┌─────────┬──────────┐
│   Key   │  Status  │
├─────────┼──────────┤
│ IK-001  │ CONSUMED │
└─────────┴──────────┘

Meaning:
- Original request completed
- Resource already created
- This is duplicate

Action:
Return HTTP 200 (OK)

200 = Request successful, but no new resource created
```

**Case 2: Status = CREATED**

```
┌─────────┬──────────┐
│   Key   │  Status  │
├─────────┼──────────┤
│ IK-001  │ CREATED  │
└─────────┴──────────┘

Meaning:
- Original request still processing
- Not yet completed
- This is duplicate while processing

Action:
Return HTTP 409 (Conflict)

409 = Conflict, original request still in progress
      Please retry later
```

**Response Summary:**

```
┌──────────────┬──────────────┬─────────────────────┐
│ Key Status   │ HTTP Code    │ Meaning             │
├──────────────┼──────────────┼─────────────────────┤
│ Not Found    │ 201          │ New request, created│
│ CONSUMED     │ 200          │ Already completed   │
│ CREATED      │ 409          │ Still processing    │
└──────────────┴──────────────┴─────────────────────┘
```

**Complete Duplicate Flow:**

```
Duplicate Request
    ↓
Validate Key (Present)
    ↓
Read from DB (Found!)
    ↓
Check Status
    ├─ CONSUMED → Return 200 (Already done)
    └─ CREATED → Return 409 (Still processing)
```

---

## Handling Parallel Requests

### The Problem

**Scenario:** Two requests arrive simultaneously

```
Time T0:
┌────────┐         ┌────────┐
│ Client │────────►│Server 1│ Request 1 (IK-001)
└────────┘         └────────┘
    │
    └─────────────►┌────────┐
                   │Server 2│ Request 2 (IK-001)
                   └────────┘

Both requests:
- Same idempotency key (IK-001)
- Arrive at same time
- Different servers (maybe)
```

**What Happens Without Protection:**

```
Request 1:                    Request 2:
    ↓                             ↓
Validate (OK)                 Validate (OK)
    ↓                             ↓
Read DB (Not found)           Read DB (Not found)
    ↓                             ↓
Create Entry (CREATED)        Create Entry (CREATED)
    ↓                             ↓
Execute Operation             Execute Operation
    ↓                             ↓
Update (CONSUMED)             Update (CONSUMED)
    ↓                             ↓
Return 201                    Return 201

Result: Both succeed! ✗
        Two resources created ✗
```

**Problem:**
```
Both requests pass validation
Both create resources
Duplicate side effects
Idempotency broken!
```

---

### Solution: Mutual Exclusion (Mutex)

**Concept:** Only one request can enter critical section

#### Identify Critical Section

```
Critical Section:
┌─────────────────────────────────┐
│ 1. Read key from DB             │
│ 2. If not found, create entry   │
│ 3. Execute operation            │
│ 4. Update status to CONSUMED    │
└─────────────────────────────────┘

Only ONE request allowed at a time
```

#### Apply Mutex

```
Request 1:                    Request 2:
    ↓                             ↓
Validate (OK)                 Validate (OK)
    ↓                             ↓
Acquire Lock ✓                Try Acquire Lock ✗ (Wait...)
    ↓
┌─────────────────────────────────┐
│ CRITICAL SECTION (Locked)       │
│                                 │
│ Read DB (Not found)             │
│ Create Entry (CREATED)          │
│ Execute Operation               │
│ Update (CONSUMED)               │
└─────────────────────────────────┘
    ↓
Release Lock
    ↓                             ↓
Return 201                    Acquire Lock ✓
                                  ↓
                              ┌─────────────────────────────────┐
                              │ CRITICAL SECTION (Locked)       │
                              │                                 │
                              │ Read DB (Found! CONSUMED)       │
                              │ Return 200 (Duplicate)          │
                              └─────────────────────────────────┘
                                  ↓
                              Release Lock
                                  ↓
                              Return 200
```

**Result:**
```
Request 1: Creates resource (201) ✓
Request 2: Detects duplicate (200) ✓
No duplicate resources ✓
```

---

### Mutex Implementation Options

**1. Synchronized Block**

```java
synchronized(idempotencyKey) {
    // Critical section
    if (!db.exists(key)) {
        db.create(key, "CREATED");
        executeOperation();
        db.update(key, "CONSUMED");
    }
}
```

**2. Semaphore**

```java
Semaphore semaphore = new Semaphore(1);
semaphore.acquire();
try {
    // Critical section
} finally {
    semaphore.release();
}
```

**3. Database Lock**

```sql
SELECT * FROM idempotency_keys 
WHERE key = 'IK-001' 
FOR UPDATE; -- Row-level lock
```

**4. Distributed Lock (Redis)**

```
SETNX idempotency:IK-001 "locked"
-- Critical section
DEL idempotency:IK-001
```

---

### Complete Parallel Handling Flow

```
┌─────────────────────────────────────────────────┐
│         Parallel Request Handling               │
├─────────────────────────────────────────────────┤
│                                                 │
│  Request 1          Request 2                   │
│      ↓                  ↓                       │
│  Validate           Validate                    │
│      ↓                  ↓                       │
│  ┌──────────────────────────────┐              │
│  │   Try Acquire Mutex          │              │
│  └──────────────────────────────┘              │
│      ↓                  ↓                       │
│   Success            Wait...                    │
│      ↓                                          │
│  ┌──────────────────────────────┐              │
│  │   CRITICAL SECTION           │              │
│  │   - Read DB                  │              │
│  │   - Create if not exists     │              │
│  │   - Execute operation        │              │
│  │   - Update status            │              │
│  └──────────────────────────────┘              │
│      ↓                                          │
│  Release Mutex                                  │
│      ↓                  ↓                       │
│   Return 201        Acquire Mutex               │
│                         ↓                       │
│                     Read DB (Found)             │
│                         ↓                       │
│                     Return 200                  │
└─────────────────────────────────────────────────┘
```

---

## Multi-Cluster/Geo-Distributed Systems

### The Problem

**Scenario:** Requests to different clusters

```
                    ┌─────────────┐
                    │   Client    │
                    └─────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
    ┌─────────────────┐     ┌─────────────────┐
    │   Cluster 1     │     │   Cluster 2     │
    │   (US Region)   │     │   (EU Region)   │
    ├─────────────────┤     ├─────────────────┤
    │   Server 1      │     │   Server 2      │
    │       ↓         │     │       ↓         │
    │    DB 1         │     │    DB 2         │
    └─────────────────┘     └─────────────────┘
              │                       │
              └───────────┬───────────┘
                          │
                    DB Replication
                  (Minutes to sync)
```

**Problem:**

```
T0: Request 1 → Server 1 → DB 1
    Create entry (IK-001, CREATED)

T1: Request 2 (duplicate) → Server 2 → DB 2
    DB 2 doesn't have IK-001 yet (not synced)
    Creates entry again! ✗

DB Replication is slow (minutes)
Idempotency broken!
```

---

### Solution: Cache

**Use Cache for Idempotency Keys**

```
                    ┌─────────────┐
                    │   Client    │
                    └─────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
    ┌─────────────────┐     ┌─────────────────┐
    │   Cluster 1     │     │   Cluster 2     │
    ├─────────────────┤     ├─────────────────┤
    │   Server 1      │     │   Server 2      │
    │       ↓         │     │       ↓         │
    │   Cache 1       │     │   Cache 2       │
    │       ↓         │     │       ↓         │
    │    DB 1         │     │    DB 2         │
    └─────────────────┘     └─────────────────┘
              │                       │
              └───────────┬───────────┘
                          │
                  Cache Replication
                  (Milliseconds!)
```

**Why Cache?**

```
┌──────────────────┬─────────────┬──────────────┐
│   Aspect         │     DB      │    Cache     │
├──────────────────┼─────────────┼──────────────┤
│ Replication Time │  Minutes    │ Milliseconds │
│ Consistency      │  Eventual   │ Near Real-time│
│ Speed            │  Slow       │ Very Fast    │
└──────────────────┴─────────────┴──────────────┘
```

**Cache Synchronization:**
```
Cache 1 updates → Replicated to Cache 2 in milliseconds
Near real-time consistency
Idempotency maintained across clusters
```

---

### Cache-Based Flow

**Step 1: Check Cache First**

```
Request arrives
    ↓
Check Cache for idempotency key
    ├─ Found → Handle as duplicate
    └─ Not found → Continue
```

**Step 2: Create in Cache**

```
Cache.set(key, "CREATED", TTL=10min)
    ↓
Replicated to other caches (milliseconds)
    ↓
Execute operation
    ↓
Cache.set(key, "CONSUMED", TTL=24hr)
```

**Step 3: Persist to DB (Async)**

```
Cache updated (fast)
    ↓
Async write to DB (slower, but okay)
    ↓
DB eventually consistent
```

**Benefits:**

```
✓ Fast synchronization (milliseconds)
✓ Near real-time consistency
✓ Works across geo-distributed clusters
✓ Idempotency maintained globally
```

---

### Complete Multi-Cluster Architecture

```
┌─────────────────────────────────────────────────┐
│         Global Idempotency System               │
├─────────────────────────────────────────────────┤
│                                                 │
│  Cluster 1 (US)              Cluster 2 (EU)     │
│  ┌──────────────┐           ┌──────────────┐   │
│  │  Server 1    │           │  Server 2    │   │
│  └──────────────┘           └──────────────┘   │
│         ↓                           ↓           │
│  ┌──────────────┐           ┌──────────────┐   │
│  │ Redis Cache  │←─────────→│ Redis Cache  │   │
│  │  (Primary)   │  Replicate│  (Replica)   │   │
│  └──────────────┘  <10ms    └──────────────┘   │
│         ↓                           ↓           │
│  ┌──────────────┐           ┌──────────────┐   │
│  │   DB 1       │←─────────→│   DB 2       │   │
│  │              │  Replicate│              │   │
│  └──────────────┘  Minutes  └──────────────┘   │
│                                                 │
│  Flow:                                          │
│  1. Check cache (fast)                          │
│  2. Update cache (replicated in milliseconds)   │
│  3. Async write to DB (eventual consistency)    │
└─────────────────────────────────────────────────┘
```

---

## Summary

### Key Concepts

**1. Idempotency Definition**
```
Safe retry of operations
No duplicate side effects
Same result regardless of retry count
```

**2. HTTP Methods**
```
Idempotent by nature:
✓ GET
✓ PUT
✓ DELETE

Not idempotent:
✗ POST (need to make it idempotent)
```

**3. Duplicate Types**
```
Sequential: Timeout → Retry
Parallel: Simultaneous requests
```

**4. Solution: Idempotency Key**
```
Client generates unique key
Server tracks key status
Detects and handles duplicates
```

**5. Key Lifecycle**
```
CREATED → Operation in progress
CONSUMED → Operation completed
```

**6. Response Codes**
```
201: Resource created (original)
200: Already completed (duplicate)
409: Still processing (duplicate)
400: Validation error (no key)
```

---

### Complete Flow Summary

```
┌─────────────────────────────────────────────────┐
│         Idempotency Handling Flow               │
├─────────────────────────────────────────────────┤
│                                                 │
│  1. Client generates idempotency key            │
│     ↓                                           │
│  2. Client sets key in request header           │
│     ↓                                           │
│  3. Server validates key presence               │
│     ├─ No key → Return 400                      │
│     └─ Has key → Continue                       │
│     ↓                                           │
│  4. Server reads key from cache/DB              │
│     ├─ Not found → Create entry (CREATED)       │
│     │              Execute operation            │
│     │              Update status (CONSUMED)     │
│     │              Return 201                   │
│     │                                           │
│     └─ Found → Check status                     │
│                ├─ CONSUMED → Return 200         │
│                └─ CREATED → Return 409          │
│                                                 │
│  5. For parallel requests: Use mutex            │
│     ↓                                           │
│  6. For multi-cluster: Use cache                │
└─────────────────────────────────────────────────┘
```

---

### Architecture Components

```
┌────────────────────────────────────────────┐
│     Idempotency System Components         │
├────────────────────────────────────────────┤
│                                            │
│  1. Idempotency Key Store                  │
│     - Cache (Redis) for fast access        │
│     - DB for persistence                   │
│                                            │
│  2. Key Lifecycle Management               │
│     - CREATED state                        │
│     - CONSUMED state                       │
│                                            │
│  3. Mutex/Lock for Parallel Requests       │
│     - Synchronized blocks                  │
│     - Distributed locks (Redis)            │
│                                            │
│  4. Cache Replication (Multi-cluster)      │
│     - Millisecond sync                     │
│     - Near real-time consistency           │
└────────────────────────────────────────────┘
```

---

## Interview Tips

### Do's ✅

**1. Explain the Problem First**
```
"POST requests create new resources each time.
Without idempotency, retries create duplicates.
This causes issues like double payments."
```

**2. Differentiate from Concurrency**
```
"Idempotency handles duplicate requests from same user.
Concurrency handles multiple users accessing same resource.
Different problems, different solutions."
```

**3. Explain Key Lifecycle**
```
"Key has two states:
- CREATED: Operation in progress
- CONSUMED: Operation completed

This helps handle duplicates during processing."
```

**4. Address Parallel Requests**
```
"For parallel requests, we use mutex on critical section.
Only one request can create/update key at a time.
Others wait and detect duplicate."
```

**5. Mention Cache for Multi-Cluster**
```
"DB replication takes minutes.
Cache replication takes milliseconds.
Use cache for idempotency keys in geo-distributed systems."
```

### Don'ts ❌

**1. Don't Confuse with Concurrency**
```
❌ "Idempotency is when multiple users access same resource"
✓ "Idempotency handles duplicate requests from same operation"
```

**2. Don't Forget Parallel Case**
```
❌ Only explain sequential duplicates
✓ Explain both sequential and parallel scenarios
```

**3. Don't Ignore Multi-Cluster**
```
❌ Only consider single server
✓ Address geo-distributed systems with cache
```

**4. Don't Skip Key Generation**
```
❌ "Server generates key"
✓ "Client generates key (UUID) for each operation"
```

### Common Interview Questions

**Q1: "Why can't server generate the idempotency key?"**

```
Answer:
"Client must generate key because:
1. Client knows when operation is retry vs new operation
2. Same key for retries, new key for new operations
3. Server cannot distinguish without client input"
```

**Q2: "What if client sends same key for different operations?"**

```
Answer:
"This violates the agreement. Client must generate:
- New key for each NEW operation
- Same key only for RETRIES of same operation

If violated, different operations would be treated as duplicates.
We can add validation: check operation details match key."
```

**Q3: "How long should we keep idempotency keys?"**

```
Answer:
"Depends on retry window:
- Active operations: Keep until CONSUMED
- Completed operations: Keep for 24-48 hours
- After that: Can be purged (client shouldn't retry after so long)

Use TTL in cache:
- CREATED: 10-15 minutes
- CONSUMED: 24-48 hours"
```

**Q4: "What if cache fails in multi-cluster setup?"**

```
Answer:
"Fallback to DB:
1. Try cache first (fast path)
2. If cache unavailable, use DB (slow path)
3. Accept eventual consistency temporarily
4. Restore cache from DB when available

Trade-off: Temporary performance impact vs availability"
```

**Q5: "How to handle idempotency for GET requests?"**

```
Answer:
"GET is already idempotent by nature:
- No side effects on server
- Same result every time
- No need for idempotency key

Only POST needs idempotency handling."
```

### Key Points to Remember

```
1. Idempotency ≠ Concurrency
   Different problems, don't confuse

2. POST needs idempotency handling
   GET, PUT, DELETE already idempotent

3. Client generates unique key
   UUID per operation

4. Key lifecycle: CREATED → CONSUMED
   Helps detect duplicates during processing

5. Response codes matter:
   201: Created
   200: Duplicate (completed)
   409: Duplicate (processing)
   400: No key

6. Parallel requests: Use mutex
   Critical section protection

7. Multi-cluster: Use cache
   Millisecond replication vs minutes for DB
```

### Real-World Examples

**Payment System:**
```
Problem: Double payment on retry
Solution: Idempotency key per transaction
Result: Safe retries, no duplicate charges
```

**E-commerce Cart:**
```
Problem: Multiple items added on retry
Solution: Idempotency key per add-to-cart
Result: One item added, safe retries
```

**Order Creation:**
```
Problem: Duplicate orders on timeout
Solution: Idempotency key per order
Result: One order created, safe retries
```

---

**End of Lecture**

Idempotency handling is critical for production systems, especially for payment and transaction systems. Understanding the difference between idempotency and concurrency, implementing idempotency keys correctly, and handling parallel requests and multi-cluster scenarios are essential skills for system design interviews.

**Key Takeaway:** Use idempotency keys to make POST requests safe to retry. Client generates unique key, server tracks status (CREATED/CONSUMED), and uses mutex for parallel requests and cache for multi-cluster consistency.
