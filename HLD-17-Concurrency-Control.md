# HLD-17: Concurrency Control (Distributed)

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [The Concurrency Problem](#the-concurrency-problem)
3. [Why Synchronized Doesn't Work](#why-synchronized-doesnt-work)
4. [Prerequisites](#prerequisites)
5. [Transactions](#transactions)
6. [Database Locking](#database-locking)
7. [Isolation Levels](#isolation-levels)
8. [Optimistic Concurrency Control](#optimistic-concurrency-control)
9. [Pessimistic Concurrency Control](#pessimistic-concurrency-control)
10. [Comparison](#comparison)
11. [Summary](#summary)
12. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Distributed Concurrency Control

**Asked In:**
- High-Level Design: "Explain distributed concurrency control"
- Low-Level Design: "How will you handle concurrency?" (BookMyShow, Parking Lot)

**Importance:**
- Critical for distributed systems
- Handles race conditions
- Prevents data inconsistency

**Coverage:**
- ✅ Concurrency problem
- ✅ Transactions & DB locking
- ✅ Isolation levels
- ✅ Optimistic concurrency control
- ✅ Pessimistic concurrency control

---

## The Concurrency Problem

### Scenario: Movie Seat Booking

```
3 users trying to book the same seat simultaneously

┌─────────┐  ┌─────────┐  ┌─────────┐
│ User 1  │  │ User 2  │  │ User 3  │
└─────────┘  └─────────┘  └─────────┘
     │            │            │
     └────────────┼────────────┘
                  ▼
         ┌─────────────────┐
         │  Seat ID: 10    │
         │  Status: FREE   │
         └─────────────────┘
```

---

### Critical Section

```
Critical Section = Code accessing shared resource

┌──────────────────────────────────┐
│     CRITICAL SECTION             │
├──────────────────────────────────┤
│ 1. Read seat (ID = 10)           │
│ 2. If status == FREE:            │
│    3. Update status = BOOKED     │
│    4. Return SUCCESS             │
└──────────────────────────────────┘

Shared Resource: Seat ID 10
```

---

### Problem Flow

```
Timeline:

T1: All 3 users read seat
    User 1: Read → Status = FREE
    User 2: Read → Status = FREE
    User 3: Read → Status = FREE

T2: All check if FREE
    User 1: FREE? YES
    User 2: FREE? YES
    User 3: FREE? YES

T3: All update to BOOKED
    User 1: Update → BOOKED ✓
    User 2: Update → BOOKED ✓
    User 3: Update → BOOKED ✓

T4: All get SUCCESS
    User 1: SUCCESS
    User 2: SUCCESS
    User 3: SUCCESS

PROBLEM: Same seat booked by 3 users! ✗
```

**Visual:**

```
Database State:

Initial:
┌─────┬────────┐
│ ID  │ Status │
├─────┼────────┤
│ 10  │  FREE  │
└─────┴────────┘

After concurrent updates:
┌─────┬────────┐
│ ID  │ Status │
├─────┼────────┤
│ 10  │ BOOKED │ ← All 3 users think they booked it!
└─────┴────────┘

Expected: Only 1 user should succeed
Actual: All 3 users succeeded
RACE CONDITION! ✗
```

---

## Why Synchronized Doesn't Work

### Single Process (Synchronized Works)

```
Process with multiple threads:

┌─────────────────────────────────┐
│         Process 1               │
├─────────────────────────────────┤
│  Thread 1                       │
│  Thread 2                       │
│  Thread 3                       │
│                                 │
│  synchronized {                 │
│    // Critical Section          │
│    // Only 1 thread at a time   │
│  }                              │
└─────────────────────────────────┘

synchronized works ✓
Threads in same process
```

---

### Distributed System (Synchronized Fails)

```
Multiple processes across machines:

┌──────────────┐
│Load Balancer │
└──────────────┘
       │
   ┌───┴───┬───────┬───────┐
   ▼       ▼       ▼       ▼
┌──────┐┌──────┐┌──────┐┌──────┐
│ M1   ││ M2   ││ M3   ││ M4   │
│      ││      ││      ││      │
│User 1││User 2││User 3││      │
└──────┘└──────┘└──────┘└──────┘

M1, M2, M3 = Separate processes
synchronized DOESN'T work ✗

Why?
- Different processes
- Different memory spaces
- synchronized is process-local
```

**Problem:**

```
Machine 1 (Process 1):
synchronized {
  // User 1 enters
}

Machine 2 (Process 2):
synchronized {
  // User 2 enters SIMULTANEOUSLY
}

Machine 3 (Process 3):
synchronized {
  // User 3 enters SIMULTANEOUSLY
}

All 3 can enter critical section!
synchronized is per-process, not global
```

---

### Solution: Distributed Concurrency Control

```
Need global coordination mechanism:
- Optimistic Concurrency Control
- Pessimistic Concurrency Control

NOT process-local synchronization
```

---

## Prerequisites

**Before understanding concurrency control, must know:**

1. **Transactions** - What and why?
2. **Database Locking** - Shared vs Exclusive
3. **Isolation Levels** - 4 levels and problems they solve

---

## Transactions

### Definition

```
Transaction = Group of DB operations
            = All succeed OR all fail (Atomicity)
```

---

### Purpose: Achieve Integrity

**Example: Money Transfer**

```
Transfer ₹20 from A to B

Database:
┌─────┬─────────┐
│ ID  │ Balance │
├─────┼─────────┤
│ A   │  ₹100   │
│ B   │  ₹50    │
└─────┴─────────┘

Transaction:
BEGIN TRANSACTION
  1. Debit A: ₹20
  2. Credit B: ₹20
COMMIT
```

---

### With Transaction (Success Case)

```
Timeline:

T0: Initial State
    A: ₹100, B: ₹50

T1: BEGIN TRANSACTION

T2: Debit A: ₹20
    A: ₹80 ✓

T3: Credit B: ₹20
    B: ₹70 ✓

T4: COMMIT
    Changes persisted

Final State:
A: ₹80, B: ₹70
Total: ₹150 (Consistent ✓)
```

---

### With Transaction (Failure Case)

```
Timeline:

T0: Initial State
    A: ₹100, B: ₹50

T1: BEGIN TRANSACTION

T2: Debit A: ₹20
    A: ₹80 ✓

T3: Credit B: ₹20
    FAILURE! ✗

T4: ROLLBACK
    Revert all changes
    A: ₹100 (reverted)

Final State:
A: ₹100, B: ₹50
Total: ₹150 (Consistent ✓)
```

---

### Without Transaction (Problem!)

```
Timeline:

T0: Initial State
    A: ₹100, B: ₹50

T1: Debit A: ₹20
    A: ₹80 ✓

T2: Credit B: ₹20
    FAILURE! ✗

T3: No rollback mechanism!

Final State:
A: ₹80, B: ₹50
Total: ₹130 (INCONSISTENT! ✗)

₹20 disappeared!
```

---

### Transaction Summary

```
Transaction ensures:
✓ Atomicity (All or nothing)
✓ Consistency (Valid state)
✓ Rollback on failure
✓ Data integrity

Without transaction:
✗ Partial updates
✗ Inconsistent state
✗ Data loss
```

---

## Database Locking

### Purpose

```
DB Locking = Prevents other transactions from updating locked rows
```

---

### Two Types of Locks

**1. Shared Lock (S)**
**2. Exclusive Lock (X)**

---

### Shared Lock (Read Lock)

```
Shared Lock (S):
- Allows READ
- Blocks WRITE
- Multiple transactions can hold S lock

Example:

Row: [ID: 1, Status: FREE]

T1: Acquires S lock
    Can READ ✓
    Cannot WRITE ✗

T2: Can acquire S lock? YES ✓
    Can READ ✓
    Cannot WRITE ✗

T3: Can acquire S lock? YES ✓
    Can READ ✓
    Cannot WRITE ✗

Multiple S locks allowed ✓
```

**Visual:**

```
┌─────────────────┐
│  Row (ID: 1)    │
│  Status: FREE   │
└─────────────────┘
       │
   ┌───┴───┬───────┬───────┐
   ▼       ▼       ▼       ▼
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│T1(S)│ │T2(S)│ │T3(S)│ │T4(S)│
└─────┘ └─────┘ └─────┘ └─────┘

All can READ simultaneously
None can WRITE
```

---

### Exclusive Lock (Write Lock)

```
Exclusive Lock (X):
- Allows WRITE
- Blocks READ
- Blocks WRITE
- Only ONE transaction can hold X lock

Example:

Row: [ID: 1, Status: FREE]

T1: Acquires X lock
    Can READ ✓
    Can WRITE ✓

T2: Can acquire S lock? NO ✗
    Cannot READ ✗

T3: Can acquire X lock? NO ✗
    Cannot WRITE ✗

Only T1 has access
Others must wait
```

**Visual:**

```
┌─────────────────┐
│  Row (ID: 1)    │
│  Status: FREE   │
└─────────────────┘
       │
       ▼
    ┌─────┐
    │T1(X)│ ← Exclusive access
    └─────┘

┌─────┐ ┌─────┐ ┌─────┐
│T2(S)│ │T3(X)│ │T4(S)│ ← All BLOCKED
└─────┘ └─────┘ └─────┘
 WAIT    WAIT    WAIT
```

---

### Lock Compatibility Matrix

```
┌──────────────┬─────────┬─────────┐
│ Current Lock │ S Lock  │ X Lock  │
├──────────────┼─────────┼─────────┤
│ S Lock       │   ✓     │   ✗     │
│ X Lock       │   ✗     │   ✗     │
└──────────────┴─────────┴─────────┘

S + S = Compatible ✓
S + X = Incompatible ✗
X + S = Incompatible ✗
X + X = Incompatible ✗
```

**Explanation:**

```
If row has S lock:
- Another S lock? YES ✓ (Multiple reads OK)
- X lock? NO ✗ (Cannot write while reading)

If row has X lock:
- S lock? NO ✗ (Cannot read while writing)
- X lock? NO ✗ (Cannot write while writing)
```

---

## Isolation Levels

### ACID - Isolation Property

```
ACID:
- Atomicity
- Consistency
- Isolation ← Focus
- Durability

Isolation:
Multiple transactions running concurrently
Each feels like working alone
```

---

### Four Isolation Levels

```
Level 0: Read Uncommitted
Level 1: Read Committed
Level 2: Repeatable Read
Level 3: Serializable

Lower level = Higher concurrency, More problems
Higher level = Lower concurrency, Fewer problems
```

---

### Three Concurrency Problems

**1. Dirty Read**
**2. Non-Repeatable Read**
**3. Phantom Read**

---

### Problem 1: Dirty Read

**Definition:**

```
Dirty Read:
Transaction A reads data written by Transaction B
BEFORE Transaction B commits
```

**Example:**

```
Database: [ID: 1, Status: FREE]

Timeline:

T1: Transaction A begins
T2: Transaction A updates Status = BOOKED
    (NOT committed yet)
    
T3: Transaction B reads ID: 1
    Gets: Status = BOOKED
    
T4: Transaction A ROLLBACK
    Status reverted to FREE

Result:
Transaction B read "BOOKED"
But actual status is "FREE"
DIRTY READ! ✗
```

**Visual:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ BEGIN                │
      │                      │
T2:   │ UPDATE               │
      │ Status = BOOKED      │
      │ (NOT committed)      │
      │                      │
T3:   │                      │ READ
      │                      │ Status = BOOKED
      │                      │
T4:   │ ROLLBACK             │
      │ Status = FREE        │
      │                      │
      ▼                      ▼

Transaction B read uncommitted data
If A rollbacks, B has dirty data
```

---

### Problem 2: Non-Repeatable Read

**Definition:**

```
Non-Repeatable Read:
Transaction reads same row multiple times
Gets DIFFERENT values each time
```

**Example:**

```
Database: [ID: 1, Status: FREE]

Timeline:

T1: Transaction A begins
T2: Transaction A reads ID: 1
    Gets: Status = FREE
    
T3: Transaction B updates ID: 1
    Status = BOOKED
    COMMITS
    
T4: Transaction A reads ID: 1 AGAIN
    Gets: Status = BOOKED

Result:
Same transaction, same row
First read: FREE
Second read: BOOKED
NON-REPEATABLE READ! ✗
```

**Visual:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ BEGIN                │
      │                      │
T2:   │ READ                 │
      │ Status = FREE        │
      │                      │
T3:   │                      │ UPDATE
      │                      │ Status = BOOKED
      │                      │ COMMIT
      │                      │
T4:   │ READ AGAIN           │
      │ Status = BOOKED      │
      │                      │
      ▼                      ▼

Same query, different results
Within same transaction
```

---

### Problem 3: Phantom Read

**Definition:**

```
Phantom Read:
Transaction executes same query multiple times
Gets DIFFERENT number of rows each time
```

**Example:**

```
Database:
┌─────┬────────┐
│ ID  │ Status │
├─────┼────────┤
│  1  │  FREE  │
│  3  │ BOOKED │
└─────┴────────┘

Timeline:

T1: Transaction A begins
T2: Transaction A queries:
    SELECT * WHERE ID > 0 AND ID < 5
    Gets: 2 rows (ID: 1, 3)
    
T3: Transaction B inserts:
    ID: 2, Status: FREE
    COMMITS
    
T4: Transaction A queries AGAIN:
    SELECT * WHERE ID > 0 AND ID < 5
    Gets: 3 rows (ID: 1, 2, 3)

Result:
Same query, different row count
First query: 2 rows
Second query: 3 rows
PHANTOM READ! ✗
```

**Visual:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ BEGIN                │
      │                      │
T2:   │ SELECT (ID 1-5)      │
      │ Result: 2 rows       │
      │                      │
T3:   │                      │ INSERT
      │                      │ ID: 2
      │                      │ COMMIT
      │                      │
T4:   │ SELECT (ID 1-5)      │
      │ Result: 3 rows       │
      │                      │
      ▼                      ▼

Same query, different row count
New row "appeared" (phantom)
```

---

### Isolation Levels Summary Table

```
┌──────────────────┬─────────┬──────────────┬─────────┐
│ Isolation Level  │  Dirty  │Non-Repeatable│ Phantom │
│                  │  Read   │     Read     │  Read   │
├──────────────────┼─────────┼──────────────┼─────────┤
│ Read Uncommitted │   ✗     │      ✗       │    ✗    │
│ Read Committed   │   ✓     │      ✗       │    ✗    │
│ Repeatable Read  │   ✓     │      ✓       │    ✗    │
│ Serializable     │   ✓     │      ✓       │    ✓    │
└──────────────────┴─────────┴──────────────┴─────────┘

✓ = Problem solved
✗ = Problem exists
```

---

### Level 0: Read Uncommitted

**Locking Strategy:**

```
READ: No lock
WRITE: No lock

No locking at all!
```

**Problems:**

```
✗ Dirty Read (possible)
✗ Non-Repeatable Read (possible)
✗ Phantom Read (possible)
```

**Concurrency:**

```
Highest concurrency
No blocking
```

**Use Case:**

```
✓ Read-only applications
✗ Any write operations (too risky!)

Example: Analytics dashboards (approximate data OK)
```

---

### Level 1: Read Committed

**Locking Strategy:**

```
READ: S lock acquired, released immediately after read
WRITE: X lock acquired, held until transaction end
```

**Example:**

```
Transaction A:

T1: READ row
    - Acquire S lock
    - Read data
    - Release S lock immediately

T2: UPDATE row
    - Acquire X lock
    - Update data
    - Hold X lock until COMMIT/ROLLBACK
```

**Problems Solved:**

```
✓ Dirty Read (solved)
✗ Non-Repeatable Read (possible)
✗ Phantom Read (possible)
```

**How Dirty Read is Solved:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ UPDATE               │
      │ Acquire X lock       │
      │ Status = BOOKED      │
      │ Hold X lock          │
      │                      │
T2:   │                      │ READ
      │                      │ Need S lock
      │                      │ BLOCKED (X lock held)
      │                      │ WAIT...
      │                      │
T3:   │ COMMIT               │
      │ Release X lock       │
      │                      │
T4:   │                      │ READ
      │                      │ Acquire S lock
      │                      │ Status = BOOKED ✓
      │                      │
      ▼                      ▼

Transaction B can only read COMMITTED data
No dirty reads ✓
```

**Why Non-Repeatable Read Still Exists:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ READ                 │
      │ Acquire S lock       │
      │ Status = FREE        │
      │ Release S lock ✓     │
      │                      │
T2:   │                      │ UPDATE
      │                      │ Status = BOOKED
      │                      │ COMMIT
      │                      │
T3:   │ READ AGAIN           │
      │ Acquire S lock       │
      │ Status = BOOKED      │
      │ Release S lock       │
      │                      │
      ▼                      ▼

S lock released after first read
Transaction B can update
Second read gets different value
Non-Repeatable Read ✗
```

---

### Level 2: Repeatable Read

**Locking Strategy:**

```
READ: S lock acquired, held until transaction end
WRITE: X lock acquired, held until transaction end
```

**Example:**

```
Transaction A:

T1: READ row
    - Acquire S lock
    - Read data
    - Hold S lock until COMMIT/ROLLBACK

T2: UPDATE row
    - Acquire X lock
    - Update data
    - Hold X lock until COMMIT/ROLLBACK
```

**Problems Solved:**

```
✓ Dirty Read (solved)
✓ Non-Repeatable Read (solved)
✗ Phantom Read (possible)
```

**How Non-Repeatable Read is Solved:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ READ                 │
      │ Acquire S lock       │
      │ Status = FREE        │
      │ Hold S lock          │
      │                      │
T2:   │                      │ UPDATE
      │                      │ Need X lock
      │                      │ BLOCKED (S lock held)
      │                      │ WAIT...
      │                      │
T3:   │ READ AGAIN           │
      │ Status = FREE ✓      │
      │ (Same value)         │
      │                      │
T4:   │ COMMIT               │
      │ Release S lock       │
      │                      │
T5:   │                      │ UPDATE
      │                      │ Acquire X lock
      │                      │ Status = BOOKED
      │                      │
      ▼                      ▼

S lock held until commit
Transaction B cannot update
Repeatable reads ✓
```

**Why Phantom Read Still Exists:**

```
No range locking
New rows can be inserted
Same query returns different row count
```

---

### Level 3: Serializable

**Locking Strategy:**

```
READ: S lock acquired, held until transaction end
WRITE: X lock acquired, held until transaction end
RANGE: Range lock on query predicates
```

**Range Lock Example:**

```
Query: SELECT * WHERE ID >= 1 AND ID <= 3

Current rows: ID 1, ID 3

Range Lock:
- Lock ID: 1 ✓
- Lock ID: 2 (doesn't exist, but locked)
- Lock ID: 3 ✓
- Lock nearby: ID 0, ID 4

No inserts allowed in range [0-4]
```

**Problems Solved:**

```
✓ Dirty Read (solved)
✓ Non-Repeatable Read (solved)
✓ Phantom Read (solved)
```

**How Phantom Read is Solved:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ SELECT (ID 1-5)      │
      │ Acquire S locks      │
      │ + Range lock [1-5]   │
      │ Result: 2 rows       │
      │                      │
T2:   │                      │ INSERT ID: 2
      │                      │ BLOCKED (Range lock)
      │                      │ WAIT...
      │                      │
T3:   │ SELECT (ID 1-5)      │
      │ Result: 2 rows ✓     │
      │ (Same count)         │
      │                      │
T4:   │ COMMIT               │
      │ Release locks        │
      │                      │
T5:   │                      │ INSERT ID: 2
      │                      │ Success
      │                      │
      ▼                      ▼

Range lock prevents inserts
No phantom reads ✓
```

---

### Isolation Levels Comparison

```
┌──────────────────┬────────────┬────────────┬──────────┐
│ Isolation Level  │ Concurrency│  Problems  │ Use Case │
├──────────────────┼────────────┼────────────┼──────────┤
│ Read Uncommitted │  Highest   │    All 3   │ Read-only│
│ Read Committed   │    High    │     2      │ Common   │
│ Repeatable Read  │   Medium   │     1      │ Moderate │
│ Serializable     │   Lowest   │     0      │ Critical │
└──────────────────┴────────────┴────────────┴──────────┘

Trade-off: Concurrency ↔ Consistency
```

---

### Setting Isolation Level

```sql
-- Set for transaction
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION;
  -- Your queries
COMMIT;

-- Default (DB-specific)
MySQL InnoDB: REPEATABLE READ
PostgreSQL: READ COMMITTED
Oracle: READ COMMITTED
```

---

## Optimistic Concurrency Control

### Overview

```
Optimistic Concurrency Control:
- Assumes conflicts are RARE
- No locking during read
- Validates before commit
- Uses versioning

Isolation Level: Read Committed (or below)
```

---

### Version-Based Approach

**Table Structure:**

```
┌─────┬────────┬─────────┐
│ ID  │ Status │ Version │
├─────┼────────┼─────────┤
│  1  │  FREE  │    1    │
└─────┴────────┴─────────┘

Version column:
- Increments on every update
- Used for conflict detection
```

---

### Optimistic Flow

```
1. Read row (no lock, read version)
2. Perform computation
3. Before update: Validate version
4. If version matches: Update + increment version
5. If version doesn't match: Rollback, retry
```

---

### Detailed Example

**Initial State:**

```
Database:
┌─────┬────────┬─────────┐
│ ID  │ Status │ Version │
├─────┼────────┼─────────┤
│  1  │  FREE  │    1    │
└─────┴────────┴─────────┘
```

**Timeline:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ BEGIN                │ BEGIN
      │                      │
T2:   │ READ ID: 1           │ READ ID: 1
      │ Status: FREE         │ Status: FREE
      │ Version: 1           │ Version: 1
      │ (No lock)            │ (No lock)
      │                      │
T3:   │ Computation          │ Computation
      │ Change: FREE→BOOKED  │ Change: FREE→BOOKED
      │                      │
T4:   │ SELECT FOR UPDATE    │
      │ Acquire X lock       │
      │                      │
T5:   │ Version validation:  │
      │ Read version: 1      │
      │ DB version: 1        │
      │ Match! ✓             │
      │                      │
T6:   │ UPDATE               │
      │ Status = BOOKED      │
      │ Version = 2          │
      │                      │
T7:   │ COMMIT               │
      │ Release X lock       │
      │                      │
T8:   │                      │ SELECT FOR UPDATE
      │                      │ Acquire X lock
      │                      │
T9:   │                      │ Version validation:
      │                      │ Read version: 1
      │                      │ DB version: 2
      │                      │ Mismatch! ✗
      │                      │
T10:  │                      │ ROLLBACK
      │                      │ Retry...
      │                      │
      ▼                      ▼
```

**After T7 (Transaction A committed):**

```
Database:
┌─────┬────────┬─────────┐
│ ID  │ Status │ Version │
├─────┼────────┼─────────┤
│  1  │ BOOKED │    2    │
└─────┴────────┴─────────┘
```

**Transaction B:**

```
Read version: 1
DB version: 2
Conflict detected!
ROLLBACK and retry
```

---

### Optimistic Flow Diagram

```
┌─────────────────┐
│ BEGIN           │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ READ row        │
│ (No lock)       │
│ Store version   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Computation     │
│ on fetched data │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ SELECT FOR      │
│ UPDATE          │
│ (Acquire X lock)│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Version         │
│ Validation      │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐ ┌───────┐
│ Match │ │Mismatch│
└───┬───┘ └───┬───┘
    │         │
    ▼         ▼
┌───────┐ ┌───────┐
│UPDATE │ │ROLLBACK│
│Version│ │ Retry │
│+1     │ │       │
└───┬───┘ └───────┘
    │
    ▼
┌───────┐
│COMMIT │
└───────┘
```

---

### Key Points

```
1. Read without locking
   - High concurrency
   - Multiple transactions can read

2. Version check before update
   - Detects conflicts
   - Prevents lost updates

3. Rollback on conflict
   - Transaction retries
   - Eventually succeeds

4. Best for:
   - Low conflict scenarios
   - Read-heavy workloads
   - High concurrency needs
```

---

## Pessimistic Concurrency Control

### Overview

```
Pessimistic Concurrency Control:
- Assumes conflicts are COMMON
- Locks during read
- Holds locks until commit
- No version needed

Isolation Level: Repeatable Read or Serializable
```

---

### Locking Strategy

```
READ: S lock acquired, held until transaction end
WRITE: X lock acquired, held until transaction end
```

---

### Pessimistic Flow

```
1. Read row (acquire S lock, hold until end)
2. Perform computation
3. Update row (acquire X lock, hold until end)
4. Commit (release all locks)
```

---

### Detailed Example

**Initial State:**

```
Database:
┌─────┬────────┐
│ ID  │ Status │
├─────┼────────┤
│  1  │  FREE  │
└─────┴────────┘
```

**Timeline:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ BEGIN                │ BEGIN
      │                      │
T2:   │ READ ID: 1           │ READ ID: 1
      │ Acquire S lock       │ Acquire S lock
      │ Status: FREE         │ Status: FREE
      │ Hold S lock          │ Hold S lock
      │                      │
T3:   │ Computation          │ Computation
      │                      │
T4:   │ UPDATE ID: 1         │
      │ Need X lock          │
      │ Wait (B has S lock)  │
      │ BLOCKED...           │
      │                      │
T5:   │                      │ UPDATE ID: 1
      │                      │ Need X lock
      │                      │ Wait (A has S lock)
      │                      │ BLOCKED...
      │                      │
      │ ← DEADLOCK! →        │
      │                      │
T6:   │ TIMEOUT/ABORT        │ TIMEOUT/ABORT
      │ Release S lock       │ Release S lock
      │                      │
      ▼                      ▼
```

---

### Deadlock Problem

**Scenario:**

```
Transaction A: READ A, WRITE B
Transaction B: READ B, WRITE A

┌─────┐         ┌─────┐
│  A  │         │  B  │
└─────┘         └─────┘

T1: A acquires S lock on A
    B acquires S lock on B

T2: A wants X lock on B (BLOCKED by B's S lock)
    B wants X lock on A (BLOCKED by A's S lock)

DEADLOCK!
Both waiting for each other
```

**Visual:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
      │ READ A               │
      │ S lock on A          │
      │                      │ READ B
      │                      │ S lock on B
      │                      │
      │ WRITE B              │
      │ Need X lock on B     │
      │ WAIT (B has S lock)  │
      │                      │ WRITE A
      │                      │ Need X lock on A
      │                      │ WAIT (A has S lock)
      │                      │
      │ ← Waiting for B →    │
      │ ← Waiting for A →    │
      │                      │
      │    DEADLOCK!         │
      │                      │
      ▼                      ▼
```

**Resolution:**

```
Database detects deadlock
Aborts one transaction
Other transaction proceeds
Aborted transaction retries
```

---

### Pessimistic Flow Diagram

```
┌─────────────────┐
│ BEGIN           │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ READ row        │
│ Acquire S lock  │
│ Hold until end  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Computation     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ UPDATE row      │
│ Acquire X lock  │
│ Hold until end  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ COMMIT          │
│ Release all     │
│ locks           │
└─────────────────┘
```

---

### Key Points

```
1. Lock during read
   - Lower concurrency
   - Prevents conflicts

2. Hold locks until commit
   - Ensures consistency
   - Blocks other transactions

3. Deadlock risk
   - Transactions wait for each other
   - DB detects and aborts one

4. Best for:
   - High conflict scenarios
   - Write-heavy workloads
   - Critical consistency needs
```

---

## Comparison

### Optimistic vs Pessimistic

```
┌──────────────────┬─────────────┬─────────────┐
│    Feature       │ Optimistic  │ Pessimistic │
├──────────────────┼─────────────┼─────────────┤
│ Isolation Level  │ Read        │ Repeatable  │
│                  │ Committed   │ Read or     │
│                  │ (or below)  │ Serializable│
│                  │             │             │
│ Concurrency      │ High        │ Low         │
│                  │             │             │
│ Locking          │ Only during │ Throughout  │
│                  │ update      │ transaction │
│                  │             │             │
│ Conflict         │ Version     │ Locks       │
│ Detection        │ validation  │             │
│                  │             │             │
│ Deadlock         │ No          │ Yes         │
│                  │             │             │
│ Rollback         │ On version  │ On deadlock │
│                  │ mismatch    │ or timeout  │
│                  │             │             │
│ Best For         │ Low conflict│ High conflict│
│                  │ Read-heavy  │ Write-heavy │
│                  │             │             │
│ Performance      │ Better for  │ Better for  │
│                  │ reads       │ writes      │
└──────────────────┴─────────────┴─────────────┘
```

---

### When to Use

**Optimistic:**

```
✓ Low conflict probability
✓ Read-heavy workload
✓ High concurrency required
✓ Short transactions

Example: E-commerce product catalog
- Many users browsing
- Few updates
- Conflicts rare
```

**Pessimistic:**

```
✓ High conflict probability
✓ Write-heavy workload
✓ Critical consistency
✓ Long transactions

Example: Banking transactions
- Frequent updates
- Conflicts common
- Consistency critical
```

---

### Optimistic Example (No Deadlock)

```
Transaction A: READ A, WRITE B
Transaction B: READ B, WRITE A

┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     A      │         │     B      │
└────────────┘         └────────────┘
      │                      │
T1:   │ READ A               │ READ B
      │ (No lock)            │ (No lock)
      │ Version: 1           │ Version: 1
      │                      │
T2:   │ WRITE B              │
      │ X lock on B          │
      │ Version check ✓      │
      │ Update B             │
      │ COMMIT               │
      │                      │
T3:   │                      │ WRITE A
      │                      │ X lock on A
      │                      │ Version check ✓
      │                      │ Update A
      │                      │ COMMIT
      │                      │
      ▼                      ▼

No deadlock!
Both complete successfully
```

---

## Summary

### Concurrency Problem

```
Multiple requests accessing shared resource
Race conditions
Data inconsistency

Solution: Distributed Concurrency Control
```

---

### Prerequisites

```
1. Transactions
   - Atomicity (All or nothing)
   - Rollback on failure
   - Ensures integrity

2. Database Locking
   - Shared Lock (S): Read, multiple allowed
   - Exclusive Lock (X): Write, only one allowed

3. Isolation Levels
   - Read Uncommitted: No locks, all problems
   - Read Committed: Solves dirty read
   - Repeatable Read: Solves non-repeatable read
   - Serializable: Solves all (range lock)
```

---

### Optimistic Concurrency Control

```
Approach: Version-based validation
Isolation: Read Committed

Flow:
1. Read (no lock, store version)
2. Compute
3. Validate version before update
4. If match: Update + increment version
5. If mismatch: Rollback, retry

Pros:
✓ High concurrency
✓ No deadlocks
✓ Good for read-heavy

Cons:
✗ Rollback overhead on conflicts
✗ Not ideal for high conflict scenarios
```

---

### Pessimistic Concurrency Control

```
Approach: Lock-based
Isolation: Repeatable Read or Serializable

Flow:
1. Read (acquire S lock, hold)
2. Compute
3. Update (acquire X lock, hold)
4. Commit (release locks)

Pros:
✓ Strong consistency
✓ Good for write-heavy
✓ Prevents conflicts upfront

Cons:
✗ Lower concurrency
✗ Deadlock risk
✗ Longer wait times
```

---

### Decision Matrix

```
Choose Optimistic if:
- Conflicts are rare
- Read-heavy workload
- High concurrency needed
- Short transactions

Choose Pessimistic if:
- Conflicts are common
- Write-heavy workload
- Critical consistency
- Can tolerate lower concurrency
```

---

## Interview Tips

### Common Questions

**Q1: "How do you handle concurrency in distributed systems?"**

```
Answer:
"In distributed systems, synchronized keyword doesn't work because:
- Multiple processes across machines
- Different memory spaces
- synchronized is process-local

Solutions:
1. Optimistic Concurrency Control
2. Pessimistic Concurrency Control

Choice depends on:
- Conflict probability
- Read/write ratio
- Consistency requirements

Example: BookMyShow seat booking
- Low conflict (many seats)
- Use Optimistic with version validation
- Read Committed isolation level"
```

**Q2: "Explain optimistic vs pessimistic concurrency control"**

```
Answer:
"Optimistic Concurrency Control:
- Assumes conflicts are rare
- No locking during read
- Version-based validation before update
- Rollback on version mismatch
- High concurrency, no deadlocks
- Best for read-heavy, low conflict

Pessimistic Concurrency Control:
- Assumes conflicts are common
- Locks during read and write
- Holds locks until commit
- Deadlock possible
- Lower concurrency, strong consistency
- Best for write-heavy, high conflict

Example:
Optimistic: E-commerce product catalog
Pessimistic: Banking account balance updates"
```

**Q3: "What are isolation levels and why are they important?"**

```
Answer:
"Isolation levels control concurrency vs consistency trade-off.

Four levels:
1. Read Uncommitted
   - No locks
   - Highest concurrency
   - All problems exist
   - Use: Read-only analytics

2. Read Committed
   - S lock released after read
   - X lock held until commit
   - Solves dirty read
   - Use: Most applications (default)

3. Repeatable Read
   - S lock held until commit
   - X lock held until commit
   - Solves dirty + non-repeatable read
   - Use: Moderate consistency needs

4. Serializable
   - All locks + range locks
   - Lowest concurrency
   - Solves all problems
   - Use: Critical transactions

Importance:
- Determines what concurrency problems can occur
- Affects performance
- Basis for optimistic/pessimistic choice"
```

**Q4: "What is a deadlock and how do you prevent it?"**

```
Answer:
"Deadlock: Two+ transactions waiting for each other's locks

Example:
Transaction A: Lock A, wait for B
Transaction B: Lock B, wait for A
Both stuck!

Occurs in: Pessimistic concurrency control

Prevention:
1. Lock ordering
   - Always acquire locks in same order
   - A before B for all transactions

2. Timeout
   - Set lock timeout
   - Abort transaction after timeout
   - Retry

3. Deadlock detection
   - Database detects cycles
   - Aborts one transaction
   - Other proceeds

4. Use Optimistic
   - No locks during read
   - No deadlocks possible

In practice:
- Database handles detection
- Application handles retry logic
- Consider optimistic if deadlocks frequent"
```

**Q5: "How does version-based optimistic locking work?"**

```
Answer:
"Version-based approach:

1. Table has version column
   [ID, Status, Version]

2. Read phase:
   - Read row + version (no lock)
   - Store version: v1

3. Computation:
   - Process data
   - Prepare update

4. Update phase:
   - SELECT FOR UPDATE (X lock)
   - Check: DB version == stored version?
   - If YES: Update + increment version
   - If NO: Rollback, retry

Example:
T1: Read (v1) → Update (v1→v2) → Commit
T2: Read (v1) → Update (v1 != v2) → Rollback

Conflict detected via version mismatch
No lost updates
High concurrency (locks only during update)

Implementation:
- MySQL: Row versioning built-in
- Oracle: Add version column manually
- Increment on every update"
```

**Q6: "What problems do isolation levels solve?"**

```
Answer:
"Three main problems:

1. Dirty Read
   - Reading uncommitted data
   - Solved by: Read Committed+
   - Example: Read data that gets rolled back

2. Non-Repeatable Read
   - Same row, different values in same transaction
   - Solved by: Repeatable Read+
   - Example: Read twice, get different status

3. Phantom Read
   - Same query, different row count
   - Solved by: Serializable
   - Example: Query returns 2 rows, then 3 rows

Isolation levels:
- Read Uncommitted: Solves none
- Read Committed: Solves dirty read
- Repeatable Read: Solves dirty + non-repeatable
- Serializable: Solves all three

Trade-off:
Higher level = Fewer problems, Lower concurrency"
```

### Key Points to Remember

```
1. synchronized doesn't work in distributed systems
   - Process-local, not global

2. Two approaches:
   - Optimistic: Version validation
   - Pessimistic: Locking

3. Isolation levels:
   - Read Uncommitted: No locks
   - Read Committed: S lock released early
   - Repeatable Read: S lock held
   - Serializable: Range locks

4. Optimistic:
   - High concurrency
   - No deadlocks
   - Version-based
   - Read-heavy

5. Pessimistic:
   - Low concurrency
   - Deadlock risk
   - Lock-based
   - Write-heavy

6. Choose based on:
   - Conflict probability
   - Read/write ratio
   - Consistency needs
```

### Do's ✅

**1. Explain with Examples**
```
"Optimistic example: E-commerce browsing
Pessimistic example: Bank transfers"
```

**2. Mention Trade-offs**
```
"Optimistic: High concurrency but rollback overhead
Pessimistic: Strong consistency but deadlock risk"
```

**3. Discuss Isolation Levels**
```
"Optimistic uses Read Committed
Pessimistic uses Repeatable Read or Serializable"
```

### Don'ts ❌

**1. Don't Say synchronized Works**
```
❌ "Use synchronized for distributed systems"
✓ "synchronized is process-local, use optimistic/pessimistic"
```

**2. Don't Forget Deadlocks**
```
❌ "Pessimistic has no issues"
✓ "Pessimistic has deadlock risk, needs detection/timeout"
```

**3. Don't Ignore Version Column**
```
❌ "Optimistic just checks data"
✓ "Optimistic uses version column for conflict detection"
```

---

**End of Lecture**

Concurrency control is critical for distributed systems. synchronized doesn't work across processes. Use Optimistic (version-based, high concurrency, no deadlocks) for low conflict scenarios or Pessimistic (lock-based, strong consistency, deadlock risk) for high conflict scenarios. Understanding isolation levels (Read Committed, Repeatable Read, Serializable) is essential for choosing the right approach.

**Key Takeaway:** Distributed concurrency needs global coordination. Optimistic = version validation, high concurrency. Pessimistic = locking, strong consistency. Choose based on conflict probability and read/write ratio. Isolation levels determine concurrency vs consistency trade-off!
