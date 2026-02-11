# HLD-18: Two-Phase Locking (2PL)

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Two-Phase Locking Basics](#two-phase-locking-basics)
3. [Basic Two-Phase Locking](#basic-two-phase-locking)
4. [Problems with Basic 2PL](#problems-with-basic-2pl)
5. [Deadlock Prevention Strategies](#deadlock-prevention-strategies)
6. [Conservative Two-Phase Locking](#conservative-two-phase-locking)
7. [Strict Two-Phase Locking](#strict-two-phase-locking)
8. [Comparison of 2PL Types](#comparison-of-2pl-types)
9. [Practical Examples](#practical-examples)
10. [Summary](#summary)
11. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Two-Phase Locking (2PL)

**Prerequisite:**
- Watch HLD-17: Distributed Concurrency Control first
- Understand Optimistic vs Pessimistic locking

**Key Point:**
```
Two-Phase Locking = Type of Pessimistic Locking
Widely used in industry
```

**Coverage:**
- ✅ 2PL fundamentals (Growing & Shrinking phases)
- ✅ Basic 2PL (problems: deadlock, cascading abort)
- ✅ Conservative 2PL (deadlock-free)
- ✅ Strict 2PL / Rigorous 2PL (cascading abort-free)
- ✅ Deadlock prevention strategies

---

## Two-Phase Locking Basics

### Definition

```
Two-Phase Locking (2PL):
Protocol with TWO distinct phases:
1. Growing Phase
2. Shrinking Phase
```

---

### Phase 1: Growing Phase

```
Growing Phase:
- Transaction can ONLY ACQUIRE locks
- Cannot release any locks
- Number of locks INCREASES with time
```

**Visual:**

```
Number of Locks
      ▲
      │     ╱
      │    ╱
      │   ╱
      │  ╱
      │ ╱
      │╱
      └──────────────► Time
      
      Growing Phase
      (Acquire locks only)
```

---

### Phase 2: Shrinking Phase

```
Shrinking Phase:
- Transaction can ONLY RELEASE locks
- Cannot acquire any new locks
- Number of locks DECREASES with time
```

**Visual:**

```
Number of Locks
      ▲
      │╲
      │ ╲
      │  ╲
      │   ╲
      │    ╲
      │     ╲
      └──────────────► Time
      
      Shrinking Phase
      (Release locks only)
```

---

### Complete 2PL Graph

```
Number of Locks
      ▲
      │     ╱╲
      │    ╱  ╲
      │   ╱    ╲
      │  ╱      ╲
      │ ╱        ╲
      │╱          ╲
      └──────────────► Time
      
      │←Growing→│←Shrinking→│
      
Phase 1: Acquire locks (growing)
Phase 2: Release locks (shrinking)

NOT allowed:
Acquire → Release → Acquire → Release
(No interleaving!)
```

---

### Lock Types Recap

```
S Lock (Shared Lock):
- For READ operations
- Multiple transactions can hold S lock
- Blocks X lock

X Lock (Exclusive Lock):
- For WRITE operations
- Only ONE transaction can hold X lock
- Blocks both S and X locks
```

---

### Three Types of 2PL

```
1. Basic 2PL
   - Standard growing/shrinking phases
   - Problems: Deadlock, Cascading abort

2. Conservative 2PL (Static 2PL)
   - Acquire ALL locks at start
   - Deadlock-free
   - Problem: Low concurrency

3. Strict 2PL (Rigorous 2PL)
   - Hold ALL locks until commit/abort
   - Cascading abort-free
   - Problem: Deadlock possible
   - MOST WIDELY USED ✓
```

---

## Basic Two-Phase Locking

### Definition

```
Basic 2PL:
- Growing phase: Acquire locks gradually
- Shrinking phase: Release locks gradually
- Can release locks BEFORE transaction end
```

---

### Example 1: Transaction T1

**Timeline:**

```
Time    Action              Phase
────────────────────────────────────
T0:     BEGIN               
T1:     X lock on A         Growing
T2:     X lock on B         Growing
T3:     (Processing)        
T4:     Unlock A            Shrinking
T5:     Unlock B            Shrinking
T6:     COMMIT              
```

**Graph:**

```
Locks
  ▲
  │   X(B)
  │   ╱╲
  │  ╱  ╲
  │ ╱    ╲ Unlock A
  │╱      ╲
  │        ╲ Unlock B
  └──────────────► Time
  T1 T2    T4 T5 T6
  
  │←Growing→│←Shrinking→│
```

**Key Point:**

```
Locks released BEFORE commit
This can cause problems! (See cascading abort)
```

---

### Example 2: Transaction T2

**Timeline:**

```
Time    Action              Phase
────────────────────────────────────
T0:     BEGIN               
T1:     X lock on B         Growing
T2:     X lock on A         Growing
T3:     (Processing)        
T4:     COMMIT              Shrinking
        (All locks released)
```

**Graph:**

```
Locks
  ▲
  │   X(A)
  │   ╱│
  │  ╱ │
  │ ╱  │
  │╱   │
  │    │ All locks held
  │    │ until commit
  └──────────────► Time
  T1 T2    T4
  
  │←Growing→│←Shrinking→│
```

**Key Point:**

```
Locks released at COMMIT
All locks released together
This is safer (prevents cascading abort)
```

---

### Two Approaches in Basic 2PL

**Approach 1: Release Before Commit**

```
BEGIN
  Acquire locks gradually
  Process data
  Release locks gradually ← Before commit
COMMIT
```

**Approach 2: Release at Commit**

```
BEGIN
  Acquire locks gradually
  Process data
COMMIT ← Release all locks here
```

---

## Problems with Basic 2PL

### Problem 1: Deadlock

**Two scenarios where deadlock occurs:**

---

#### Deadlock Scenario 1: Two Resources

**Setup:**

```
Resources: A, B
Transactions: T1, T2
```

**Timeline:**

```
Time    T1                  T2
────────────────────────────────────
T0:     BEGIN               BEGIN
T1:     X lock on A         X lock on B
        SUCCESS ✓           SUCCESS ✓
T2:     Want X lock on B    Want X lock on A
        WAIT (T2 has it)    WAIT (T1 has it)
        ↓                   ↓
        DEADLOCK! Both waiting for each other
```

**Visual:**

```
┌──────────┐         ┌──────────┐
│    A     │         │    B     │
└──────────┘         └──────────┘
     │                    │
     │ X lock by T1       │ X lock by T2
     │                    │
     
T1 wants B ──────────────→ (BLOCKED by T2)
T2 wants A ←────────────── (BLOCKED by T1)

Circular wait = DEADLOCK!
```

**Detailed Flow:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     T1     │         │     T2     │
└────────────┘         └────────────┘
      │                      │
T0:   │ BEGIN                │ BEGIN
      │                      │
T1:   │ X lock on A          │ X lock on B
      │ SUCCESS ✓            │ SUCCESS ✓
      │                      │
      A: [X by T1]           B: [X by T2]
      │                      │
T2:   │ Request X on B       │ Request X on A
      │ BLOCKED              │ BLOCKED
      │ (T2 holds X on B)    │ (T1 holds X on A)
      │                      │
      │ Waiting for T2...    │ Waiting for T1...
      │                      │
      ▼                      ▼
      
      DEADLOCK!
```

---

#### Deadlock Scenario 2: Single Resource

**Setup:**

```
Resource: A
Transactions: T1, T2
Both want to upgrade S lock to X lock
```

**Timeline:**

```
Time    T1                  T2
────────────────────────────────────
T0:     BEGIN               BEGIN
T1:     S lock on A         S lock on A
        SUCCESS ✓           SUCCESS ✓
        (Both can read)     (Both can read)
T2:     Want X lock on A    Want X lock on A
        (Upgrade S→X)       (Upgrade S→X)
        WAIT (T2 has S)     WAIT (T1 has S)
        ↓                   ↓
        DEADLOCK! Both waiting for each other
```

**Visual:**

```
┌──────────────────┐
│        A         │
└──────────────────┘
         │
    ┌────┴────┐
    │         │
S lock by T1  S lock by T2
(Both reading)

T1 wants X lock ──→ Need T2 to release S
T2 wants X lock ──→ Need T1 to release S

Both holding S locks
Both want X locks
Neither will release S
DEADLOCK!
```

**Detailed Flow:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     T1     │         │     T2     │
└────────────┘         └────────────┘
      │                      │
T0:   │ BEGIN                │ BEGIN
      │                      │
T1:   │ S lock on A          │ S lock on A
      │ SUCCESS ✓            │ SUCCESS ✓
      │                      │
      A: [S by T1, S by T2]
      │                      │
      │ READ A               │ READ A
      │ Value: 10            │ Value: 10
      │                      │
T2:   │ UPDATE A             │ UPDATE A
      │ Need X lock          │ Need X lock
      │ (Upgrade S→X)        │ (Upgrade S→X)
      │                      │
      │ BLOCKED              │ BLOCKED
      │ (T2 has S lock)      │ (T1 has S lock)
      │                      │
      │ Waiting for T2...    │ Waiting for T1...
      │                      │
      ▼                      ▼
      
      DEADLOCK!
      
Lock conversion requires:
- No other locks on resource
- Both have S locks
- Neither can upgrade
```

---

### Problem 2: Cascading Abort

**Definition:**

```
Cascading Abort:
Transaction T1 releases lock before commit
Transaction T2 reads uncommitted data from T1
If T1 aborts, T2 must also abort
(T2 read dirty data)
```

---

#### Cascading Abort Example

**Setup:**

```
Resources: A, B
Initial values: A = 10, B = 5
```

**Timeline:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
T1:     X lock on A             
T2:     X lock on B             
T3:     UPDATE A: 10 → 11       
        (A = 11 in DB)          
T4:     Unlock A                
        (Released!)             
T5:                             BEGIN
T6:                             X lock on A
                                SUCCESS ✓
T7:                             READ A
                                Value: 11
                                (Processing on 11)
T8:     ABORT!                  
        (Rollback A: 11 → 10)   
T9:                             Must ABORT!
                                (Read dirty value 11)
```

**Visual:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     T1     │         │     T2     │
└────────────┘         └────────────┘
      │                      │
T1:   │ X lock on A          │
      │ A: 10                │
      │                      │
T3:   │ UPDATE A = 11        │
      │ A: 11 (uncommitted)  │
      │                      │
T4:   │ Unlock A             │
      │ (Lock released!)     │
      │                      │
T6:   │                      │ X lock on A
      │                      │ SUCCESS
      │                      │
T7:   │                      │ READ A
      │                      │ Value: 11
      │                      │ (Dirty read!)
      │                      │
T8:   │ ABORT                │
      │ Rollback: A = 10     │
      │                      │
      │                      │ Problem!
      │                      │ T2 used value 11
      │                      │ But 11 is invalid
      │                      │
T9:   │                      │ ABORT
      │                      │ (Cascading abort)
      │                      │
      ▼                      ▼

T1 abort → T2 must abort
Cascading effect!
```

**Problem:**

```
T1 released lock BEFORE commit
T2 read uncommitted data (dirty read)
T1 aborted → T2's data is invalid
T2 must also abort (cascading abort)

If T2 had updated other data:
- Those transactions may also need to abort
- Cascading chain of aborts
- Very expensive!
```

---

### Summary of Basic 2PL Problems

```
┌──────────────────┬─────────────────────────┐
│    Problem       │      Description        │
├──────────────────┼─────────────────────────┤
│ Deadlock         │ Transactions wait for   │
│                  │ each other's locks      │
│                  │ Circular dependency     │
│                  │                         │
│ Cascading Abort  │ Reading uncommitted data│
│                  │ Chain of aborts         │
│                  │ Very expensive          │
└──────────────────┴─────────────────────────┘

Both problems make Basic 2PL unsuitable for production
Need better approaches!
```

---

## Deadlock Prevention Strategies

### Strategy 1: Timeout

**Approach:**

```
Scheduler monitors transaction wait times
If transaction waits too long:
  - Assume deadlock
  - Abort transaction
```

**How It Works:**

```
T1 waiting for lock held by T2
│
├─ Wait time < Timeout → Continue waiting
│
└─ Wait time ≥ Timeout → Abort T1
                         (Assume deadlock)
```

**Example:**

```
Time    T1                  T2
────────────────────────────────────
T0:     X lock on A         X lock on B
T1:     Want X on B         (Processing...)
        WAIT...             
T2:     WAIT...             (Still processing)
T3:     WAIT...             (Still processing)
...     
T10:    TIMEOUT!            (Still processing)
        ABORT               
        
T1 aborted due to timeout
But T2 was just slow, not deadlocked!
```

**Disadvantage:**

```
False Positives:
- Transaction may just be slow
- No actual deadlock
- Unnecessary abort

Example:
T1 waits for T2's lock
T2 is doing long computation (valid)
Timeout triggers → T1 aborted
But there was NO deadlock!

Problem: Can't distinguish deadlock from slow transaction
```

---

### Strategy 2: Wait-For Graph (WFG)

**Approach:**

```
Scheduler maintains directed graph:
- Nodes = Transactions
- Edge Ti → Tj = Ti waiting for Tj to release lock
- Periodically check for cycles
- Cycle = Deadlock
```

---

#### Wait-For Graph Structure

**Graph Construction:**

```
Edge Ti → Tj means:
"Transaction Ti is waiting for Transaction Tj to release a lock"

Example:
T1 waiting for T2 → Edge: T1 → T2
T2 waiting for T3 → Edge: T2 → T3
```

**Graph Maintenance:**

```
Add edge:
- When transaction starts waiting for lock

Remove edge:
- When lock is released
- When transaction commits/aborts
```

---

#### WFG Example: No Deadlock

```
Graph:

T1 → T2 → T3 → T4 → T5

No cycle = No deadlock
```

---

#### WFG Example: Deadlock Detected

```
Graph:

    T1 → T2
    ↑     ↓
    ↑     ↓
    T5 ← T3
    ↑     ↓
    └─ T4 ←

Cycle detected:
T1 → T2 → T3 → T4 → T5 → T1

DEADLOCK!
```

**Visual:**

```
┌────┐     ┌────┐
│ T1 │────→│ T2 │
└────┘     └────┘
  ↑          │
  │          ↓
┌────┐     ┌────┐
│ T5 │←────│ T3 │
└────┘     └────┘
  ↑          │
  │          ↓
  └────────┬────┐
           │ T4 │
           └────┘

Cycle: T1→T2→T3→T4→T5→T1
Deadlock detected!
```

---

#### Victim Selection

**When deadlock detected, choose victim to abort:**

**4 Criteria:**

```
1. Amount of work done:
   - Transaction that did less work
   - Less waste if aborted

2. Time to completion:
   - Transaction close to completion
   - Don't abort (almost done)

3. Cost of rollback:
   - How many updates to rollback?
   - Choose transaction with fewer updates

4. Number of cycles:
   - Transaction in multiple cycles
   - Aborting it breaks multiple deadlocks
```

**Example:**

```
Cycle 1: T1 → T2 → T3 → T1
Cycle 2: T3 → T4 → T5 → T3

T3 is in BOTH cycles

Victim selection:
- T3 appears in 2 cycles
- Aborting T3 breaks both deadlocks
- T3 is good victim candidate
```

**Decision Matrix:**

```
┌──────┬──────┬──────┬──────┬──────┐
│Trans │ Work │ Time │ Cost │Cycles│ Score
│      │ Done │ Left │Rollbk│      │
├──────┼──────┼──────┼──────┼──────┼──────
│ T1   │ 80%  │ 20%  │ High │  1   │  Bad
│ T2   │ 30%  │ 70%  │ Low  │  1   │ Good
│ T3   │ 50%  │ 50%  │ Med  │  2   │ Best
└──────┴──────┴──────┴──────┴──────┴──────

Choose T3 as victim:
- In 2 cycles (breaks both)
- Medium rollback cost
- Not too close to completion
```

---

#### WFG Algorithm

```
1. Maintain WFG:
   - Add edge when transaction waits
   - Remove edge when lock released

2. Periodically check for cycles:
   - Run cycle detection algorithm
   - DFS or similar

3. If cycle found:
   - Deadlock detected
   - Select victim (4 criteria)
   - Abort victim transaction

4. Victim transaction:
   - Rollback changes
   - Release all locks
   - Retry later
```

---

### Strategy 3: Timestamp-Based Detection

**Concept:**

```
Older transaction = Higher priority
Newer transaction = Lower priority

Timestamp:
- T1 starts at time 1 → Timestamp: 1 (OLD, high priority)
- T2 starts at time 5 → Timestamp: 5 (NEW, low priority)

Lower timestamp = Higher priority
```

---

#### Wait-Die Scheme

**Rule:**

```
Old transaction WAITS for new transaction
New transaction DIES (aborts) if wants lock held by old
```

**Logic:**

```
If Ti wants lock held by Tj:
  If Ti is OLDER (lower timestamp):
    → Ti WAITS for Tj
  If Ti is NEWER (higher timestamp):
    → Ti DIES (aborts)
```

**Example 1:**

```
T1 (timestamp: 1, OLD)
T2 (timestamp: 5, NEW)

Scenario: T2 has lock, T1 wants it
T1 is OLDER → T1 WAITS ✓

Timeline:
T2: X lock on A
T1: Want X on A
    T1 is older → WAIT for T2
```

**Example 2:**

```
T1 (timestamp: 1, OLD)
T2 (timestamp: 5, NEW)

Scenario: T1 has lock, T2 wants it
T2 is NEWER → T2 DIES ✗

Timeline:
T1: X lock on A
T2: Want X on A
    T2 is newer → ABORT (DIE)
```

---

#### Wound-Wait Scheme

**Rule:**

```
Old transaction WOUNDS (aborts) new transaction
New transaction WAITS for old transaction
```

**Logic:**

```
If Ti wants lock held by Tj:
  If Ti is OLDER (lower timestamp):
    → Ti WOUNDS Tj (Tj aborts)
  If Ti is NEWER (higher timestamp):
    → Ti WAITS for Tj
```

**Example 1:**

```
T1 (timestamp: 1, OLD)
T2 (timestamp: 5, NEW)

Scenario: T2 has lock, T1 wants it
T1 is OLDER → T1 WOUNDS T2 (T2 aborts)

Timeline:
T2: X lock on A
T1: Want X on A
    T1 is older → WOUND T2
    T2 ABORTS
    T1 gets lock
```

**Example 2:**

```
T1 (timestamp: 1, OLD)
T2 (timestamp: 5, NEW)

Scenario: T1 has lock, T2 wants it
T2 is NEWER → T2 WAITS ✓

Timeline:
T1: X lock on A
T2: Want X on A
    T2 is newer → WAIT for T1
```

---

#### Wait-Die vs Wound-Wait Comparison

```
┌──────────────┬─────────────┬─────────────┐
│   Scenario   │  Wait-Die   │ Wound-Wait  │
├──────────────┼─────────────┼─────────────┤
│ OLD wants    │    WAIT     │    WOUND    │
│ lock held by │             │  (NEW dies) │
│ NEW          │             │             │
│              │             │             │
│ NEW wants    │     DIE     │    WAIT     │
│ lock held by │  (NEW dies) │             │
│ OLD          │             │             │
└──────────────┴─────────────┴─────────────┘

Both prevent deadlock:
- No circular wait
- Priority-based resolution
```

**Visual Comparison:**

```
Wait-Die:
Old → New: WAIT
New → Old: DIE

Wound-Wait:
Old → New: WOUND (New dies)
New → Old: WAIT

Both schemes:
✓ Prevent deadlock
✓ Priority-based
✓ No cycles possible
```

---

## Conservative Two-Phase Locking

### Definition

```
Conservative 2PL (Static 2PL):
- Acquire ALL locks at START of transaction
- If any lock unavailable, wait
- No locks granted until ALL available
- Then process transaction
- Release locks gradually or at end
```

---

### How It Works

**Flow:**

```
1. Transaction submits all operations
2. Scheduler identifies all required locks
3. Scheduler tries to acquire ALL locks
4. If ALL successful → Grant all locks
5. If ANY fails → Grant NONE, wait
6. Process transaction
7. Release locks (gradually or at end)
```

---

### Example

**Transaction:**

```
BEGIN
  READ A
  WRITE B
  READ C
COMMIT

Required locks:
- S lock on A
- X lock on B
- S lock on C
```

**Scenario 1: All Locks Available**

```
Time    Action
────────────────────────────────────
T0:     Request S(A), X(B), S(C)
        All available ✓
        
T1:     Grant ALL locks
        S(A) ✓
        X(B) ✓
        S(C) ✓
        
T2:     Process transaction
        (All locks held)
        
T3:     Release locks
        (Gradually or at commit)
```

**Scenario 2: One Lock Unavailable**

```
Time    Action
────────────────────────────────────
T0:     Request S(A), X(B), S(C)
        S(A) available ✓
        X(B) held by T2 ✗
        S(C) available ✓
        
T1:     Grant NONE
        Wait for X(B)
        
T2:     Still waiting...
        
T3:     X(B) released by T2
        
T4:     Grant ALL locks
        S(A) ✓
        X(B) ✓
        S(C) ✓
        
T5:     Process transaction
```

---

### Graph

```
Number of Locks
      ▲
      │
      │ ████████
      │ ████████│
      │ ████████│
      │ ████████│
      │         │╲
      │         │ ╲
      │         │  ╲
      │         │   ╲
      └─────────────────► Time
      
      │←Growing→│←Shrinking→│
      
All locks acquired at START
Held during processing
Released gradually
```

---

### Deadlock Prevention

**Why No Deadlock?**

```
Transaction has ALL required locks before starting
Never waits for additional locks during execution
No circular wait possible

Example:
T1 needs: A, B
T2 needs: B, C

Conservative 2PL:
T1: Gets A, B (or waits for both)
T2: Gets B, C (or waits for both)

No deadlock:
- T1 either has both A,B or has none
- T2 either has both B,C or has none
- No partial acquisition
- No circular wait
```

**Contrast with Basic 2PL:**

```
Basic 2PL (Deadlock possible):
T1: Lock A → Want B (T2 has it) → WAIT
T2: Lock B → Want A (T1 has it) → WAIT
DEADLOCK!

Conservative 2PL (No deadlock):
T1: Want A,B → Wait until both available
T2: Want B,C → Wait until both available
No partial locks → No deadlock
```

---

### Disadvantages

**1. Low Concurrency**

```
Problem:
Transaction holds ALL locks for entire duration
Even if not using all resources immediately

Example:
Transaction: READ A, sleep(10s), WRITE B

Conservative 2PL:
- Lock A and B at start
- Hold both for 10+ seconds
- Low concurrency

Basic 2PL:
- Lock A, read, release A
- Sleep 10s
- Lock B, write, release B
- Higher concurrency
```

**2. Scheduler Overhead**

```
Scheduler must:
- Analyze transaction beforehand
- Identify all required locks
- Track lock availability
- Coordinate lock acquisition

Extra work compared to on-demand locking
```

**3. Requires Advance Knowledge**

```
Must know ALL operations before starting
Not suitable for:
- Interactive transactions
- Dynamic queries
- Conditional operations
```

---

### Summary

```
Conservative 2PL:

Pros:
✓ Deadlock-free
✓ Simple to reason about
✓ Predictable behavior

Cons:
✗ Low concurrency
✗ Scheduler overhead
✗ Requires advance knowledge
✗ Cascading abort still possible

Use case:
- Batch processing
- Known workloads
- When deadlock prevention is critical
```

---

## Strict Two-Phase Locking

### Definition

```
Strict 2PL (Rigorous 2PL):
- Acquire locks gradually (growing phase)
- Hold ALL locks until COMMIT/ABORT
- Release ALL locks at transaction end
- No early release

Also called: Rigorous Two-Phase Locking
MOST WIDELY USED in industry ✓
```

---

### How It Works

**Flow:**

```
1. BEGIN transaction
2. Acquire locks as needed (gradually)
3. Process operations
4. Hold ALL locks until end
5. COMMIT or ABORT
6. Release ALL locks together
```

---

### Graph

```
Number of Locks
      ▲
      │     ╱│
      │    ╱ │
      │   ╱  │
      │  ╱   │
      │ ╱    │
      │╱     │ All locks held
      │      │ until commit
      └──────────────► Time
      
      │←Growing→│←Shrinking→│
                 (At commit only)
      
Growing: Acquire locks gradually
Shrinking: Release ALL at commit
```

---

### Example

**Transaction:**

```
BEGIN
  READ A
  WRITE B
  READ C
COMMIT
```

**Timeline:**

```
Time    Action              Locks Held
────────────────────────────────────────
T0:     BEGIN               None
T1:     READ A              S(A)
        Acquire S(A)        
T2:     WRITE B             S(A), X(B)
        Acquire X(B)        
T3:     READ C              S(A), X(B), S(C)
        Acquire S(C)        
T4:     Processing...       S(A), X(B), S(C)
        (All locks held)    
T5:     COMMIT              None
        Release ALL locks   
```

**Visual:**

```
┌────────────────────────────────────┐
│          Transaction               │
├────────────────────────────────────┤
│ T0: BEGIN                          │
│                                    │
│ T1: S lock on A ─────┐             │
│                      │             │
│ T2: X lock on B ─────┤             │
│                      │             │
│ T3: S lock on C ─────┤             │
│                      │             │
│ T4: Processing       │             │
│     (All held) ──────┤             │
│                      │             │
│ T5: COMMIT           │             │
│     Release all ←────┘             │
└────────────────────────────────────┘

All locks held until COMMIT
No early release
```

---

### Cascading Abort Prevention

**How Strict 2PL Prevents Cascading Abort:**

```
Problem in Basic 2PL:
T1: Update A, Unlock A (before commit)
T2: Read A (dirty read)
T1: Abort → T2 must abort (cascading)

Solution in Strict 2PL:
T1: Update A, Hold lock until commit
T2: Want to read A → BLOCKED
T1: Commit → Release lock
T2: Read A (committed data)
No dirty read, no cascading abort ✓
```

---

#### Example: No Cascading Abort

**Setup:**

```
Resource: A
Initial value: A = 10
```

**Timeline:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
T1:     X lock on A             
        A = 10                  
T2:     UPDATE A: 10 → 11       
        (A = 11, uncommitted)   
        Hold X lock ✓           
T3:                             BEGIN
T4:                             Want X lock on A
                                BLOCKED
                                (T1 holds X lock)
T5:     Processing...           WAITING...
T6:     COMMIT                  
        Release X lock          
T7:                             Acquire X lock
                                Read A = 11 ✓
                                (Committed data)
```

**Visual:**

```
┌────────────┐         ┌────────────┐
│Transaction │         │Transaction │
│     T1     │         │     T2     │
└────────────┘         └────────────┘
      │                      │
T1:   │ X lock on A          │
      │ A: 10                │
      │                      │
T2:   │ UPDATE A = 11        │
      │ Hold X lock ✓        │
      │                      │
T4:   │                      │ Want X lock on A
      │                      │ BLOCKED
      │                      │ (T1 holds it)
      │                      │
T5:   │ Processing...        │ WAITING...
      │ (Still holds X)      │
      │                      │
T6:   │ COMMIT               │
      │ Release X lock       │
      │                      │
T7:   │                      │ Acquire X lock
      │                      │ Read A = 11
      │                      │ (Committed ✓)
      │                      │
      ▼                      ▼

T2 reads COMMITTED data only
No dirty read
No cascading abort ✓
```

**If T1 Aborts:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
T1:     X lock on A             
T2:     UPDATE A: 10 → 11       
        Hold X lock ✓           
T3:                             BEGIN
T4:                             Want X lock on A
                                BLOCKED
T5:     ABORT                   
        Rollback: A = 10        
        Release X lock          
T6:                             Acquire X lock
                                Read A = 10 ✓
                                (Correct value)

T2 never saw uncommitted value
No cascading abort needed ✓
```

---

### Deadlock Still Possible

**Strict 2PL does NOT prevent deadlock:**

```
Example:
T1: Lock A → Want B
T2: Lock B → Want A
DEADLOCK!

Strict 2PL only prevents cascading abort
Still need deadlock detection/prevention
(Use WFG or timestamp-based schemes)
```

---

### Why Most Widely Used?

```
Advantages:
✓ Prevents cascading abort (major problem)
✓ Higher concurrency than Conservative 2PL
✓ No advance knowledge required
✓ Gradual lock acquisition

Disadvantages:
✗ Deadlock possible (but manageable with WFG)

Trade-off:
Cascading abort is very expensive
Deadlock is manageable with detection
→ Strict 2PL is best choice for most systems
```

---

## Comparison of 2PL Types

### Summary Table

```
┌──────────────┬─────────┬──────────────┬─────────┐
│   2PL Type   │Deadlock │  Cascading   │Concurr- │
│              │         │    Abort     │  ency   │
├──────────────┼─────────┼──────────────┼─────────┤
│ Basic        │   ✗     │      ✗       │  High   │
│              │Possible │   Possible   │         │
│              │         │              │         │
│ Conservative │   ✓     │      ✗       │  Low    │
│ (Static)     │   Free  │   Possible   │         │
│              │         │              │         │
│ Strict       │   ✗     │      ✓       │ Medium  │
│ (Rigorous)   │Possible │     Free     │         │
└──────────────┴─────────┴──────────────┴─────────┘

✓ = Problem solved
✗ = Problem exists
```

---

### Detailed Comparison

```
┌──────────────────────────────────────────────────┐
│              BASIC 2PL                           │
├──────────────────────────────────────────────────┤
│ Growing:   Acquire locks gradually               │
│ Shrinking: Release locks gradually (before end)  │
│                                                  │
│ Problems:                                        │
│ ✗ Deadlock possible                              │
│ ✗ Cascading abort possible                       │
│                                                  │
│ Concurrency: HIGH                                │
│ Use case: NOT RECOMMENDED (too many problems)    │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│          CONSERVATIVE 2PL (Static)               │
├──────────────────────────────────────────────────┤
│ Growing:   Acquire ALL locks at START            │
│ Shrinking: Release locks gradually or at end     │
│                                                  │
│ Problems:                                        │
│ ✓ Deadlock-free (all locks at start)             │
│ ✗ Cascading abort possible                       │
│                                                  │
│ Concurrency: LOW                                 │
│ Use case: Batch processing, known workloads      │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│       STRICT 2PL (Rigorous) ★ MOST USED          │
├──────────────────────────────────────────────────┤
│ Growing:   Acquire locks gradually               │
│ Shrinking: Release ALL locks at COMMIT/ABORT     │
│                                                  │
│ Problems:                                        │
│ ✗ Deadlock possible (use WFG to detect)          │
│ ✓ Cascading abort-free (locks held until end)    │
│                                                  │
│ Concurrency: MEDIUM                              │
│ Use case: PRODUCTION SYSTEMS (industry standard) │
└──────────────────────────────────────────────────┘
```

---

### When to Use Each

```
Basic 2PL:
❌ Don't use (too many problems)

Conservative 2PL:
✓ Batch processing
✓ Known workloads
✓ Deadlock prevention critical
✓ Can tolerate low concurrency

Strict 2PL:
✓ Production systems ★
✓ OLTP workloads
✓ General-purpose databases
✓ When cascading abort is expensive
✓ Can handle deadlock with WFG
```

---

## Practical Examples

### Example 1: Money Transfer

**Scenario:**

```
Transfer ₹10 from A to B
T1: Transfer ₹10 (A → B)
T2: Calculate sum (A + B)

Initial:
A = ₹100
B = ₹100
```

---

#### Basic 2PL (Problem!)

**Timeline:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
T1:     X lock on A             
        A: ₹100                 
T2:     UPDATE A: 100 → 90      
        (Debit ₹10)             
T3:     Unlock A ✓              
        (Released early!)       
T4:                             BEGIN
T5:                             S lock on A
                                Read A = ₹90
T6:                             Unlock A
T7:                             S lock on B
                                Read B = ₹100
T8:                             Unlock B
                                Sum = ₹190 ✗
T9:                             COMMIT
T10:    X lock on B             
        B: ₹100                 
T11:    UPDATE B: 100 → 110     
        (Credit ₹10)            
T12:    Unlock B                
T13:    COMMIT                  

Problem:
T2 calculated sum = ₹190
But should be ₹200!
T2 read inconsistent state
```

**Visual:**

```
Database State:

T0:  A: ₹100, B: ₹100  (Total: ₹200)
T2:  A: ₹90,  B: ₹100  (T1 updated A)
T8:  Sum = ₹190 ✗      (T2 calculated)
T13: A: ₹90,  B: ₹110  (Total: ₹200)

T2 saw intermediate state!
Inconsistent read
```

---

#### Conservative 2PL (Works!)

**Timeline:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
        Request X(A), X(B)      
        Both available ✓        
T1:     Grant X(A), X(B)        
        (All locks at start)    
T2:     UPDATE A: 100 → 90      
T3:     UPDATE B: 100 → 110     
T4:                             BEGIN
                                Request S(A), S(B)
                                BLOCKED
                                (T1 has X locks)
T5:     Processing...           WAITING...
T6:     Unlock A                
T7:     Unlock B                
T8:                             Grant S(A), S(B)
                                Read A = ₹90
                                Read B = ₹110
                                Sum = ₹200 ✓
T9:     COMMIT                  COMMIT

Correct!
T2 waited for T1 to complete
Read consistent state
```

---

#### Strict 2PL (Works!)

**Timeline:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
T1:     X lock on A             
        UPDATE A: 100 → 90      
        Hold X lock ✓           
T2:                             BEGIN
T3:                             Want S lock on A
                                BLOCKED
                                (T1 has X lock)
T4:     X lock on B             
        UPDATE B: 100 → 110     
        Hold X lock ✓           
T5:     Processing...           WAITING...
T6:     COMMIT                  
        Release X(A), X(B)      
T7:                             Acquire S(A), S(B)
                                Read A = ₹90
                                Read B = ₹110
                                Sum = ₹200 ✓
T8:                             COMMIT

Correct!
T2 waited for T1 to commit
Read consistent, committed state
```

---

### Example 2: Cascading Abort

**Scenario:**

```
T1: Update A, Update B
T2: Read A, process

Initial: A = 10, B = 5
```

---

#### Basic 2PL (Cascading Abort!)

**Timeline:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
T1:     X lock on A             
        UPDATE A: 10 → 11       
T2:     Unlock A ✓              
        (Released early!)       
T3:                             BEGIN
T4:                             X lock on A
                                Read A = 11
                                (Dirty read!)
T5:                             Processing on 11...
T6:     X lock on B             
        UPDATE B: 5 → 6         
T7:     ABORT!                  
        Rollback: A = 10, B = 5 
T8:                             Must ABORT!
                                (Used dirty value 11)

Cascading abort:
T1 abort → T2 must abort
T2 read uncommitted data
```

---

#### Strict 2PL (No Cascading Abort!)

**Timeline:**

```
Time    T1                      T2
──────────────────────────────────────────
T0:     BEGIN                   
T1:     X lock on A             
        UPDATE A: 10 → 11       
        Hold X lock ✓           
T2:                             BEGIN
T3:                             Want X lock on A
                                BLOCKED
T4:     X lock on B             
        UPDATE B: 5 → 6         
        Hold X lock ✓           
T5:                             WAITING...
T6:     ABORT!                  
        Rollback: A = 10, B = 5 
        Release locks           
T7:                             Acquire X lock on A
                                Read A = 10 ✓
                                (Committed value)

No cascading abort:
T2 never saw uncommitted value
Read correct value after T1 abort
```

---

## Summary

### Two-Phase Locking Fundamentals

```
Two Phases:
1. Growing Phase
   - Acquire locks only
   - Cannot release

2. Shrinking Phase
   - Release locks only
   - Cannot acquire

Lock Types:
- S Lock (Shared): Read, multiple allowed
- X Lock (Exclusive): Write, only one allowed
```

---

### Three Types of 2PL

```
1. Basic 2PL:
   ✗ Deadlock possible
   ✗ Cascading abort possible
   ✓ High concurrency
   ❌ NOT RECOMMENDED

2. Conservative 2PL (Static):
   ✓ Deadlock-free (all locks at start)
   ✗ Cascading abort possible
   ✗ Low concurrency
   ✓ Use for batch processing

3. Strict 2PL (Rigorous):
   ✗ Deadlock possible (use WFG)
   ✓ Cascading abort-free (locks until commit)
   ✓ Medium concurrency
   ★ MOST WIDELY USED
```

---

### Deadlock Prevention

```
1. Timeout:
   - Abort if waiting too long
   - Problem: False positives

2. Wait-For Graph (WFG):
   - Detect cycles
   - Select victim (4 criteria)
   - Best approach ✓

3. Timestamp-Based:
   - Wait-Die: Old waits, new dies
   - Wound-Wait: Old wounds, new waits

4. Conservative 2PL:
   - All locks at start
   - No deadlock possible
```

---

### Industry Standard

```
Strict 2PL + WFG:
- Strict 2PL prevents cascading abort
- WFG detects and resolves deadlocks
- Best balance of concurrency and safety
- Used in most production databases
```

---

## Interview Tips

### Common Questions

**Q1: "What is Two-Phase Locking?"**

```
Answer:
"Two-Phase Locking is a concurrency control protocol with two phases:

1. Growing Phase:
   - Transaction acquires locks
   - Cannot release any locks
   - Number of locks increases

2. Shrinking Phase:
   - Transaction releases locks
   - Cannot acquire new locks
   - Number of locks decreases

Key rule: Once you release ANY lock, you enter shrinking phase
and cannot acquire new locks.

This ensures serializability of transactions."
```

**Q2: "What are the types of 2PL and which is most used?"**

```
Answer:
"Three types:

1. Basic 2PL:
   - Can release locks before commit
   - Problems: Deadlock, cascading abort
   - Not recommended for production

2. Conservative 2PL (Static):
   - Acquire ALL locks at start
   - Deadlock-free
   - Problem: Low concurrency
   - Use: Batch processing

3. Strict 2PL (Rigorous):
   - Hold ALL locks until commit/abort
   - Prevents cascading abort
   - Deadlock possible (use WFG)
   - MOST WIDELY USED ★

Strict 2PL is industry standard because:
- Prevents cascading abort (very expensive)
- Deadlock manageable with detection
- Good concurrency balance"
```

**Q3: "How do you prevent deadlock in 2PL?"**

```
Answer:
"Four main strategies:

1. Timeout (Simple but imperfect):
   - Abort transaction if waiting too long
   - Problem: May abort valid slow transactions

2. Wait-For Graph (Best approach):
   - Maintain directed graph of waiting transactions
   - Periodically check for cycles
   - Cycle = Deadlock
   - Select victim based on:
     * Work done
     * Time to completion
     * Rollback cost
     * Number of cycles involved

3. Timestamp-Based:
   - Wait-Die: Old waits, new aborts
   - Wound-Wait: Old aborts new, new waits
   - Priority-based, no cycles

4. Conservative 2PL:
   - Acquire all locks at start
   - No partial acquisition
   - Deadlock impossible

In practice: Strict 2PL + WFG most common"
```

**Q4: "What is cascading abort and how to prevent it?"**

```
Answer:
"Cascading Abort:
When one transaction abort forces other transactions to abort
because they read uncommitted data.

Example:
T1: Update A = 11, Unlock A (before commit)
T2: Read A = 11 (dirty read)
T1: Abort → A reverts to 10
T2: Must abort (used invalid value 11)

If T2 updated other data:
- Those transactions may also abort
- Cascading chain of aborts
- Very expensive!

Prevention:
Use Strict 2PL:
- Hold ALL locks until commit/abort
- T2 cannot read A until T1 commits
- No dirty reads
- No cascading aborts

Strict 2PL ensures:
- Only committed data is read
- No cascading abort possible"
```

**Q5: "Why is Strict 2PL most widely used?"**

```
Answer:
"Strict 2PL is industry standard because:

1. Prevents Cascading Abort:
   - Holds locks until commit
   - No dirty reads
   - Cascading abort is very expensive to handle

2. Good Concurrency:
   - Better than Conservative 2PL
   - Gradual lock acquisition
   - Only holds when needed

3. Manageable Deadlock:
   - Deadlock possible but detectable
   - Use Wait-For Graph
   - Select victim intelligently
   - Deadlock less expensive than cascading abort

4. No Advance Knowledge:
   - Don't need to know all operations upfront
   - Suitable for interactive transactions
   - Flexible

Trade-off:
- Cascading abort: Very expensive, hard to handle
- Deadlock: Manageable with detection

Therefore Strict 2PL + WFG is optimal for most systems."
```

**Q6: "Explain Wait-For Graph deadlock detection"**

```
Answer:
"Wait-For Graph (WFG):

Structure:
- Nodes = Transactions
- Edge Ti → Tj = Ti waiting for Tj's lock

Algorithm:
1. Add edge when transaction waits
2. Remove edge when lock released
3. Periodically check for cycles
4. Cycle = Deadlock detected

Example:
T1 → T2 → T3 → T1 (Cycle = Deadlock)

Victim Selection (4 criteria):
1. Work done: Choose transaction with less work
2. Time left: Don't abort near-complete transactions
3. Rollback cost: Choose fewer updates
4. Cycle count: Choose transaction in multiple cycles

Advantage:
- Accurate detection
- No false positives
- Intelligent victim selection

Used with Strict 2PL in production systems."
```

### Key Points to Remember

```
1. Two phases: Growing (acquire), Shrinking (release)
   - Once you release, cannot acquire

2. Three types:
   - Basic: Both problems
   - Conservative: Deadlock-free, low concurrency
   - Strict: Cascading abort-free, MOST USED ★

3. Deadlock prevention:
   - WFG (best)
   - Timestamp-based
   - Conservative 2PL
   - Timeout (simple)

4. Cascading abort prevention:
   - Strict 2PL (hold until commit)

5. Industry standard:
   - Strict 2PL + WFG
   - Best balance
```

### Do's ✅

**1. Explain with Examples**
```
"Strict 2PL prevents cascading abort:
T1 updates A, holds lock until commit
T2 cannot read A until T1 commits
No dirty reads"
```

**2. Mention Trade-offs**
```
"Conservative 2PL: No deadlock but low concurrency
Strict 2PL: Cascading abort-free but deadlock possible"
```

**3. Discuss Industry Practice**
```
"Strict 2PL most widely used because
cascading abort is more expensive than deadlock"
```

### Don'ts ❌

**1. Don't Confuse Types**
```
❌ "Conservative 2PL prevents cascading abort"
✓ "Strict 2PL prevents cascading abort"
```

**2. Don't Forget Deadlock**
```
❌ "Strict 2PL solves all problems"
✓ "Strict 2PL prevents cascading abort but deadlock possible"
```

**3. Don't Ignore WFG**
```
❌ "Use timeout for deadlock"
✓ "Use WFG for accurate deadlock detection"
```

---

**End of Lecture**

Two-Phase Locking is a pessimistic concurrency control protocol with growing and shrinking phases. Three types: Basic (both problems), Conservative (deadlock-free, low concurrency), and Strict/Rigorous (cascading abort-free, most widely used). Deadlock prevention uses WFG, timestamp-based schemes, or Conservative 2PL. Industry standard: Strict 2PL + WFG for optimal balance.

**Key Takeaway:** Strict 2PL is the industry standard because it prevents cascading abort (very expensive) while deadlock is manageable with Wait-For Graph detection. Hold all locks until commit/abort to ensure only committed data is read!
