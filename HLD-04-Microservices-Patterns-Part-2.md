# HLD-04: Microservices Design Patterns - Part 2

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Pattern 1: Strangler Pattern](#pattern-1-strangler-pattern)
   - [When to Use](#when-to-use)
   - [How It Works](#how-it-works)
   - [Traffic Migration Strategy](#traffic-migration-strategy)
3. [Database Management in Microservices](#database-management-in-microservices)
   - [Database per Service](#database-per-service)
   - [Shared Database](#shared-database)
   - [Comparison](#comparison)
4. [Pattern 2: Saga Pattern](#pattern-2-saga-pattern)
   - [Why Saga is Needed](#why-saga-is-needed)
   - [What is Saga](#what-is-saga)
   - [Choreography-Based Saga](#choreography-based-saga)
   - [Orchestration-Based Saga](#orchestration-based-saga)
   - [Interview Question: Payment Scenario](#interview-question-payment-scenario)
5. [Pattern 3: CQRS Pattern](#pattern-3-cqrs-pattern)
   - [What is CQRS](#what-is-cqrs)
   - [How It Works](#how-it-works-1)
   - [Synchronization Strategies](#synchronization-strategies)
6. [Summary](#summary)

---

## Introduction

**Prerequisites:** Watch Part 1 (Decomposition Pattern) first

This lecture covers **three important microservices design patterns:**

1. **Strangler Pattern** ⭐⭐⭐ (Very Important)
2. **Saga Pattern** ⭐⭐⭐ (Very Important - Interview Favorite)
3. **CQRS Pattern** ⭐⭐ (Important)

**Why these patterns?**
- Strangler: Refactoring monolithic to microservices
- Saga: Managing distributed transactions
- CQRS: Handling complex queries across services

---

## Pattern 1: Strangler Pattern

### When to Use

**Use Case:** Refactoring monolithic to microservices

**Scenario:**
- You already have a monolithic service
- Need to transition to microservices
- Decomposition pattern tells you how to divide
- Strangler pattern tells you how to migrate

### How It Works

**Core Concept:** Gradually migrate traffic from monolithic to microservices using a controller

```
                    ┌─────────────────┐
                    │  Incoming API   │
                    │     Traffic     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   Controller    │
                    │                 │
                    │ Traffic Manager │
                    └────────┬────────┘
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
        ┌──────────────┐          ┌──────────────┐
        │ Microservices│          │  Monolithic  │
        │              │          │  Application │
        │ (10% traffic)│          │ (90% traffic)│
        └──────────────┘          └──────────────┘
```

**Controller's Job:**
- Manage percentage of traffic
- Decide where to send API traffic
- Route to microservices or monolithic

### Traffic Migration Strategy

**Step-by-Step Migration:**

```
Phase 1: Initial Setup (Start with one flow)
┌──────────────┐          ┌──────────────┐
│ Microservices│          │  Monolithic  │
│   10% ████   │          │ 90% ████████ │
└──────────────┘          └──────────────┘

Phase 2: Gain Confidence
┌──────────────┐          ┌──────────────┐
│ Microservices│          │  Monolithic  │
│   20% ████   │          │ 80% ████████ │
└──────────────┘          └──────────────┘

Phase 3: Increase Traffic
┌──────────────┐          ┌──────────────┐
│ Microservices│          │  Monolithic  │
│   50% ████   │          │ 50% ████████ │
└──────────────┘          └──────────────┘

Phase 4: Almost Complete
┌──────────────┐          ┌──────────────┐
│ Microservices│          │  Monolithic  │
│  100% ████   │          │  0%          │
└──────────────┘          └──────────────┘

Phase 5: Delete Monolithic
┌──────────────┐          
│ Microservices│          ❌ Monolithic deleted
│  100% ████   │          
└──────────────┘          
```

### Migration Workflow

**Example: Converting one flow at a time**

```
Step 1: Convert one flow from monolithic to microservices
        Enable that flow in microservices

Step 2: Start with 10% traffic
        - 10 APIs → Microservices
        - 90 APIs → Monolithic

Step 3: If microservices fail
        - Immediately set to 0% on microservices
        - Send 100% to monolithic
        - Fix the bug

Step 4: After bug fix, test again
        - Start with 10% traffic again
        - Monitor carefully

Step 5: Gradually increase
        10% → 20% → 50% → 100%

Step 6: As more flows are added
        Monolithic: 1000 → 900 → 500 → 200 → 100 → 0
        Microservices: 0 → 100 → 500 → 800 → 900 → 1000
```

### Key Characteristics

**Why "Strangler"?**
- Slowly **throttle** the monolithic
- Gradually **strangle** the old system
- Eventually **replace** it completely

**Important Points:**
- ❌ Never convert entire monolithic at once
- ✅ Build small parts incrementally
- ✅ Keep taking traffic gradually
- ✅ Can rollback if issues arise
- ✅ Control traffic percentage anytime

**Rollback Capability:**

```
If issue in microservices:
Controller → "Don't take traffic here, issue detected"
           → Move traffic back to monolithic
           → Fix issue
           → Resume migration
```

---

## Database Management in Microservices

Before understanding Saga and CQRS, we need to understand database strategies.

### Two Types of Database Approaches

1. **Database per Service** (Recommended)
2. **Shared Database** (Not recommended)

---

## Shared Database

### Architecture

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Service  │    │ Service  │    │ Service  │
│    1     │    │    2     │    │    3     │
└──────────┘    └──────────┘    └──────────┘
     │               │               │
     └───────────────┴───────────────┘
                     │
                     ▼
            ┌─────────────────┐
            │   Shared DB     │
            │                 │
            │ Table 1 - 10    │
            └─────────────────┘
```

### Advantages

✅ **Easy to Join Queries**
- All tables in one database
- Can easily join across tables

✅ **Easy Transaction Management**
- One database = One transaction
- ACID properties easy to maintain
- If one table fails, rollback all

**Example:**

```
Transaction across 10 tables:
┌─────────────────────────────────────┐
│  Start Transaction                  │
│    Insert into Table 1  ✓           │
│    Insert into Table 2  ✓           │
│    Insert into Table 3  ✓           │
│    ...                              │
│    Insert into Table 8  ✗ (Failed)  │
│  Rollback All                       │
└─────────────────────────────────────┘

Result: Either all succeed or none succeed
```

### Disadvantages

❌ **Cannot Scale Individual Components**

**Problem:**

```
┌──────────────────────────────────────────┐
│  Service 1: Order (High Traffic)         │
│  Service 2: Inventory (Low Traffic)      │
│  Service 3: Payment (Low Traffic)        │
└──────────────────────────────────────────┘
                │
                ▼
        ┌──────────────┐
        │  Shared DB   │
        │   (2 GB)     │
        └──────────────┘

Problem: Need to scale for Order service
         Must scale ENTIRE database
         Even though Inventory and Payment don't need it
         
Solution in Shared DB:
┌──────────────┐      ┌──────────────┐
│  Shared DB   │  →   │  Shared DB   │
│   (2 GB)     │      │   (4 GB)     │
└──────────────┘      └──────────────┘
                      (Scales everything!)
```

❌ **Cannot Make Independent Changes**

**Problem:**

```
Scenario: Service 3 needs to delete a column from Table 10

┌──────────────────────────────────────────────┐
│  Table 10 is shared by:                      │
│  - Service 1                                 │
│  - Service 2                                 │
│  - Service 3                                 │
└──────────────────────────────────────────────┘

Service 3 cannot delete column because:
- Must check all dependencies
- Service 1 might be using it
- Service 2 might be using it
- Need coordination across teams
- Slows down development
```

### When to Use Shared Database

✅ **Use when:**
- Small service with few components
- Won't scale quickly
- Simple use cases

❌ **Don't use when:**
- Need to scale independently
- Multiple teams working on different services
- High scalability requirements

---

## Database per Service

### Architecture

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Service  │         │ Service  │         │ Service  │
│    1     │         │    2     │         │    3     │
└──────────┘         └──────────┘         └──────────┘
     │                    │                    │
     ▼                    ▼                    ▼
┌──────────┐         ┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │         │   DB 3   │
│ (Tables  │         │ (Tables  │         │ (Tables  │
│  1-3)    │         │  4-6)    │         │  7-10)   │
└──────────┘         └──────────┘         └──────────┘
```

### Important Rule

**No service can query another service's database**

```
❌ Wrong:
┌──────────┐                    ┌──────────┐
│ Service  │───────────────────►│   DB 3   │
│    2     │  Direct query      │          │
└──────────┘                    └──────────┘

✅ Correct:
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Service  │────────►│ Service  │────────►│   DB 3   │
│    2     │ API call│    3     │  Query  │          │
└──────────┘         └──────────┘         └──────────┘
```

**If Service 2 needs data from DB 3:**
1. Service 2 calls Service 3's API
2. Service 3 queries its own DB 3
3. Service 3 returns data to Service 2

### Advantages

✅ **Each Service Can Use Different Database Technology**

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Service  │         │ Service  │         │ Service  │
│    1     │         │    2     │         │    3     │
└──────────┘         └──────────┘         └──────────┘
     │                    │                    │
     ▼                    ▼                    ▼
┌──────────┐         ┌──────────┐         ┌──────────┐
│   SQL    │         │ MongoDB  │         │ Postgres │
│(Relational)│       │  (NoSQL) │         │          │
└──────────┘         └──────────┘         └──────────┘

Each service chooses DB based on its needs!
```

✅ **Easy Maintenance**

```
Service 1 wants to modify DB 1:
- Only needs to worry about itself
- No impact on Service 2 or Service 3
- No need to ask other teams
- Independent development
```

✅ **Independent Scaling**

```
Scenario: DB 1 has high traffic

Before Scaling:
┌──────────┐         ┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │         │   DB 3   │
│  (2 GB)  │         │  (1 GB)  │         │  (1 GB)  │
│High Load │         │Low Load  │         │Low Load  │
└──────────┘         └──────────┘         └──────────┘

After Scaling:
┌──────────┐         ┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │         │   DB 3   │
│  (5 GB)  │         │  (1 GB)  │         │  (1 GB)  │
│ Scaled!  │         │No change │         │No change │
└──────────┘         └──────────┘         └──────────┘

Only scale what's needed!
```

### Disadvantages

❌ **Cannot Join Queries Across Databases**

**Problem:**

```
Table 6 in DB 2, Table 10 in DB 3

Need to join Table 6 and Table 10:
┌──────────┐         ┌──────────┐
│   DB 2   │    ?    │   DB 3   │
│ Table 6  │◄───────►│ Table 10 │
└──────────┘         └──────────┘

Cannot directly join across different databases!
```

**Solution:** CQRS Pattern (covered later)

❌ **Difficult Transaction Management**

**Problem:**

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │         │   DB 3   │
└──────────┘         └──────────┘         └──────────┘
     │                    │                    │
     ▼                    ▼                    ▼
┌──────────┐         ┌──────────┐         ┌──────────┐
│  Local   │         │  Local   │         │  Local   │
│Transaction│        │Transaction│        │Transaction│
└──────────┘         └──────────┘         └──────────┘

Each DB has its own local transaction
Cannot create common transaction across all three!
```

**Scenario:**

```
Order placement needs to update all three DBs:

DB 1 (Order):    Insert order row      → Success ✓
DB 2 (Inventory): Update inventory     → Success ✓
DB 3 (Payment):   Record payment       → Failed ✗

Problem: DB 1 and DB 2 committed, but DB 3 failed!
         How to rollback DB 1 and DB 2?
```

**Solution:** Saga Pattern (covered next)

---

## Pattern 2: Saga Pattern

### Why Saga is Needed

**Problem:** Distributed transactions across multiple databases

**Example Scenario:**

```
Place Order Transaction:

┌──────────┐         ┌──────────┐         ┌──────────┐
│ Order DB │         │Inventory │         │ Payment  │
│          │         │    DB    │         │    DB    │
└──────────┘         └──────────┘         └──────────┘
     │                    │                    │
     ▼                    ▼                    ▼
Insert order         Update inventory    Record payment
   row                   (Success)           (Failed!)
(Success)

Problem: Order inserted, Inventory updated, but Payment failed!
         Need to rollback Order and Inventory
```

**Traditional ACID properties work on single database:**
- Can maintain ACID on DB 1 ✓
- Can maintain ACID on DB 2 ✓
- Can maintain ACID on DB 3 ✓
- **Cannot maintain ACID across all three** ✗

**Saga solves this problem!**

### What is Saga

**Saga Definition:** Sequence of local transactions

```
Saga = Sequence of Local Transactions

Transaction 1 (Local) → Transaction 2 (Local) → Transaction 3 (Local)
```

**How Saga Works:**

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Service  │         │ Service  │         │ Service  │
│    1     │         │    2     │         │    3     │
└──────────┘         └──────────┘         └──────────┘
     │                    │                    │
     ▼                    ▼                    ▼
┌──────────┐         ┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │         │   DB 3   │
└──────────┘         └──────────┘         └──────────┘

Each service:
1. Updates its own DB (local transaction)
2. Publishes an event
3. Listens to events from other services
4. If failure → Publishes compensation event
```

### Two Types of Saga

1. **Choreography-Based Saga**
2. **Orchestration-Based Saga**

---

## Choreography-Based Saga

### Architecture

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Service  │         │ Service  │         │ Service  │
│    1     │         │    2     │         │    3     │
└──────────┘         └──────────┘         └──────────┘
     │                    │                    │
     │                    │                    │
     ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────┐
│              Event Bus / Message Queue              │
│                                                     │
│  ┌──────────────┐        ┌──────────────┐         │
│  │ Success Queue│        │ Failure Queue│         │
│  └──────────────┘        └──────────────┘         │
└─────────────────────────────────────────────────────┘
```

### How It Works

**Success Flow:**

```
Step 1: Service 1 processes and publishes event
┌──────────┐
│ Service  │  1. Update DB 1
│    1     │  2. Publish "Order Created" event
└──────────┘
     │
     ▼
┌──────────┐
│  Event   │  "Order Created"
│   Bus    │
└──────────┘
     │
     ▼
┌──────────┐
│ Service  │  1. Listen to "Order Created"
│    2     │  2. Update DB 2 (Inventory)
│          │  3. Publish "Inventory Updated" event
└──────────┘
     │
     ▼
┌──────────┐
│  Event   │  "Inventory Updated"
│   Bus    │
└──────────┘
     │
     ▼
┌──────────┐
│ Service  │  1. Listen to "Inventory Updated"
│    3     │  2. Update DB 3 (Payment)
│          │  3. Publish "Payment Recorded" event
└──────────┘
```

**Failure Flow:**

```
Step 1: Service 3 fails
┌──────────┐
│ Service  │  1. Payment recording failed
│    3     │  2. Publish "Payment Failed" event
└──────────┘
     │
     ▼
┌──────────┐
│  Event   │  "Payment Failed"
│   Bus    │  (Failure Queue)
└──────────┘
     │
     ▼
┌──────────┐
│ Service  │  1. Listen to "Payment Failed"
│    2     │  2. Rollback inventory update
│          │  3. Publish "Inventory Rollback" event
└──────────┘
     │
     ▼
┌──────────┐
│  Event   │  "Inventory Rollback"
│   Bus    │  (Failure Queue)
└──────────┘
     │
     ▼
┌──────────┐
│ Service  │  1. Listen to "Inventory Rollback"
│    1     │  2. Rollback order creation
│          │  3. Transaction fully rolled back
└──────────┘
```

### Characteristics

**How services communicate:**
- Each service listens to events
- Each service publishes events
- No central coordinator

**Event Queues:**
- **Success Queue:** For successful operations
- **Failure Queue:** For failed operations and compensations

### Disadvantage

❌ **Circular Dependency / Cycle Dependency**

```
Problem:
S1 → Event → S2
S2 → Event → S1
S1 → Event → S2
S2 → Event → S1
...
(Cycle can form)

┌──────────┐         ┌──────────┐
│ Service  │────────►│ Service  │
│    1     │◄────────│    2     │
└──────────┘         └──────────┘
      ▲                    │
      │                    │
      └────────────────────┘
        Circular dependency!
```

---

## Orchestration-Based Saga

### Architecture

```
                ┌─────────────────┐
                │  Orchestrator   │
                │   (Coordinator) │
                └─────────────────┘
                   │    │    │
        ┌──────────┘    │    └──────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│ Service  │    │ Service  │    │ Service  │
│    1     │    │    2     │    │    3     │
└──────────┘    └──────────┘    └──────────┘
     │               │               │
     ▼               ▼               ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│   DB 1   │    │   DB 2   │    │   DB 3   │
└──────────┘    └──────────┘    └──────────┘
```

### How It Works

**Success Flow:**

```
Step 1: Orchestrator calls Service 1
Orchestrator → Service 1: "Create order"
Service 1 → DB 1: Insert order
Service 1 → Orchestrator: "Success"

Step 2: Orchestrator calls Service 2
Orchestrator → Service 2: "Update inventory"
Service 2 → DB 2: Update inventory
Service 2 → Orchestrator: "Success"

Step 3: Orchestrator calls Service 3
Orchestrator → Service 3: "Record payment"
Service 3 → DB 3: Record payment
Service 3 → Orchestrator: "Success"

Result: Transaction complete!
```

**Failure Flow:**

```
Step 1: Service 1 succeeds
Orchestrator → Service 1: "Create order"
Service 1 → Orchestrator: "Success"

Step 2: Service 2 succeeds
Orchestrator → Service 2: "Update inventory"
Service 2 → Orchestrator: "Success"

Step 3: Service 3 fails
Orchestrator → Service 3: "Record payment"
Service 3 → Orchestrator: "Failed"

Step 4: Orchestrator initiates rollback
Orchestrator → Service 2: "Rollback inventory"
Service 2 → DB 2: Reverse inventory update
Service 2 → Orchestrator: "Rollback complete"

Step 5: Continue rollback
Orchestrator → Service 1: "Rollback order"
Service 1 → DB 1: Delete order
Service 1 → Orchestrator: "Rollback complete"

Result: Full rollback complete!
```

### Characteristics

**Orchestrator's responsibilities:**
- Calls services in sequence
- Manages state of the transaction
- Handles rollback if any service fails
- Central coordinator

**Advantages over Choreography:**
✅ No circular dependencies
✅ Clear transaction flow
✅ Easier to understand and debug
✅ Centralized control

---

## Interview Question: Payment Scenario

### Problem Statement

**Scenario:**
- Person A needs to pay Person B ₹10
- Using payment app (Paytm, PhonePe, GPay, etc.)
- Person A has ₹100 balance

**Microservices Setup:**

```
┌──────────────┐         ┌──────────────┐
│   Balance    │         │   Payment    │
│   Service    │         │   Service    │
│              │         │              │
│ Stores user  │         │ Records      │
│ balance      │         │ transactions │
└──────────────┘         └──────────────┘
     │                        │
     ▼                        ▼
┌──────────────┐         ┌──────────────┐
│  Balance DB  │         │  Payment DB  │
│              │         │              │
│ A: ₹100      │         │ Transaction  │
│              │         │   History    │
└──────────────┘         └──────────────┘
```

### The Challenge

**Transaction Flow:**

```
Step 1: Check balance
Request: "A pays B ₹10"
Balance Service: Check if A has ₹10
Result: Yes, A has ₹100

Step 2: Deduct balance
Balance Service: Deduct ₹10 from A
Balance DB: A = ₹100 - ₹10 = ₹90
Status: Success ✓

Step 3: Record payment
Payment Service: Record "A paid B ₹10"
Payment DB: Insert transaction record
Status: Failed ✗

Problem:
- Balance deducted (₹90)
- Payment not recorded
- No history of payment
- Money disappeared!
```

**Visual Representation:**

```
Before Transaction:
┌──────────────┐         ┌──────────────┐
│  Balance DB  │         │  Payment DB  │
│  A: ₹100     │         │  (Empty)     │
└──────────────┘         └──────────────┘

After Failed Transaction:
┌──────────────┐         ┌──────────────┐
│  Balance DB  │         │  Payment DB  │
│  A: ₹90      │         │  (Empty)     │
└──────────────┘         └──────────────┘
     ✓                        ✗
  Deducted              Not recorded

Problem: ₹10 deducted but no payment record!
```

### Interview Question

**Q: How will you resolve this issue in microservices architecture?**

### Solution: Using Saga Pattern

**Answer:**

```
Step 1: Balance Service deducts ₹10
┌──────────────┐
│   Balance    │  1. Deduct ₹10 from A
│   Service    │  2. A = ₹100 - ₹10 = ₹90
│              │  3. Publish "Balance Deducted" event
└──────────────┘
     │
     ▼
┌──────────────┐
│  Event Bus   │  Event: "Balance Deducted"
│              │  Details: Transaction ID, A→B, ₹10
└──────────────┘

Step 2: Payment Service tries to record
┌──────────────┐
│   Payment    │  1. Listen to "Balance Deducted"
│   Service    │  2. Try to record payment
│              │  3. Recording failed!
│              │  4. Publish "Payment Failed" event
└──────────────┘
     │
     ▼
┌──────────────┐
│  Event Bus   │  Event: "Payment Failed"
│              │  Details: Transaction ID, A→B, ₹10
└──────────────┘  (Failure Queue)

Step 3: Balance Service compensates
┌──────────────┐
│   Balance    │  1. Listen to "Payment Failed"
│   Service    │  2. Identify transaction ID
│              │  3. Compensation: Add ₹10 back to A
│              │  4. A = ₹90 + ₹10 = ₹100
│              │  5. Rollback complete!
└──────────────┘
```

**Final State:**

```
After Saga Compensation:
┌──────────────┐         ┌──────────────┐
│  Balance DB  │         │  Payment DB  │
│  A: ₹100     │         │  (Empty)     │
└──────────────┘         └──────────────┘
     ✓                        ✓
  Restored              No record (correct)

Result: Transaction fully rolled back
        A has ₹100 again
        No payment recorded (as expected)
```

### Complete Flow Diagram

```
┌─────────────────────────────────────────────────────┐
│           Saga Pattern - Payment Scenario           │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. Request: A pays B ₹10                           │
│     │                                               │
│     ▼                                               │
│  2. Balance Service                                 │
│     ├─ Check: A has ₹100 ✓                          │
│     ├─ Deduct: A = ₹90                              │
│     └─ Publish: "Balance Deducted" event            │
│     │                                               │
│     ▼                                               │
│  3. Payment Service                                 │
│     ├─ Listen: "Balance Deducted"                   │
│     ├─ Try: Record payment                          │
│     ├─ Result: Failed ✗                             │
│     └─ Publish: "Payment Failed" event              │
│     │                                               │
│     ▼                                               │
│  4. Balance Service (Compensation)                  │
│     ├─ Listen: "Payment Failed"                     │
│     ├─ Compensate: Add ₹10 back                     │
│     ├─ Result: A = ₹100                             │
│     └─ Rollback: Complete ✓                         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Key Points for Interview

**When asked this question, mention:**

1. **Problem Identification:**
   - Balance deducted but payment not recorded
   - Distributed transaction across two databases
   - Cannot use traditional ACID

2. **Solution: Saga Pattern:**
   - Use event-based communication
   - Balance Service publishes events
   - Payment Service listens and processes

3. **Compensation Logic:**
   - If payment fails, publish failure event
   - Balance Service listens to failure
   - Performs compensation transaction
   - Adds money back to restore state

4. **Implementation Choice:**
   - Can use Choreography (event-based)
   - Or Orchestration (coordinator-based)
   - Choreography simpler for 2 services

---

## Pattern 3: CQRS Pattern

### What is CQRS

**CQRS** = Command Query Responsibility Segregation

**Breaking it down:**
- **Command:** Create, Update, Delete
- **Query:** Select (Read)
- **Responsibility Segregation:** Separate them

```
┌─────────────────────────────────────┐
│             CQRS                    │
├─────────────────────────────────────┤
│                                     │
│  Command (Write Operations)         │
│  ├─ Create                          │
│  ├─ Update                          │
│  └─ Delete                          │
│                                     │
│  Query (Read Operations)            │
│  └─ Select                          │
│                                     │
│  Segregation: Separate databases    │
│                                     │
└─────────────────────────────────────┘
```

### Why CQRS is Needed

**Problem:** Cannot join queries across different databases

**Scenario:**

```
┌──────────┐         ┌──────────┐
│ Service  │         │ Service  │
│    1     │         │    2     │
└──────────┘         └──────────┘
     │                    │
     ▼                    ▼
┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │
│ Table 1  │         │ Table 2  │
└──────────┘         └──────────┘

Need: Join Table 1 and Table 2
Problem: Cannot join across different databases!
         Service 1 cannot query DB 2 directly
```

**CQRS Solution:** Create a separate read database

### How It Works

```
┌──────────┐         ┌──────────┐
│ Service  │         │ Service  │
│    1     │         │    2     │
└──────────┘         └──────────┘
     │                    │
     ▼                    ▼
┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │
│ (Write)  │         │ (Write)  │
│ Table 1  │         │ Table 2  │
└──────────┘         └──────────┘
     │                    │
     │    Events          │
     └────────┬───────────┘
              │
              ▼
     ┌─────────────────┐
     │   View DB       │
     │   (Read Only)   │
     │                 │
     │ Table 1 + Table 2│
     │   (Joined)      │
     └─────────────────┘
```

### Architecture

**Write Operations (Commands):**

```
Create, Update, Delete → Go to respective service DBs

┌──────────┐         ┌──────────┐
│ Service  │         │ Service  │
│    1     │         │    2     │
└──────────┘         └──────────┘
     │                    │
     ▼                    ▼
┌──────────┐         ┌──────────┐
│   DB 1   │         │   DB 2   │
│  Orders  │         │Inventory │
└──────────┘         └──────────┘

All write operations happen here
```

**Read Operations (Queries):**

```
Select, Join → Go to View DB

┌─────────────────────────────────┐
│        View DB (Read)           │
│                                 │
│  Common History / View          │
│  - Orders data (from DB 1)      │
│  - Inventory data (from DB 2)   │
│  - Can join both!               │
└─────────────────────────────────┘

All read operations happen here
Can join across tables easily
```

### Example

**Scenario: Orders and Inventory**

```
Write Operations:
┌──────────────┐         ┌──────────────┐
│  Orders DB   │         │ Inventory DB │
│  (Service 1) │         │  (Service 2) │
├──────────────┤         ├──────────────┤
│ Table 1-5    │         │ Table 6-8    │
│              │         │              │
│ Create ✓     │         │ Create ✓     │
│ Update ✓     │         │ Update ✓     │
│ Delete ✓     │         │ Delete ✓     │
└──────────────┘         └──────────────┘

Read Operations:
┌─────────────────────────────────┐
│    Common View DB (Read)        │
├─────────────────────────────────┤
│ Table 1-5 (from Orders)         │
│ Table 6-8 (from Inventory)      │
│                                 │
│ Can join Table 1 and Table 6! ✓ │
└─────────────────────────────────┘
```

### Synchronization Strategies

**Challenge:** How does View DB stay updated when write DBs change?

**Three Solutions:**

#### 1. Events

```
┌──────────────┐
│  Orders DB   │  1. Create/Update/Delete
│              │  2. Publish event
└──────────────┘
       │
       ▼
┌──────────────┐
│  Event Bus   │  Event: "Order Created"
└──────────────┘
       │
       ▼
┌──────────────┐
│  View DB     │  1. Listen to event
│              │  2. Apply same change
└──────────────┘

Same for Inventory DB
```

**Flow:**

```
Step 1: Write to Orders DB
Orders DB: INSERT INTO orders VALUES (...)
Orders DB: Publish "Order Created" event

Step 2: View DB listens
View DB: Listen to "Order Created" event
View DB: INSERT INTO orders VALUES (...)

Result: View DB synchronized!
```

#### 2. Database Triggers

```
┌──────────────┐
│  Orders DB   │
│              │
│  Trigger:    │  When INSERT/UPDATE/DELETE
│  ON INSERT → │  Automatically update View DB
│  ON UPDATE → │
│  ON DELETE → │
└──────────────┘
       │
       ▼
┌──────────────┐
│  View DB     │  Automatically updated
└──────────────┘
```

**Example:**

```sql
-- In Orders DB
CREATE TRIGGER sync_to_view
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    -- Update View DB
    INSERT INTO view_db.orders VALUES (NEW.*);
END;
```

#### 3. Stored Procedures

```
┌──────────────┐
│  Orders DB   │
│              │
│  Procedure:  │  After write operation
│  1. Write    │  Call procedure to sync
│  2. Call     │  with View DB
│     Proc     │
└──────────────┘
       │
       ▼
┌──────────────┐
│  View DB     │  Updated by procedure
└──────────────┘
```

**Example:**

```sql
-- Stored Procedure
CREATE PROCEDURE sync_order_to_view(order_data)
BEGIN
    -- Write to Orders DB
    INSERT INTO orders VALUES (order_data);
    
    -- Sync to View DB
    INSERT INTO view_db.orders VALUES (order_data);
END;
```

### CQRS Summary

**Advantages:**
✅ Can join queries across services
✅ Optimized read performance
✅ Separate scaling for reads and writes
✅ Write DBs can use different technologies

**Disadvantages:**
❌ Data duplication
❌ Synchronization complexity
❌ Eventual consistency (slight delay)

---

## Summary

### Three Patterns Covered

**1. Strangler Pattern** ⭐⭐⭐

**Purpose:** Refactoring monolithic to microservices

**Key Points:**
- Use controller to manage traffic percentage
- Start with 10% traffic to microservices
- Gradually increase: 10% → 20% → 50% → 100%
- Can rollback if issues arise
- Never convert entire monolithic at once
- Build flows incrementally

**When to use:**
- Migrating from monolithic to microservices
- Need gradual, safe migration
- Want ability to rollback

---

**2. Saga Pattern** ⭐⭐⭐

**Purpose:** Managing distributed transactions across multiple databases

**Key Points:**
- Sequence of local transactions
- Two types: Choreography and Orchestration
- Choreography: Event-based, services listen to each other
- Orchestration: Central coordinator manages flow
- Compensation events for rollback
- Solves distributed transaction problem

**When to use:**
- Database per service architecture
- Need to maintain consistency across services
- Transactions span multiple databases

**Interview Scenario:**
- Payment deduction but recording fails
- Use Saga to rollback balance deduction
- Compensation transaction restores state

---

**3. CQRS Pattern** ⭐⭐

**Purpose:** Separating read and write operations

**Key Points:**
- Command (Write): Create, Update, Delete
- Query (Read): Select
- Separate databases for read and write
- View DB for complex queries and joins
- Synchronization via events, triggers, or procedures

**When to use:**
- Need to join queries across services
- Want to optimize read performance
- Complex reporting requirements

**Synchronization methods:**
1. Events (publish/subscribe)
2. Database triggers
3. Stored procedures

---

### Comparison Table

| Pattern | Purpose | Complexity | When to Use |
|---------|---------|------------|-------------|
| **Strangler** | Monolithic → Microservices | Medium | Migration scenarios |
| **Saga** | Distributed transactions | High | Database per service |
| **CQRS** | Query optimization | Medium | Complex queries/joins |

### Key Takeaways

1. **Strangler Pattern:**
   - Essential for safe migration
   - Traffic control is key
   - Incremental approach

2. **Saga Pattern:**
   - Critical for microservices
   - Two implementation types
   - Compensation is key concept
   - Very important for interviews

3. **CQRS Pattern:**
   - Solves join problem
   - Separate read/write concerns
   - Multiple sync strategies

### Interview Preparation

**Most Important:**
- Saga Pattern (especially payment scenario)
- Strangler Pattern (migration strategy)

**Know:**
- When to use each pattern
- Trade-offs of each approach
- Real-world examples

**Be Ready to Explain:**
- How Saga handles failures
- How Strangler manages traffic
- How CQRS synchronizes data

---

**End of Lecture**

These patterns are fundamental to microservices architecture. Understanding them is crucial for designing robust distributed systems. More patterns will be covered as we solve real-world system design problems.
