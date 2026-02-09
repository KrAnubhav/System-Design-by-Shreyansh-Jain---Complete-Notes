# HLD-06: Consistent Hashing

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What is Hashing?](#what-is-hashing)
   - [Hash Function](#hash-function)
   - [Mod Hashing](#mod-hashing)
3. [Problem with Traditional Hashing](#problem-with-traditional-hashing)
   - [Use Case 1: Load Balancing](#use-case-1-load-balancing)
   - [Use Case 2: Horizontal Sharding](#use-case-2-horizontal-sharding)
   - [The Rebalancing Problem](#the-rebalancing-problem)
4. [What is Consistent Hashing?](#what-is-consistent-hashing)
   - [Purpose](#purpose)
   - [Goal](#goal)
5. [How Consistent Hashing Works](#how-consistent-hashing-works)
   - [Virtual Ring Concept](#virtual-ring-concept)
   - [Placing Servers on Ring](#placing-servers-on-ring)
   - [Placing Keys on Ring](#placing-keys-on-ring)
   - [Clockwise Rule](#clockwise-rule)
6. [Adding and Removing Servers](#adding-and-removing-servers)
   - [Adding a Server](#adding-a-server)
   - [Removing a Server](#removing-a-server)
7. [Problem with Basic Consistent Hashing](#problem-with-basic-consistent-hashing)
8. [Solution: Virtual Nodes](#solution-virtual-nodes)
9. [When to Use Consistent Hashing](#when-to-use-consistent-hashing)
10. [Summary](#summary)

---

## Introduction

**Prerequisites:** Watch "Scaling from Zero to Million Users" lecture (covered Horizontal Sharding)

**Topic:** Consistent Hashing

**What we'll learn:**
- How consistent hashing helps in sharding
- Where it is used
- How it solves the rebalancing problem

---

## What is Hashing?

### Hash Function

**Definition:** Converts arbitrary length input into fixed length output

```
Input (Arbitrary Length)
        │
        ▼
┌─────────────────┐
│  Hash Function  │
└─────────────────┘
        │
        ▼
Output (Fixed Length)
```

### Example

```
Input: "Shreyansh" (9 characters - arbitrary length)
       │
       ▼
Hash Function
       │
       ▼
Output: 14321 (or 20 for simplicity)
```

**Characteristics:**
- Takes arbitrary length value as input
- Produces fixed length value as output
- Same input always produces same output

---

## Mod Hashing

**Definition:** Technique to map hash values to fixed-size hash table

### How It Works

```
Hash Table Size: 6 (indices 0-5)

┌───┬───┬───┬───┬───┬───┐
│ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │
└───┴───┴───┴───┴───┴───┘

Hash Value: 20
Mod Operation: 20 % 6 = 2
Store at Index: 2
```

### Complete Flow

```
Step 1: Input key
Key: "Shreyansh"

Step 2: Apply hash function
Hash("Shreyansh") = 20

Step 3: Apply mod operation
20 % 6 = 2

Step 4: Store at index
┌───┬───┬────────────┬───┬───┬───┐
│ 0 │ 1 │ Shreyansh  │ 3 │ 4 │ 5 │
└───┴───┴────────────┴───┴───┴───┘
              ↑
           Index 2
```

**Formula:**
```
Index = Hash(Key) % Table_Size
```

---

## Problem with Traditional Hashing

### When is Hashing Fine?

**Hashing works perfectly when:**
- Size is **fixed**
- Number of servers/nodes is **constant**

**Example:**
```
Hash Table Size = 6 (Fixed)
Mod 6 operation always works ✓
```

### When Does Hashing Fail?

**Hashing fails when:**
- Size is **dynamic**
- Number of servers/nodes **changes**

---

## Use Case 1: Load Balancing

### Architecture

```
┌─────────────────┐
│  Load Balancer  │
│                 │
│  Using: Mod 3   │
└─────────────────┘
         │
    ┌────┼────┐
    │    │    │
    ▼    ▼    ▼
┌──────┐ ┌──────┐ ┌──────┐
│ App  │ │ App  │ │ App  │
│Server│ │Server│ │Server│
│  1   │ │  2   │ │  3   │
└──────┘ └──────┘ └──────┘
```

### Load Balancer's Job

**Purpose:** Equally divide traffic among app servers

**Example:**
```
10 requests → Divide into 3, 3, 4
             → No single server overloaded
```

### How Load Balancer Uses Hashing

```
Request arrives
    │
    ▼
Hash(Request_Key) % 3
    │
    ├─ Result = 0 → Server 1
    ├─ Result = 1 → Server 2
    └─ Result = 2 → Server 3
```

---

## Use Case 2: Horizontal Sharding

### Architecture

```
Original DB: 100 rows
        │
        ▼
    Sharding
        │
    ┌───┴───┐
    │       │
    ▼       ▼
┌────────┐ ┌────────┐
│  DB 1  │ │  DB 2  │
│ A - P  │ │ Q - Z  │
│50 rows │ │50 rows │
└────────┘ └────────┘
```

### Problem: Uneven Distribution

```
Reality:
┌────────┐ ┌────────┐
│  DB 1  │ │  DB 2  │
│ A - P  │ │ Q - Z  │
│80 rows │ │20 rows │
│ (Full!)│ │(Empty) │
└────────┘ └────────┘

Problem: More names start with A-P
         DB 1 fills up quickly
         DB 2 remains empty
```

**Need:** Consistent hashing to equally divide data

---

## The Rebalancing Problem

### Scenario: 3 Servers Initially

```
┌──────┐ ┌──────┐ ┌──────┐
│Node 1│ │Node 2│ │Node 3│
└──────┘ └──────┘ └──────┘

Using: Mod 3
```

### Data Distribution

```
Key: "Shreyansh"
Hash("Shreyansh") % 3 = 1
Stored in: Node 1

Key: "Rahul"
Hash("Rahul") % 3 = 2
Stored in: Node 2

Key: "Shivam"
Hash("Shivam") % 3 = 0
Stored in: Node 3
```

### Problem: Adding a Server

```
Before: 3 servers (Mod 3)
┌──────┐ ┌──────┐ ┌──────┐
│Node 1│ │Node 2│ │Node 3│
└──────┘ └──────┘ └──────┘

After: 4 servers (Mod 4)
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│Node 1│ │Node 2│ │Node 3│ │Node 4│
└──────┘ └──────┘ └──────┘ └──────┘
                            ↑
                         New node
```

**Impact:**

```
Key: "Shreyansh"
Before: Hash("Shreyansh") % 3 = 1 → Node 1 ✓
After:  Hash("Shreyansh") % 4 = 2 → Node 2 ✗

Problem: Data is in Node 1, but now looking in Node 2!
```

### Rebalancing Required

```
Step 1: "Shreyansh" data was in Node 1
Step 2: Now Mod 4 points to Node 2
Step 3: Must move data from Node 1 to Node 2

┌──────┐                    ┌──────┐
│Node 1│ ──── Move data ───►│Node 2│
└──────┘                    └──────┘
```

**Scale of Problem:**

```
Real-world scenario:
- Millions of DB entries
- All need to be rebalanced
- Very expensive operation!

Example:
1,000,000 entries × Rebalancing = Huge cost
```

### Why Traditional Hashing Fails

**Problem:**
```
Servers are DYNAMIC:
- Can add new servers
- Servers can crash
- Servers can be removed

When servers change:
- Mod value changes (Mod 3 → Mod 4)
- All hash calculations change
- Must rebalance ALL data
```

**Not Applicable For:**
- Load balancing (dynamic app servers)
- Horizontal sharding (dynamic DB servers)

---

## What is Consistent Hashing?

### Purpose

**Consistent Hashing solves:**
- Minimize rebalancing when nodes are added/removed
- Reduce cost of server changes

### Goal

**Formula:**
```
Rebalancing = (1 / n) × 100% of total keys

Where:
n = Number of active nodes
```

**Example:**

```
Scenario: 3 nodes, 1000 keys

Traditional Hashing:
- Add 1 node → Rebalance ~750 keys (75%)

Consistent Hashing:
- Add 1 node → Rebalance ~250 keys (25% = 1/4)
- Only (1/n)% of keys rebalanced!
```

### Comparison

| Approach | Nodes Change | Keys to Rebalance |
|----------|--------------|-------------------|
| **Traditional Hashing** | 3 → 4 | ~75% (750/1000) |
| **Consistent Hashing** | 3 → 4 | ~25% (250/1000) |

**Benefit:** Minimize rebalancing to acceptable levels

---

## How Consistent Hashing Works

### Virtual Ring Concept

**Key Idea:** Use a virtual ring (circular queue)

```
        0
    11  │  1
  10    │    2
9       │       3
  8     │     4
    7   │   5
        6

Virtual Ring: 0 to 11 (then back to 0)
Size: 12
```

**Characteristics:**
- Circular structure
- Fixed size (e.g., 0-11)
- Wraps around (11 → 0)

---

## Placing Servers on Ring

### Step 1: Hash Servers

**Apply hash function to servers:**

```
Server 1:
Hash(Server1) % 12 = 2
Place at position 2

Server 2:
Hash(Server2) % 12 = 6
Place at position 6

Server 3:
Hash(Server3) % 12 = 11
Place at position 11
```

### Step 2: Place on Ring

```
        0
    11  │  1
  S3    │    2 S1
9       │       3
  8     │     4
    7   │   5
        6 S2

S1 at position 2
S2 at position 6
S3 at position 11
```

---

## Placing Keys on Ring

### Step 1: Hash Keys

**Apply hash function to keys:**

```
Key1: Hash(Key1) % 12 = 1  → Position 1
Key2: Hash(Key2) % 12 = 3  → Position 3
Key3: Hash(Key3) % 12 = 5  → Position 5
Key4: Hash(Key4) % 12 = 7  → Position 7
Key5: Hash(Key5) % 12 = 8  → Position 8
Key6: Hash(Key6) % 12 = 9  → Position 9
Key7: Hash(Key7) % 12 = 4  → Position 4
```

### Step 2: Place on Ring

```
        0
    11  │  1 K1
  S3    │    2 S1
9 K6    │       3 K2
  8 K5  │     4 K7
    7 K4│   5 K3
        6 S2
```

---

## Clockwise Rule

**Rule:** Move clockwise from key position to find responsible server

### Example 1: Key1

```
        0
    11  │  1 K1
  S3    │    2 S1 ← First server clockwise
9       │       3
  8     │     4
    7   │   5
        6 S2

Key1 at position 1
Move clockwise → First server = S1
Key1 handled by Server 1
```

### Example 2: Key7

```
        0
    11  │  1
  S3    │    2 S1
9       │       3
  8     │     4 K7
    7   │   5
        6 S2 ← First server clockwise

Key7 at position 4
Move clockwise → First server = S2
Key7 handled by Server 2
```

### Complete Mapping

```
Server 1 handles:
- Key1 (position 1)
- Key2 (position 3)

Server 2 handles:
- Key7 (position 4)
- Key3 (position 5)

Server 3 handles:
- Key4 (position 7)
- Key5 (position 8)
- Key6 (position 9)
```

**Visual:**

```
        0
    11  │  1 K1 → S1
  S3    │    2 S1
9 K6→S3 │       3 K2 → S1
  8 K5→S3│     4 K7 → S2
    7 K4→S3│  5 K3 → S2
        6 S2
```

---

## Adding and Removing Servers

### Adding a Server

**Scenario:** Add Server 4 at position 5

```
Before:
        0
    11  │  1 K1
  S3    │    2 S1
9 K6    │       3 K2
  8 K5  │     4 K7
    7 K4│   5 K3
        6 S2

After:
        0
    11  │  1 K1
  S3    │    2 S1
9 K6    │       3 K2
  8 K5  │     4 K7
    7 K4│   5 S4 K3
        6 S2
```

### Impact Analysis

**Keys that DON'T change:**

```
Key1 → S1 (No change) ✓
Key2 → S1 (No change) ✓
Key6 → S3 (No change) ✓
Key5 → S3 (No change) ✓
Key4 → S3 (No change) ✓
Key3 → S3 (No change) ✓
```

**Keys that DO change:**

```
Key7:
Before: Position 4 → Clockwise → S2
After:  Position 4 → Clockwise → S4 (New!)
Must rebalance Key7 from S2 to S4
```

**Result:**

```
Total keys: 7
Keys rebalanced: 1
Percentage: 1/7 = 14% ✓

Much better than traditional hashing (would rebalance ~75%)
```

---

### Removing a Server

**Scenario:** Remove Server 2 (position 6)

```
Before:
        0
    11  │  1 K1
  S3    │    2 S1
9 K6    │       3 K2
  8 K5  │     4 K7
    7 K4│   5 K3
        6 S2 ← Remove this

After:
        0
    11  │  1 K1
  S3    │    2 S1
9 K6    │       3 K2
  8 K5  │     4 K7
    7 K4│   5 K3
        6 (Empty)
```

### Impact Analysis

**Keys that change:**

```
Key7:
Before: Position 4 → Clockwise → S2
After:  Position 4 → Clockwise → S3 (Next server)
Must rebalance Key7 from S2 to S3

Key3:
Before: Position 5 → Clockwise → S2
After:  Position 5 → Clockwise → S3 (Next server)
Must rebalance Key3 from S2 to S3
```

**Result:**

```
Total keys: 7
Keys rebalanced: 2 (Key7, Key3)
Percentage: 2/7 = 28% ✓

Only keys between removed server and next server affected
```

---

## Problem with Basic Consistent Hashing

### Uneven Distribution Problem

**Scenario:** Servers cluster together

```
        0
    11  │  1
  S3 S2 │    2
9       │       3 S1
  8     │     4
    7   │   5
        6

All servers clustered at top!
```

### Impact on Keys

```
        0
    11  │  1
  S3 S2 │    2
9       │       3 S1
  8     │     4 K1
    7 K6│   5 K2
        6 K3 K4 K5

All keys go to S1 (clockwise rule):
K1 → S1
K2 → S1
K3 → S1
K4 → S1
K5 → S1
K6 → S1

Problem: S1 overloaded!
         S2 and S3 idle!
```

**Result:**

```
Server 1: 100% load (All keys)
Server 2: 0% load
Server 3: 0% load

Failed to distribute load equally!
```

---

## Solution: Virtual Nodes

### Concept

**Virtual Nodes** = Replicate servers at multiple positions on ring

**Idea:**
- Place each server at multiple positions
- Create "virtual" copies
- Better distribution

### Implementation

**Original positions (from hash):**
```
S1 at position 2
S2 at position 6
S3 at position 11
```

**Add virtual nodes:**

```
Server 1:
- Original: Position 2
- Virtual 1: Position 8
- Virtual 2: Position 11

Server 2:
- Original: Position 6
- Virtual 1: Position 3
- Virtual 2: Position 10

Server 3:
- Original: Position 11
- Virtual 1: Position 1
- Virtual 2: Position 9
```

### Visual Representation

```
        0
    11 S1│  1 S3
  S3    │    2 S1
9 S3    │       3 S2
  8 S1  │     4
    7   │   5
        6 S2
       10 S2

Each server appears at multiple positions
Better distribution across ring
```

### Key Distribution with Virtual Nodes

```
        0
    11 S1│  1 S3 K1
  S3    │    2 S1
9 S3    │       3 S2 K2
  8 S1  │     4 K7
    7 K4│   5 K3
        6 S2 K6
       10 S2 K5

K1 → S1 (clockwise from position 1)
K2 → S1 (clockwise from position 3)
K3 → S2 (clockwise from position 5)
K6 → S1 (clockwise from position 6)
K7 → S1 (clockwise from position 4)
K4 → S3 (clockwise from position 7)
K5 → S3 (clockwise from position 10)
```

**Result:**

```
Server 1: K1, K2, K6, K7 (4 keys)
Server 2: K3 (1 key)
Server 3: K4, K5 (2 keys)

Much better distribution!
```

### How Many Virtual Nodes?

**Answer:** As many as needed to achieve target rebalancing percentage

**Formula:**
```
Target: (1/n)% rebalancing

Create enough virtual nodes to achieve this target
```

**Example:**

```
3 servers, 1000 keys
Target: ~33% rebalancing (1/3)

Start with:
- 2 virtual nodes per server → Test distribution
- 3 virtual nodes per server → Test distribution
- Continue until target achieved
```

**Trade-off:**

```
More virtual nodes:
✅ Better distribution
✅ Less rebalancing
❌ More memory (tracking virtual nodes)
❌ More computation (checking positions)

Fewer virtual nodes:
❌ Worse distribution
❌ More rebalancing
✅ Less memory
✅ Less computation
```

---

## When to Use Consistent Hashing

### Use Case 1: Load Balancing

**Scenario:**
```
┌─────────────────┐
│  Load Balancer  │
└─────────────────┘
         │
    ┌────┼────┐
    │    │    │
    ▼    ▼    ▼
┌──────┐ ┌──────┐ ┌──────┐
│ App  │ │ App  │ │ App  │
│Server│ │Server│ │Server│
└──────┘ └──────┘ └──────┘

Servers are DYNAMIC:
- Can add new servers
- Servers can crash
- Need to distribute traffic equally
```

**Why Consistent Hashing:**
- Servers change dynamically
- Minimize request redistribution
- Maintain session affinity

### Use Case 2: Horizontal Sharding

**Scenario:**
```
┌────────┐ ┌────────┐ ┌────────┐
│  DB 1  │ │  DB 2  │ │  DB 3  │
└────────┘ └────────┘ └────────┘

DBs are DYNAMIC:
- Can add new DB shards
- Can remove shards
- Need to distribute data equally
```

**Why Consistent Hashing:**
- DB nodes change dynamically
- Minimize data migration
- Equal data distribution

### When NOT to Use

**Static Scenarios:**

```
Fixed 3 servers (never changes):
┌──────┐ ┌──────┐ ┌──────┐
│Node 1│ │Node 2│ │Node 3│
└──────┘ └──────┘ └──────┘

Use traditional hashing:
- Simpler implementation
- No need for virtual ring
- Mod 3 always works
```

**Rule:**
```
Dynamic nodes → Use Consistent Hashing ✓
Static nodes  → Use Traditional Hashing ✓
```

---

## Summary

### Problem with Traditional Hashing

**Issue:**
```
When nodes change (add/remove):
- Mod value changes
- All keys need rebalancing
- Very expensive operation

Example:
3 nodes → 4 nodes
Mod 3 → Mod 4
~75% of keys rebalanced
```

### Consistent Hashing Solution

**Key Concepts:**

1. **Virtual Ring**
   - Circular structure (0 to N, back to 0)
   - Fixed size

2. **Server Placement**
   - Hash servers to positions on ring
   - Servers placed at hash positions

3. **Key Placement**
   - Hash keys to positions on ring
   - Keys placed at hash positions

4. **Clockwise Rule**
   - Move clockwise from key
   - First server found handles key

5. **Virtual Nodes**
   - Replicate servers at multiple positions
   - Better distribution
   - Prevents clustering

### Benefits

✅ **Minimal Rebalancing**
```
Target: (1/n)% of keys
Example: 4 nodes → 25% rebalancing
Much better than 75% in traditional hashing
```

✅ **Equal Distribution**
```
With virtual nodes:
- Load balanced across servers
- No single server overloaded
```

✅ **Dynamic Scalability**
```
Add/remove servers:
- Only affected keys rebalanced
- Most keys stay in place
```

### Use Cases

**Use Consistent Hashing for:**
- Load balancing (dynamic app servers)
- Horizontal sharding (dynamic DB servers)
- Distributed caching
- Any scenario with dynamic nodes

**Don't use for:**
- Static, fixed number of nodes
- Simple scenarios (use traditional hashing)

### Key Formula

```
Rebalancing Percentage = (1 / n) × 100%

Where:
n = Number of active nodes

Example:
4 nodes → 25% rebalancing
10 nodes → 10% rebalancing
100 nodes → 1% rebalancing
```

### Visual Summary

```
Traditional Hashing:
┌──────┐ ┌──────┐ ┌──────┐     Add Node     ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│Node 1│ │Node 2│ │Node 3│  ────────────►  │Node 1│ │Node 2│ │Node 3│ │Node 4│
└──────┘ └──────┘ └──────┘                  └──────┘ └──────┘ └──────┘ └──────┘
  Mod 3                                        Mod 4
                                              ~75% rebalanced ❌

Consistent Hashing:
        0                                             0
    11  │  1                                      11  │  1
  S3    │    2 S1          Add S4 at 5         S3    │    2 S1
9       │       3       ────────────────►     9       │       3
  8     │     4                               8       │     4
    7   │   5                                   7     │   5 S4
        6 S2                                          6 S2
                                              ~25% rebalanced ✓
```

### Interview Tips

**Be Ready to Explain:**

1. **Problem:**
   - Why traditional hashing fails with dynamic nodes
   - Rebalancing cost

2. **Solution:**
   - Virtual ring concept
   - Clockwise rule
   - Virtual nodes

3. **Benefits:**
   - Minimal rebalancing
   - Equal distribution
   - Scalability

4. **Use Cases:**
   - Load balancing
   - Sharding
   - Distributed systems

**Common Interview Question:**
> "How does consistent hashing minimize rebalancing when nodes are added or removed?"

**Answer:**
- Uses virtual ring with clockwise rule
- Only keys between new node and next node affected
- Achieves (1/n)% rebalancing vs ~75% in traditional hashing
- Virtual nodes ensure equal distribution

---

**End of Lecture**

Consistent hashing is a fundamental technique in distributed systems, enabling efficient scaling with minimal data movement. Understanding this concept is crucial for designing scalable architectures.
