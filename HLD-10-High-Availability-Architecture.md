# HLD-10: High Availability Architecture (Active-Passive vs Active-Active)

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Different Ways This Question is Asked](#different-ways-this-question-is-asked)
3. [Single Node Architecture (The Problem)](#single-node-architecture-the-problem)
4. [Multi-Node Architecture](#multi-node-architecture)
5. [Active-Passive Architecture](#active-passive-architecture)
   - [Basic Setup](#basic-setup)
   - [How It Works](#how-it-works)
   - [Failure Handling](#failure-handling)
   - [Advantages](#advantages)
   - [Disadvantages](#disadvantages)
6. [Active-Active Architecture](#active-active-architecture)
   - [Basic Setup](#basic-setup-1)
   - [How It Works](#how-it-works-1)
   - [Advantages](#advantages-1)
   - [Challenges](#challenges)
7. [Comparison: Active-Passive vs Active-Active](#comparison-active-passive-vs-active-active)
8. [Summary](#summary)
9. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** High Availability Architecture Design

**Importance:**
- Very important interview question
- Theoretical foundation for many implementation questions
- Critical for production systems

**Goal:** Design architecture that achieves:
- ✅ High availability (99.999%)
- ✅ Data resilience
- ✅ No single point of failure
- ✅ Disaster recovery capability

---

## Different Ways This Question is Asked

### Interview Question Variations

**1. Design High Availability Architecture**
```
"Design a system with high availability"
```

**2. Design Data Resilience Architecture**
```
"Design an architecture that can recover from failures"

Resilience = Capability to come out of failure
```

**3. Design for Five Nines Availability**
```
"Design architecture to achieve 99.999% availability"

Five Nines = 99.999%
Downtime allowed: ~5 minutes per year
```

**4. Avoid Single Point of Failure**
```
"Design architecture with no single point of failure"

SPOF = If one component fails, entire system fails
```

**5. Active-Passive vs Active-Active**
```
"Explain active-passive vs active-active architecture"
"When to use which architecture?"
```

**All these questions point to the same concept!**

---

## Single Node Architecture (The Problem)

### Basic Architecture

```
┌────────┐
│ Client │ (Mobile, Laptop, Desktop)
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────────────────┐
│   Microservices Layer           │
├─────────────────────────────────┤
│  ┌──────┐  ┌──────┐  ┌──────┐  │
│  │App X │  │App Y │  │App Z │  │
│  └──────┘  └──────┘  └──────┘  │
│     │         │         │       │
│     └─────────┼─────────┘       │
└───────────────┼─────────────────┘
                ▼
         ┌─────────────┐
         │ Primary DB  │ ← Single Node
         └─────────────┘
```

---

### The Problem: DB Failure

**Scenario: DB Goes Down**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────────────────┐
│   Microservices Layer           │
│  ┌──────┐  ┌──────┐  ┌──────┐  │
│  │App X │  │App Y │  │App Z │  │
│  └──────┘  └──────┘  └──────┘  │
│     │         │         │       │
│     └─────────┼─────────┘       │
└───────────────┼─────────────────┘
                ▼
         ┌─────────────┐
         │ Primary DB  │
         │   ✗ DOWN    │ ← Failure!
         └─────────────┘

Result:
❌ All read requests fail
❌ All write requests fail
❌ Entire application down
```

---

### Problems with Single Node

```
Question: Does it achieve 99.999% availability?
Answer: ❌ NO

Question: Does it have single point of failure?
Answer: ✅ YES (DB is SPOF)

Question: Is it data resilient?
Answer: ❌ NO (Cannot recover from failure)

Question: How long to recover?
Answer: Hours or even days (manual intervention needed)
```

**Conclusion:**
```
Single node architecture:
❌ Not highly available
❌ Has single point of failure
❌ Not resilient
❌ Cannot achieve five nines

Need: Multi-node architecture
```

---

## Multi-Node Architecture

### Two Types

```
1. Active-Passive Architecture
   - One primary DB
   - One or more replica DBs
   - Disaster recovery setup

2. Active-Active Architecture
   - Multiple primary DBs
   - All DBs active
   - Bidirectional sync
```

---

## Active-Passive Architecture

### Basic Setup

**Data Centers:**

```
Data Center 1 (Mumbai)          Data Center 2 (Pune)
┌─────────────────────┐        ┌─────────────────────┐
│                     │        │                     │
│  Microservices      │        │  Microservices      │
│  ┌──────┐┌──────┐  │        │  ┌──────┐┌──────┐  │
│  │App X││App Y│  │        │  │App X││App Y│  │
│  └──────┘└──────┘  │        │  └──────┘└──────┘  │
│  ┌──────┐          │        │  ┌──────┐          │
│  │App Z │          │        │  │App Z │          │
│  └──────┘          │        │  └──────┘          │
│       │            │        │       │            │
│       ▼            │        │       │            │
│  ┌─────────┐      │        │       │            │
│  │Primary  │      │        │       │            │
│  │Live DB  │◄─────┼────────┼───────┘            │
│  │(R/W)    │      │        │                     │
│  └─────────┘      │        │  ┌─────────┐       │
│       │           │        │  │ Replica │       │
│       │ Sync      │        │  │Read-Only│       │
│       └──────────►│────────┼─►│   DB    │       │
│                   │        │  └─────────┘       │
└─────────────────────┘        └─────────────────────┘
     PRIMARY                    DISASTER RECOVERY
                                    (DR)
```

**Key Characteristics:**

```
Primary Data Center (Mumbai):
✓ Has Primary/Live/Read-Write DB
✓ Handles all writes
✓ Handles all reads (for requests to DC1)

DR Data Center (Pune):
✓ Has Replica/Read-Only DB
✓ Receives sync from Primary
✓ Standby for disaster recovery
```

---

### Terminology

**Different Names for Same Concept:**

```
Primary DB = Live DB = Read-Write DB
All mean: The main database that accepts writes

Replica DB = Read-Only DB = Standby DB
All mean: Copy of primary, for reads and failover

DR Data Center = Disaster Recovery Data Center
Backup data center for failover
```

---

### How It Works

#### Request Flow: Data Center 1

**Write Request to DC1:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐
│  Data Center 1      │
│  (Mumbai - Primary) │
│                     │
│  ┌──────┐          │
│  │App X │          │
│  └──────┘          │
│      │             │
│      ▼             │
│  ┌─────────┐      │
│  │Primary  │      │
│  │  DB     │      │
│  │ WRITE ✓ │      │
│  └─────────┘      │
└─────────────────────┘

Write goes to Primary DB
```

**Read Request to DC1:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐
│  Data Center 1      │
│  (Mumbai - Primary) │
│                     │
│  ┌──────┐          │
│  │App X │          │
│  └──────┘          │
│      │             │
│      ▼             │
│  ┌─────────┐      │
│  │Primary  │      │
│  │  DB     │      │
│  │ READ ✓  │      │
│  └─────────┘      │
└─────────────────────┘

Read from Primary DB
```

---

#### Request Flow: Data Center 2 (DR)

**Write Request to DC2:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐        ┌─────────────────────┐
│  Data Center 2      │        │  Data Center 1      │
│  (Pune - DR)        │        │  (Mumbai - Primary) │
│                     │        │                     │
│  ┌──────┐          │        │                     │
│  │App X │          │        │                     │
│  └──────┘          │        │                     │
│      │             │        │                     │
│      │ Route to    │        │                     │
│      │ Primary     │        │                     │
│      └─────────────┼───────►│  ┌─────────┐       │
│                     │        │  │Primary  │       │
│  ┌─────────┐      │        │  │  DB     │       │
│  │Replica  │      │        │  │ WRITE ✓ │       │
│  │  DB     │      │        │  └─────────┘       │
│  │(Not Used)│     │        │                     │
│  └─────────┘      │        │                     │
└─────────────────────┘        └─────────────────────┘

Write request routes to Primary DB in DC1
```

**Key Point:**
```
Request to DR Data Center:
- Application routes traffic to Primary DB
- Primary DB is in different data center
- Network latency added
```

---

#### Improved Model: Read from Replica

**Read Request to DC2:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐
│  Data Center 2      │
│  (Pune - DR)        │
│                     │
│  ┌──────┐          │
│  │App X │          │
│  └──────┘          │
│      │             │
│      ▼             │
│  ┌─────────┐      │
│  │Replica  │      │
│  │  DB     │      │
│  │ READ ✓  │      │
│  └─────────┘      │
└─────────────────────┘

Read from local Replica DB (Read-Only)
```

**Optimization:**
```
DC1 (Primary):
- Read → Primary DB
- Write → Primary DB

DC2 (DR):
- Read → Replica DB (local, faster)
- Write → Primary DB (remote, slower)
```

---

### Data Synchronization

**Unidirectional Sync:**

```
┌─────────────────────┐        ┌─────────────────────┐
│  Data Center 1      │        │  Data Center 2      │
│                     │        │                     │
│  ┌─────────┐       │        │  ┌─────────┐       │
│  │Primary  │       │        │  │Replica  │       │
│  │  DB     │───────┼───────►│  │  DB     │       │
│  └─────────┘       │  Sync  │  └─────────┘       │
│                     │        │                     │
└─────────────────────┘        └─────────────────────┘

One-way replication:
Primary → Replica
```

**Sync Process:**

```
1. Write happens on Primary DB
2. Primary DB logs transaction
3. Replication process reads log
4. Sends changes to Replica DB
5. Replica DB applies changes
```

---

### Failure Handling

**Scenario: Primary DB Fails**

```
BEFORE FAILURE:

DC1 (Mumbai)                    DC2 (Pune)
┌─────────────┐                ┌─────────────┐
│ ┌─────────┐ │                │ ┌─────────┐ │
│ │Primary  │ │────Sync───────►│ │Replica  │ │
│ │  DB     │ │                │ │  DB     │ │
│ └─────────┘ │                │ └─────────┘ │
└─────────────┘                └─────────────┘
   ↑ Traffic                      (Standby)


FAILURE OCCURS:

DC1 (Mumbai)                    DC2 (Pune)
┌─────────────┐                ┌─────────────┐
│ ┌─────────┐ │                │ ┌─────────┐ │
│ │Primary  │ │                │ │Replica  │ │
│ │  DB     │ │                │ │  DB     │ │
│ │ ✗ DOWN  │ │                │ └─────────┘ │
│ └─────────┘ │                └─────────────┘
└─────────────┘
   ✗ No Traffic


AFTER FAILOVER:

DC1 (Mumbai)                    DC2 (Pune)
┌─────────────┐                ┌─────────────┐
│ ┌─────────┐ │                │ ┌─────────┐ │
│ │Primary  │ │                │ │Primary  │ │
│ │  DB     │ │                │ │  DB     │ │
│ │ ✗ DOWN  │ │                │ │(Promoted)│ │
│ └─────────┘ │                │ └─────────┘ │
└─────────────┘                └─────────────┘
                                  ↑ Traffic
```

**Failover Steps:**

```
1. Detect Primary DB failure
2. Route traffic from DC1 to DC2
3. Promote Replica DB to Primary
4. DC2 now handles all traffic
5. System remains available ✓
```

**Recovery Steps:**

```
After DC1 DB is fixed:
1. Bring DC1 DB back online
2. Make it Replica/Read-Only
3. Sync from new Primary (DC2)
4. Optionally: Fail back to DC1 as Primary
```

---

### Advantages

```
✅ High Availability
   - System survives DB failure
   - Automatic failover possible

✅ No Single Point of Failure
   - Multiple data centers
   - Redundant databases

✅ Data Resilience
   - Can recover from failure
   - Data replicated to DR site

✅ Disaster Recovery
   - Entire data center can fail
   - DR site takes over

✅ Read Scalability (with optimization)
   - Reads can use Replica DB
   - Reduces load on Primary
```

---

### Disadvantages

#### 1. Latency for DR Data Center Requests

**Problem:**

```
Request to DC1 (Primary):
Client → LB → DC1 App → DC1 DB
Latency: 1 second

Request to DC2 (DR):
Client → LB → DC2 App → DC1 DB (remote)
Latency: 2 seconds (network overhead)
```

**Explanation:**

```
DC2 (Pune) → DC1 (Mumbai) for writes
- Network latency added
- Slower response time
- Inconsistent user experience
```

---

#### 2. Downtime During Failover

**Problem:**

```
T0: Primary DB fails
T1: Detect failure (monitoring)
T2: Initiate failover
T3: Route traffic to DR
T4: Promote Replica to Primary
T5: Resume operations

Downtime: T0 to T5 (10-15 minutes)
```

**During Failover:**

```
All writes fail ✗
Reads may fail ✗
System partially down
```

---

#### 3. Write Scalability Limited

**Problem:**

```
All writes go to ONE Primary DB

High write load:
┌──────┐  ┌──────┐  ┌──────┐
│App 1 │  │App 2 │  │App 3 │
└──────┘  └──────┘  └──────┘
    │         │         │
    └─────────┼─────────┘
              ▼
         ┌─────────┐
         │Primary  │ ← Bottleneck!
         │  DB     │
         └─────────┘

Cannot scale writes horizontally
```

**Limitation:**

```
Write-heavy applications:
❌ Active-Passive doesn't scale well
❌ Single Primary becomes bottleneck
✓ Need Active-Active for write scaling
```

---

#### 4. Database Limitations

**Oracle, MySQL, PostgreSQL:**

```
❌ Not Multi-Master
❌ Only one Primary/Live DB
❌ Cannot write to multiple DBs simultaneously

Must choose:
- Which DB is Primary
- Others are Replicas
```

---

### Summary: Active-Passive

```
┌────────────────────────────────────────────────┐
│         Active-Passive Architecture            │
├────────────────────────────────────────────────┤
│                                                │
│ Setup:                                         │
│ - One Primary DB (read/write)                  │
│ - One or more Replica DBs (read-only)          │
│ - Unidirectional sync (Primary → Replica)      │
│                                                │
│ Advantages:                                    │
│ ✓ High availability                            │
│ ✓ Disaster recovery                            │
│ ✓ No single point of failure                   │
│ ✓ Read scalability (with optimization)         │
│                                                │
│ Disadvantages:                                 │
│ ✗ Latency for DR requests                      │
│ ✗ Downtime during failover (10-15 min)         │
│ ✗ Write scalability limited                    │
│ ✗ Underutilized resources (Replica idle)       │
│                                                │
│ Best For:                                      │
│ - Read-heavy applications                      │
│ - Oracle, MySQL, PostgreSQL                    │
│ - Disaster recovery requirement                │
└────────────────────────────────────────────────┘
```

---

## Active-Active Architecture

### Basic Setup

**Data Centers:**

```
Data Center 1 (Mumbai)          Data Center 2 (Pune)
┌─────────────────────┐        ┌─────────────────────┐
│                     │        │                     │
│  Microservices      │        │  Microservices      │
│  ┌──────┐┌──────┐  │        │  ┌──────┐┌──────┐  │
│  │App X││App Y│  │        │  │App X││App Y│  │
│  └──────┘└──────┘  │        │  └──────┘└──────┘  │
│  ┌──────┐          │        │  ┌──────┐          │
│  │App Z │          │        │  │App Z │          │
│  └──────┘          │        │  └──────┘          │
│       │            │        │       │            │
│       ▼            │        │       ▼            │
│  ┌─────────┐      │        │  ┌─────────┐       │
│  │Primary  │◄─────┼────────┼─►│Primary  │       │
│  │Live DB  │      │  Sync  │  │Live DB  │       │
│  │(R/W)    │──────┼───────►│  │(R/W)    │       │
│  └─────────┘      │        │  └─────────┘       │
│                   │        │                     │
└─────────────────────┘        └─────────────────────┘
     PRIMARY                       PRIMARY
     ACTIVE                        ACTIVE
```

**Key Characteristics:**

```
Both Data Centers:
✓ Have Primary/Live DB
✓ Handle reads AND writes
✓ Bidirectional sync
✓ Fully active and utilized
```

---

### Multi-Master Support

**Database Requirements:**

```
Active-Active requires Multi-Master support:

✓ Cassandra (NoSQL)
✓ DynamoDB (NoSQL)
✓ MongoDB (with replica sets)
✓ CouchDB (NoSQL)
✓ Most NoSQL databases

✗ Oracle (Not multi-master)
✗ MySQL (Not multi-master by default)
✗ PostgreSQL (Not multi-master by default)
```

**Multi-Master:**
```
Multiple DBs can accept writes simultaneously
No single "Primary" designation
All DBs are equal peers
```

---

### How It Works

#### Request Flow: Data Center 1

**Write Request to DC1:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐
│  Data Center 1      │
│  (Mumbai - Active)  │
│                     │
│  ┌──────┐          │
│  │App X │          │
│  └──────┘          │
│      │             │
│      ▼             │
│  ┌─────────┐      │
│  │Primary  │      │
│  │  DB     │      │
│  │ WRITE ✓ │      │
│  └─────────┘      │
└─────────────────────┘

Write to local DB (fast)
```

**Read Request to DC1:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐
│  Data Center 1      │
│  (Mumbai - Active)  │
│                     │
│  ┌──────┐          │
│  │App X │          │
│  └──────┘          │
│      │             │
│      ▼             │
│  ┌─────────┐      │
│  │Primary  │      │
│  │  DB     │      │
│  │ READ ✓  │      │
│  └─────────┘      │
└─────────────────────┘

Read from local DB (fast)
```

---

#### Request Flow: Data Center 2

**Write Request to DC2:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐
│  Data Center 2      │
│  (Pune - Active)    │
│                     │
│  ┌──────┐          │
│  │App X │          │
│  └──────┘          │
│      │             │
│      ▼             │
│  ┌─────────┐      │
│  │Primary  │      │
│  │  DB     │      │
│  │ WRITE ✓ │      │
│  └─────────┘      │
└─────────────────────┘

Write to local DB (fast)
No routing to other DC!
```

**Read Request to DC2:**

```
┌────────┐
│ Client │
└────────┘
    │
    ▼
┌─────────────┐
│Load Balancer│
└─────────────┘
    │
    ▼
┌─────────────────────┐
│  Data Center 2      │
│  (Pune - Active)    │
│                     │
│  ┌──────┐          │
│  │App X │          │
│  └──────┘          │
│      │             │
│      ▼             │
│  ┌─────────┐      │
│  │Primary  │      │
│  │  DB     │      │
│  │ READ ✓  │      │
│  └─────────┘      │
└─────────────────────┘

Read from local DB (fast)
```

**Key Difference from Active-Passive:**
```
Active-Passive:
DC2 write → Routes to DC1 (slow)

Active-Active:
DC2 write → Local DB (fast)
```

---

### Data Synchronization

**Bidirectional Sync:**

```
┌─────────────────────┐        ┌─────────────────────┐
│  Data Center 1      │        │  Data Center 2      │
│                     │        │                     │
│  ┌─────────┐       │        │  ┌─────────┐       │
│  │Primary  │◄──────┼────────┼──│Primary  │       │
│  │  DB     │───────┼───────►│  │  DB     │       │
│  └─────────┘       │  Sync  │  └─────────┘       │
│                     │        │                     │
└─────────────────────┘        └─────────────────────┘

Two-way replication:
DC1 ↔ DC2
```

**Sync Process:**

```
Write in DC1:
1. Write to DC1 DB
2. Replicate to DC2 DB

Write in DC2:
1. Write to DC2 DB
2. Replicate to DC1 DB

Both happening simultaneously!
```

---

### Advantages

```
✅ Full Resource Utilization
   - Both DBs active
   - No idle resources
   - Better ROI

✅ Higher Throughput
   - Both DBs handle reads
   - Both DBs handle writes
   - 2x capacity

✅ Low Latency
   - All requests local
   - No cross-DC routing
   - Consistent performance

✅ Write Scalability
   - Writes distributed across DBs
   - No single bottleneck
   - Horizontal scaling

✅ High Availability
   - Either DC can fail
   - Other continues serving
   - No downtime

✅ Better User Experience
   - Fast response times
   - No latency variation
   - Consistent performance
```

---

### Challenges

#### 1. Synchronization Complexity

**The Core Challenge:**

```
Bidirectional sync is complex!

DC1 DB ↔ DC2 DB

Challenges:
- Conflict resolution
- Consistency guarantees
- Network partitions
- Concurrent updates
```

---

#### 2. Concurrent Updates to Same Row

**Problem:**

```
Time T0:
DC1: UPDATE user SET name = 'Alice' WHERE id = 1
DC2: UPDATE user SET name = 'Bob' WHERE id = 1

Both updates happen simultaneously!

Sync:
DC1 → DC2: name = 'Alice'
DC2 → DC1: name = 'Bob'

Conflict! Which value wins?
```

**Conflict Resolution Strategies:**

```
1. Last Write Wins (LWW)
   - Use timestamp
   - Latest update wins
   - May lose data

2. Version Vectors
   - Track causality
   - Detect conflicts
   - Application resolves

3. CRDTs (Conflict-free Replicated Data Types)
   - Mathematically commutative
   - Automatic merge
   - No conflicts

4. Application-Level Resolution
   - Custom logic
   - Business rules
   - Manual intervention
```

---

#### 3. Read-Your-Write Consistency

**Problem:**

```
T0: User writes to DC1
    DC1 DB: data = 'new'

T1: User reads from DC2 (before sync)
    DC2 DB: data = 'old' (stale!)

User sees old data after writing new data!
```

**Solutions:**

```
1. Sticky Sessions
   - Route user to same DC
   - Consistent view

2. Read-After-Write Consistency
   - Read from same DB that was written
   - Guaranteed fresh data

3. Quorum Reads
   - Read from majority of nodes
   - Ensure latest data
```

---

#### 4. Network Partitions

**Problem:**

```
DC1 (Mumbai)          DC2 (Pune)
┌─────────┐          ┌─────────┐
│Primary  │    ✗     │Primary  │
│  DB     │  Network │  DB     │
│         │  Failure │         │
└─────────┘          └─────────┘

Cannot sync!
Both continue accepting writes
Diverging data!
```

**Solutions:**

```
1. Eventual Consistency
   - Accept temporary divergence
   - Sync when network recovers

2. Conflict Resolution
   - Merge diverged data
   - Use version vectors

3. CAP Theorem Trade-offs
   - Choose AP (Availability + Partition Tolerance)
   - Sacrifice strong Consistency
```

---

#### 5. Increased Complexity

**Operational Complexity:**

```
Active-Passive:
- Simple failover
- Clear Primary/Replica roles
- Easier to reason about

Active-Active:
- Complex sync logic
- Conflict resolution needed
- Harder to debug
- More monitoring required
```

---

### When to Use Active-Active

```
✓ Write-heavy applications
  - High write throughput needed
  - Cannot bottleneck on single DB

✓ Global applications
  - Users worldwide
  - Low latency critical
  - Local writes important

✓ NoSQL databases
  - Cassandra, DynamoDB, etc.
  - Built-in multi-master support

✓ Eventual consistency acceptable
  - Can tolerate temporary inconsistency
  - Not financial transactions

✓ High availability critical
  - Cannot afford any downtime
  - Need instant failover
```

---

### Summary: Active-Active

```
┌────────────────────────────────────────────────┐
│         Active-Active Architecture             │
├────────────────────────────────────────────────┤
│                                                │
│ Setup:                                         │
│ - Multiple Primary DBs (all read/write)        │
│ - Bidirectional sync                           │
│ - Multi-master support required                │
│                                                │
│ Advantages:                                    │
│ ✓ Full resource utilization                    │
│ ✓ Higher throughput (reads + writes)           │
│ ✓ Low latency (local operations)               │
│ ✓ Write scalability                            │
│ ✓ High availability                            │
│                                                │
│ Challenges:                                    │
│ ✗ Synchronization complexity                   │
│ ✗ Conflict resolution needed                   │
│ ✗ Read-your-write consistency issues           │
│ ✗ Network partition handling                   │
│ ✗ Operational complexity                       │
│                                                │
│ Best For:                                      │
│ - Write-heavy applications                     │
│ - NoSQL databases (Cassandra, DynamoDB)        │
│ - Global low-latency requirements              │
│ - Eventual consistency acceptable              │
└────────────────────────────────────────────────┘
```

---

## Comparison: Active-Passive vs Active-Active

### Side-by-Side Comparison

```
┌──────────────────┬─────────────────┬─────────────────┐
│    Aspect        │ Active-Passive  │ Active-Active   │
├──────────────────┼─────────────────┼─────────────────┤
│ Primary DBs      │ One             │ Multiple        │
│                  │                 │                 │
│ Replica DBs      │ One or more     │ None (all       │
│                  │                 │ primary)        │
│                  │                 │                 │
│ Sync Direction   │ Unidirectional  │ Bidirectional   │
│                  │ (Primary→Replica│ (Both ways)     │
│                  │                 │                 │
│ Write Handling   │ Only Primary    │ All DBs         │
│                  │                 │                 │
│ Read Handling    │ Primary +       │ All DBs         │
│                  │ Replica (opt)   │                 │
│                  │                 │                 │
│ Resource Usage   │ Replica idle    │ Fully utilized  │
│                  │                 │                 │
│ Write Latency    │ High for DR DC  │ Low (local)     │
│                  │                 │                 │
│ Read Latency     │ Low (with opt)  │ Low (local)     │
│                  │                 │                 │
│ Failover Time    │ 10-15 minutes   │ Instant         │
│                  │                 │                 │
│ Complexity       │ Low             │ High            │
│                  │                 │                 │
│ Consistency      │ Strong (single  │ Eventual        │
│                  │ Primary)        │                 │
│                  │                 │                 │
│ Conflict Res.    │ Not needed      │ Required        │
│                  │                 │                 │
│ DB Support       │ Oracle, MySQL,  │ Cassandra,      │
│                  │ PostgreSQL      │ DynamoDB, NoSQL │
│                  │                 │                 │
│ Write Scaling    │ Limited (single │ Horizontal      │
│                  │ Primary)        │                 │
│                  │                 │                 │
│ Best For         │ Read-heavy apps │ Write-heavy apps│
└──────────────────┴─────────────────┴─────────────────┘
```

---

### Decision Matrix

**Choose Active-Passive When:**

```
✓ Using Oracle, MySQL, PostgreSQL
✓ Read-heavy application
✓ Strong consistency required
✓ Simpler operations preferred
✓ Disaster recovery is main goal
✓ Write load is manageable
```

**Choose Active-Active When:**

```
✓ Using Cassandra, DynamoDB, NoSQL
✓ Write-heavy application
✓ Eventual consistency acceptable
✓ Global low-latency required
✓ Maximum availability needed
✓ High write throughput needed
```

---

### Visual Comparison

**Active-Passive:**

```
┌─────────────┐          ┌─────────────┐
│   DC 1      │          │   DC 2      │
│             │          │             │
│ ┌─────────┐ │          │ ┌─────────┐ │
│ │Primary  │ │──Sync───►│ │Replica  │ │
│ │  DB     │ │          │ │  DB     │ │
│ │ (R/W)   │ │          │ │ (R only)│ │
│ └─────────┘ │          │ └─────────┘ │
└─────────────┘          └─────────────┘
   ↑ All writes             ↑ Reads only
   ↑ All reads (DC1)        (optional)

Characteristics:
- One active, one standby
- Unidirectional sync
- Underutilized resources
```

**Active-Active:**

```
┌─────────────┐          ┌─────────────┐
│   DC 1      │          │   DC 2      │
│             │          │             │
│ ┌─────────┐ │          │ ┌─────────┐ │
│ │Primary  │◄┼──Sync───┼►│Primary  │ │
│ │  DB     │─┼──Sync──►│ │  DB     │ │
│ │ (R/W)   │ │          │ │ (R/W)   │ │
│ └─────────┘ │          │ └─────────┘ │
└─────────────┘          └─────────────┘
   ↑ All operations       ↑ All operations

Characteristics:
- Both active
- Bidirectional sync
- Fully utilized resources
```

---

## Summary

### Key Concepts

**1. High Availability Goals:**
```
✓ 99.999% uptime (five nines)
✓ No single point of failure
✓ Data resilience
✓ Disaster recovery
```

**2. Single Node Problems:**
```
❌ Single point of failure
❌ No redundancy
❌ Cannot recover from failure
❌ Downtime = hours/days
```

**3. Active-Passive:**
```
Setup:
- One Primary DB
- One or more Replica DBs
- Unidirectional sync

Best for:
- Read-heavy apps
- Oracle/MySQL/PostgreSQL
- Strong consistency needs
```

**4. Active-Active:**
```
Setup:
- Multiple Primary DBs
- Bidirectional sync
- Multi-master support

Best for:
- Write-heavy apps
- NoSQL databases
- Global applications
```

---

### Architecture Evolution

```
Single Node
    ↓
    ❌ Single point of failure
    ❌ No resilience
    ↓
Active-Passive
    ↓
    ✓ High availability
    ✓ Disaster recovery
    ❌ Write bottleneck
    ❌ Latency for DR
    ↓
Active-Active
    ↓
    ✓ Full utilization
    ✓ Write scalability
    ✓ Low latency
    ❌ Sync complexity
```

---

## Interview Tips

### Do's ✅

**1. Start with Single Node Problem**
```
"Let me first explain why single node doesn't work.
If the DB fails, entire system goes down.
This violates high availability requirements."
```

**2. Explain Both Architectures**
```
"There are two approaches:
1. Active-Passive: One primary, replicas for failover
2. Active-Active: Multiple primaries, bidirectional sync

Let me explain each..."
```

**3. Mention Database Constraints**
```
"Active-Passive works with Oracle, MySQL, PostgreSQL
because they don't support multi-master.

Active-Active requires multi-master support,
found in Cassandra, DynamoDB, and most NoSQL databases."
```

**4. Discuss Trade-offs**
```
"Active-Passive:
+ Simpler
+ Strong consistency
- Write bottleneck
- Latency for DR

Active-Active:
+ Write scalability
+ Low latency
- Sync complexity
- Eventual consistency"
```

**5. Address Synchronization**
```
"Active-Passive: Unidirectional sync, simpler

Active-Active: Bidirectional sync, needs:
- Conflict resolution
- Version vectors or CRDTs
- Network partition handling"
```

### Don'ts ❌

**1. Don't Confuse the Two**
```
❌ "Active-Passive has multiple primaries"
✓ "Active-Passive has ONE primary, others are replicas"
```

**2. Don't Ignore Database Limitations**
```
❌ "Use Active-Active with MySQL"
✓ "MySQL doesn't support multi-master by default,
    use Active-Passive or switch to NoSQL"
```

**3. Don't Forget Failover Time**
```
❌ "Active-Passive has instant failover"
✓ "Active-Passive failover takes 10-15 minutes
    Active-Active has instant failover"
```

**4. Don't Overlook Complexity**
```
❌ "Active-Active is always better"
✓ "Active-Active has sync complexity,
    only use when write scaling needed"
```

### Common Interview Questions

**Q1: "Why not always use Active-Active?"**

```
Answer:
"Active-Active has significant complexity:
1. Conflict resolution needed
2. Eventual consistency (not always acceptable)
3. Complex operational overhead
4. Requires multi-master DB support

Active-Passive is simpler and sufficient for:
- Read-heavy applications
- Strong consistency requirements
- Traditional RDBMS (Oracle, MySQL)

Choose based on requirements, not just features."
```

**Q2: "How do you handle conflicts in Active-Active?"**

```
Answer:
"Several strategies:
1. Last Write Wins (LWW) - Use timestamps
2. Version Vectors - Track causality
3. CRDTs - Conflict-free data types
4. Application-level resolution

Choice depends on:
- Data type
- Business requirements
- Consistency needs

Example: For counter, use CRDT.
For user profile, use LWW with timestamp."
```

**Q3: "What happens during network partition?"**

```
Answer:
"Network partition between DCs:

Active-Passive:
- Primary continues serving
- Replica can't sync (but not serving writes)
- Sync resumes when network recovers

Active-Active:
- Both DCs continue serving (AP in CAP)
- Data diverges temporarily
- Conflict resolution when network recovers
- Eventual consistency model"
```

**Q4: "How to achieve five nines (99.999%)?"**

```
Answer:
"Five nines = 5 minutes downtime per year

Strategies:
1. Multi-DC setup (Active-Passive or Active-Active)
2. Automated failover (reduce manual intervention)
3. Health monitoring and alerts
4. Regular disaster recovery drills
5. Redundancy at every layer:
   - Load balancers
   - Application servers
   - Databases
   - Network paths

Active-Active gets closer to five nines because:
- Instant failover
- No downtime during DC failure"
```

**Q5: "Can you use Active-Active with MySQL?"**

```
Answer:
"MySQL doesn't support multi-master by default.

Options:
1. Use Active-Passive (recommended)
2. Use MySQL Group Replication (limited multi-master)
3. Use third-party solutions (Galera Cluster)
4. Switch to NoSQL (Cassandra, DynamoDB)

For true Active-Active with full multi-master:
Recommend NoSQL databases designed for it."
```

### Key Points to Remember

```
1. Single node = Single point of failure
   Always avoid in production

2. Active-Passive = One Primary + Replicas
   Unidirectional sync, simpler

3. Active-Active = Multiple Primaries
   Bidirectional sync, complex

4. Database matters:
   Oracle/MySQL/PostgreSQL → Active-Passive
   Cassandra/DynamoDB → Active-Active

5. Trade-offs:
   Simplicity vs Scalability
   Consistency vs Availability

6. Failover time:
   Active-Passive: 10-15 minutes
   Active-Active: Instant

7. Sync complexity:
   Active-Passive: Simple (one-way)
   Active-Active: Complex (conflicts)
```

---

**End of Lecture**

High availability architecture is fundamental to production systems. Understanding the difference between Active-Passive and Active-Active, their trade-offs, and when to use each is critical for system design interviews. Remember: choose based on requirements (read/write patterns, consistency needs, database constraints), not just features.

**Key Takeaway:** Active-Passive for simplicity and strong consistency with traditional RDBMS. Active-Active for write scalability and low latency with NoSQL databases. Both eliminate single point of failure and provide disaster recovery.
