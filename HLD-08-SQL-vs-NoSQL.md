# HLD-08: SQL vs NoSQL - When to Use Which?

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Why This Topic is Important](#why-this-topic-is-important)
3. [SQL (Structured Query Language)](#sql-structured-query-language)
   - [Definition](#definition)
   - [Structure](#structure)
   - [Nature](#nature)
   - [Scalability](#scalability)
   - [Property (ACID)](#property-acid)
4. [NoSQL (Not Only SQL)](#nosql-not-only-sql)
   - [Definition](#definition-1)
   - [Structure (4 Types)](#structure-4-types)
   - [Nature](#nature-1)
   - [Scalability](#scalability-1)
   - [Property (BASE)](#property-base)
5. [When to Use SQL](#when-to-use-sql)
6. [When to Use NoSQL](#when-to-use-nosql)
7. [Decision Framework](#decision-framework)
8. [Summary](#summary)
9. [Interview Tips](#interview-tips)

---

## Introduction

**Scope of This Lecture:**
- ✅ When to use SQL
- ✅ When to use NoSQL
- ✅ Basic understanding of both
- ❌ NOT teaching SQL in detail
- ❌ NOT teaching NoSQL in detail

**Goal:** Make informed database decisions in system design interviews

---

## Why This Topic is Important

### Common Interview Mistakes

**❌ Bad Answer 1:**
```
Interviewer: "Which DB will you use - SQL or NoSQL?"
You: "We can use anything"

Problem: Shows lack of understanding
```

**❌ Bad Answer 2:**
```
Interviewer: "Which DB will you use?"
You: "I will use SQL" (or "I will use NoSQL")

Problem: No reasoning provided
```

**✅ Good Answer:**
```
Interviewer: "Which DB will you use?"
You: "I will use SQL because:
     - Data is relational
     - Need ACID properties
     - Require complex queries
     - Data integrity is critical"

Shows: Systematic thinking and understanding
```

### The Problem

```
Starting design without DB justification:
┌─────────────┐
│Load Balancer│
└─────────────┘
       ↓
┌─────────────┐
│     CDN     │
└─────────────┘
       ↓
┌─────────────┐
│   Servers   │
└─────────────┘
       ↓
┌─────────────┐
│   Database  │ ← Which one? Why?
└─────────────┘

Without proper reasoning:
❌ Design appears random
❌ No consideration of constraints
❌ Negative impression
```

---

## SQL (Structured Query Language)

### Definition

**SQL** = Structured Query Language

**Purpose:** Query Relational Database Management System (RDBMS)

```
SQL
 ↓
Query
 ↓
RDBMS (Relational Database Management System)
```

### Structure

#### 1. Tables, Rows, and Columns

```
Table: Employee
┌────────┬──────────────┬────────────┬──────────┐
│   ID   │     Name     │ Department │  Salary  │
├────────┼──────────────┼────────────┼──────────┤
│   1    │  Shreyansh   │     IT     │  100000  │
│   2    │    Rahul     │   Sales    │   80000  │
│   3    │   Shivam     │     HR     │   90000  │
└────────┴──────────────┴────────────┴──────────┘
    ↑           ↑            ↑           ↑
  Column     Column       Column      Column

Each row = Record
Each column = Field
```

#### 2. Predetermined Schema

**What is Schema?**

```
Before inserting data, define:
┌─────────────────────────────────┐
│        Schema Definition        │
├─────────────────────────────────┤
│ Table Name: Employee            │
│                                 │
│ Column 1:                       │
│   Name: "name"                  │
│   Type: VARCHAR(200)            │
│                                 │
│ Column 2:                       │
│   Name: "roll_number"           │
│   Type: INTEGER                 │
│   Length: 20                    │
│                                 │
│ Column 3:                       │
│   Name: "salary"                │
│   Type: DECIMAL(10,2)           │
└─────────────────────────────────┘

Schema MUST be defined BEFORE data insertion
```

**Characteristics:**
- Schema defined beforehand
- Fixed structure
- Cannot insert data without schema

#### 3. Relations Between Tables

```
Parent Table: Department
┌────────┬──────────────┐
│ Dept_ID│ Dept_Name    │
├────────┼──────────────┤
│   1    │      IT      │
│   2    │    Sales     │
└────────┴──────────────┘
    ↑
    │ Primary Key
    │
    │ Foreign Key relationship
    ↓
Child Table: Employee
┌────────┬──────────────┬────────────┐
│ Emp_ID │     Name     │  Dept_ID   │
├────────┼──────────────┼────────────┤
│   1    │  Shreyansh   │     1      │
│   2    │    Rahul     │     2      │
└────────┴──────────────┴────────────┘

Relations maintain data integrity
```

**Summary - Structure:**
```
✓ Tables with rows and columns
✓ Predetermined schema
✓ Relations between tables
```

---

### Nature

**Nature = How data is distributed**

#### Centralized/Concentrated

```
Server 1:
┌─────────────────────────────────────┐
│         DB: Employee                │
├─────────────────────────────────────┤
│                                     │
│  Table 1: Personal_Info             │
│  ┌──────────────────────────────┐   │
│  │ ID: 1                        │   │
│  │ Name: Shreyansh              │   │
│  │ Email: s@example.com         │   │
│  └──────────────────────────────┘   │
│                                     │
│  Table 2: Department_Info           │
│  ┌──────────────────────────────┐   │
│  │ ID: 1                        │   │
│  │ Department: IT               │   │
│  └──────────────────────────────┘   │
│                                     │
│  Table 3: Salary_Info               │
│  ┌──────────────────────────────┐   │
│  │ ID: 1                        │   │
│  │ Salary: 100000               │   │
│  │ Address: XYZ                 │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘

For Employee ID = 1 (Shreyansh):
ALL data across ALL tables is in Server 1
```

**Key Point:**
```
For a particular employee:
- ALL their data
- Across ALL tables
- Resides in ONE server

NOT distributed like:
❌ Table 1 in Server 1
❌ Table 2 in Server 2
❌ Table 3 in Server 3

OR

❌ 5 columns in Server 1
❌ 5 columns in Server 2
```

**Nature: Centralized/Concentrated**

---

### Scalability

**Problem:** Table has 10 million records and growing

**Two Ways to Scale:**

#### 1. Vertical Scaling ⭐ (Better for SQL)

```
Before:
┌─────────────────┐
│    Server 1     │
│  RAM: 16 GB     │
│  Storage: 1 TB  │
│  CPU: 4 cores   │
└─────────────────┘

After Vertical Scaling:
┌─────────────────┐
│    Server 1     │
│  RAM: 64 GB     │ ← Increased
│  Storage: 4 TB  │ ← Increased
│  CPU: 16 cores  │ ← Increased
└─────────────────┘

Increase capacity of SAME server
```

#### 2. Horizontal Scaling (Sharding)

```
Before:
┌─────────────────┐
│    Server 1     │
│  10M records    │
└─────────────────┘

After Horizontal Scaling (Sharding):
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Server 1   │  │  Server 2   │  │  Server 3   │
│  Table 1    │  │  Table 2    │  │  Table 3    │
│  Data       │  │  Data       │  │  Data       │
└─────────────┘  └─────────────┘  └─────────────┘

OR

┌─────────────┐  ┌─────────────┐
│  Server 1   │  │  Server 2   │
│ Columns 1-5 │  │ Columns 6-10│
└─────────────┘  └─────────────┘

Add more servers, distribute data
```

**Problem with Horizontal Scaling in SQL:**
```
❌ NOT well supported
❌ Breaks relational integrity
❌ Complex to manage
❌ Difficult to maintain ACID properties
```

**Scalability Preference:**
```
SQL → Vertical Scaling (Better fit)
```

---

### Property (ACID)

**ACID** = Atomicity, Consistency, Isolation, Durability

```
A = Atomicity
    Transaction is all-or-nothing
    
C = Consistency
    Data remains in valid state
    
I = Isolation
    Concurrent transactions don't interfere
    
D = Durability
    Committed data is permanent
```

#### ACID Ensures

```
✓ Data Integrity
✓ Data Consistency
✓ Transaction Rules
✓ No data loss
```

#### Example: Bank Transfer

```
Transaction: Transfer $100 from Account A to Account B

Step 1: Deduct $100 from Account A
Step 2: Add $100 to Account B

ACID ensures:
- Both steps complete OR neither completes (Atomicity)
- Total money remains same (Consistency)
- Other transactions don't interfere (Isolation)
- Once committed, permanent (Durability)

If Step 1 succeeds but Step 2 fails:
→ Rollback Step 1
→ No money lost
```

**Summary - ACID:**
```
Ensures: Data integrity and consistency
Critical for: Financial systems, critical data
```

---

## NoSQL (Not Only SQL)

### Definition

**Three Names (All Correct):**
1. Non-relational
2. NoSQL
3. **Not Only SQL** ⭐ (Most preferred)

**Why "Not Only SQL"?**
- Can store data in ways beyond traditional SQL
- Flexible data models
- Not limited to relational structure

---

### Structure (4 Types)

NoSQL has **4 different structures** (not just tables):

```
1. Key-Value DB
2. Document DB
3. Column-Wise DB
4. Graph DB
```

---

#### 1. Key-Value DB

**Structure:**

```
┌─────────────┬──────────────────────┐
│     Key     │        Value         │
├─────────────┼──────────────────────┤
│      1      │  "Opaque data"       │
│      2      │  { json object }     │
│      3      │  12345               │
│      4      │  "Any type of data"  │
└─────────────┴──────────────────────┘
```

**Characteristics:**

```
Key: Used for searching
Value: Opaque (cannot query on value)

Value can be:
- String
- Integer
- JSON
- Any data type

Search: ONLY by key
Cannot search by value content
```

**Example: DynamoDB**

```
┌─────────────┬──────────────────────┐
│     Key     │        Value         │
├─────────────┼──────────────────────┤
│   user_1    │  { name: "John",     │
│             │    age: 30,          │
│             │    city: "NYC" }     │
└─────────────┴──────────────────────┘

Query: Get user_1 ✓ (Fast)
Query: Find all users in NYC ✗ (Cannot)
```

**Benefits:**
- Very fast lookups
- Simple structure
- High performance

**Use Cases:**
- Session storage
- User preferences
- Cache

---

#### 2. Document DB

**Structure:**

```
┌─────────────┬──────────────────────────────┐
│     Key     │      Value (JSON/XML)        │
├─────────────┼──────────────────────────────┤
│      1      │  {                           │
│             │    "name": "Shreyansh",      │
│             │    "dept": "IT",             │
│             │    "salary": 100000,         │
│             │    "skills": ["Java", "Go"]  │
│             │  }                           │
└─────────────┴──────────────────────────────┘
```

**Key Difference from Key-Value:**

```
Key-Value DB:
- Value is OPAQUE
- Cannot query on value
- Search ONLY by key

Document DB:
- Value is JSON/XML
- CAN query on value
- Search by key AND value fields
```

**Example: MongoDB**

```
Query by key:
db.users.find({ _id: 1 }) ✓

Query by value fields:
db.users.find({ dept: "IT" }) ✓
db.users.find({ salary: { $gt: 80000 } }) ✓
db.users.find({ skills: "Java" }) ✓

All queries supported!
```

**Benefits:**
- Flexible schema
- Rich queries
- Nested documents

**Use Cases:**
- Content management
- User profiles
- Product catalogs

---

#### 3. Column-Wise DB

**Structure:**

```
Key → List of (Column, Value) pairs

┌─────────────┬──────────────────────────────────┐
│     Key     │      Column-Value Pairs          │
├─────────────┼──────────────────────────────────┤
│   100001    │  [                               │
│             │    (first_name, "Shreyansh"),    │
│             │    (department, "IT"),           │
│             │    (hobby, "Basketball")         │
│             │  ]                               │
│             │                                  │
│   100002    │  [                               │
│             │    (first_name, "Raj"),          │
│             │    (department, "Sales")         │
│             │  ]                               │
└─────────────┴──────────────────────────────────┘
```

**Key Characteristic: Dynamic Columns**

```
Key 100001 has 3 columns:
- first_name
- department
- hobby

Key 100002 has 2 columns:
- first_name
- department

Number of columns can vary per row!
```

**Difference from SQL:**

```
SQL:
┌────────┬────────────┬────────────┬──────────┐
│   ID   │ First_Name │ Department │  Hobby   │
├────────┼────────────┼────────────┼──────────┤
│ 100001 │ Shreyansh  │     IT     │Basketball│
│ 100002 │    Raj     │   Sales    │   NULL   │
└────────┴────────────┴────────────┴──────────┘
All rows must have same columns (NULL if empty)

Column-Wise NoSQL:
Each row can have different columns
No NULL values needed
```

**Example: Cassandra, HBase**

**Benefits:**
- Flexible columns per row
- Efficient storage
- Fast column-based queries

**Use Cases:**
- Time-series data
- Event logging
- Analytics

---

#### 4. Graph DB

**Structure:**

```
Nodes + Edges (Relationships)

Node: Entity
Edge: Relationship between nodes
```

**Visual Representation:**

```
    ┌─────────────┐
    │  Shreyansh  │
    │   (Node)    │
    └─────────────┘
           │
           │ friend_of (Edge)
           ▼
    ┌─────────────┐         ┌─────────────┐
    │     XYZ     │────────►│     ABC     │
    │   (Node)    │friend_of│   (Node)    │
    └─────────────┘ (Edge)  └─────────────┘
           │
           │ works_at (Edge)
           ▼
    ┌─────────────┐
    │  Company A  │
    │   (Node)    │
    └─────────────┘
```

**How It Works:**

```
Direct Relationship Storage:
Node stores references to connected nodes
No need to scan entire database

Example:
Shreyansh node stores:
- friend_of → [XYZ, ABC]
- works_at → [Company A]

Query: "Find Shreyansh's friends"
→ Direct lookup (Very fast!)
```

**Comparison with SQL:**

```
SQL (Relational):
┌──────────────┬──────────────┐
│   Person     │   Friend     │
├──────────────┼──────────────┤
│  Shreyansh   │     XYZ      │
│  Shreyansh   │     ABC      │
│     XYZ      │     DEF      │
└──────────────┴──────────────┘

To find friends:
1. Scan entire table
2. Filter by Person = "Shreyansh"
3. Return Friend column
(Slow for large data)

Graph DB:
Direct relationship stored
Instant lookup
(Very fast!)
```

**Example: Neo4j**

**Benefits:**
- Fast relationship queries
- Natural representation
- Efficient traversal

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

---

### Nature

**Nature = How data is distributed**

#### Distributed

```
Node 1          Node 2          Node 3
┌─────────┐    ┌─────────┐    ┌─────────┐
│ 2M      │    │ 2M      │    │ 2M      │
│ records │    │ records │    │ records │
└─────────┘    └─────────┘    └─────────┘

Total: 6M records distributed across 3 nodes
```

**Example: User Table with 10M Records**

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Node 1    │  │   Node 2    │  │   Node 3    │
│             │  │             │  │             │
│ Users 1-3M  │  │ Users 3-6M  │  │ Users 6-10M │
└─────────────┘  └─────────────┘  └─────────────┘

Data distributed across multiple nodes
```

**Comparison:**

```
SQL: Centralized/Concentrated
- All data for one entity in one server

NoSQL: Distributed
- Data spread across multiple nodes
- Easy to distribute
```

**How Distribution Works:**
(Refer to previous video on Key-Value Store for details)

---

### Scalability

**Horizontal Scaling** ⭐ (Natural fit for NoSQL)

```
Initial: 1M users
┌─────────────┐
│   Node 1    │
│  1M users   │
└─────────────┘

Growth: Add more nodes
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Node 1    │  │   Node 2    │  │   Node 3    │
│  1M users   │  │  1M users   │  │  1M users   │
└─────────────┘  └─────────────┘  └─────────────┘

Total: 3M users

Continue adding nodes as needed
```

**Comparison:**

```
SQL: Vertical Scaling
- Increase server capacity
- Has limits

NoSQL: Horizontal Scaling
- Add more nodes
- Virtually unlimited
```

---

### Property (BASE)

**BASE** = Basically Available, Soft state, Eventual consistency

```
B = Basically Available
A = (Soft state)
S = Soft State
E = Eventual Consistency
```

**NOT ACID!**

---

#### B - Basically Available

**Meaning:** Highly available

**How?**

```
Data Replication:

Server 1          Server 2          Server 3
┌─────────┐      ┌─────────┐      ┌─────────┐
│Shreyansh│      │Shreyansh│      │Shreyansh│
│ (Copy 1)│      │ (Copy 2)│      │ (Copy 3)│
└─────────┘      └─────────┘      └─────────┘

If Server 1 fails:
→ Server 2 can serve request
→ Server 3 can serve request
→ System remains available
```

**Result:**
```
✓ Highly available
✓ Data never lost
✓ System rarely down
```

---

#### S - Soft State

**Meaning:** State can change without user interaction

**How?**

```
Vector Clocks:

Server 1: Shreyansh, Clock: (S1, 1)
Server 2: Shreyansh, Clock: (S2, 2) ← Latest
Server 3: Shreyansh, Clock: (S1, 1)

Automatic Sync:
Server 1 syncs with Server 2
→ Detects Server 2 has latest data
→ Updates itself
→ No user interaction needed!

State changed automatically
```

**Result:**
```
✓ Automatic synchronization
✓ Self-updating
✓ No manual intervention
```

---

#### E - Eventual Consistency

**Meaning:** May get stale data initially, but eventually consistent

**Example:**

```
Time T0: Update on Server 1
Server 1: Data = "New" ✓
Server 2: Data = "Old" (not synced yet)
Server 3: Data = "Old" (not synced yet)

Query at T0 to Server 2:
→ Returns "Old" (Stale data)

Time T1: After sync
Server 1: Data = "New"
Server 2: Data = "New" ✓ (Synced)
Server 3: Data = "New" ✓ (Synced)

Query at T1 to Server 2:
→ Returns "New" (Latest data)
```

**Result:**
```
✓ Eventually consistent
✓ May return stale data initially
✓ Retry will get latest data
```

---

### Summary: SQL vs NoSQL Properties

```
┌──────────────┬─────────────┬─────────────────┐
│   Property   │     SQL     │     NoSQL       │
├──────────────┼─────────────┼─────────────────┤
│  Follows     │    ACID     │      BASE       │
│              │             │                 │
│ Consistency  │   Strong    │   Eventual      │
│              │             │                 │
│ Availability │   Lower     │   Very High     │
│              │             │                 │
│ Data Loss    │   Never     │   Rare (maybe)  │
└──────────────┴─────────────┴─────────────────┘
```

---

## When to Use SQL

### 1. Flexible Query Functionality

**SQL:** Supports complex, flexible queries

```
Today's requirement:
SELECT * FROM Employee
JOIN Department ON Employee.dept_id = Department.id
WHERE salary > 80000

Tomorrow's requirement:
SELECT * FROM Employee
JOIN Department ON Employee.dept_id = Department.id
JOIN Project ON Employee.id = Project.emp_id
JOIN Client ON Project.client_id = Client.id
WHERE salary > 80000 AND Client.region = 'USA'

SQL handles both easily!
```

**NoSQL:** Limited query support

```
Basic queries only:
- Get by key
- Simple filters

Complex joins:
❌ Not supported
❌ Need application-level joins
```

**Decision:**
```
Flexible queries needed → SQL ✓
Basic queries only → NoSQL ✓
```

---

### 2. Relational Data

**When data has relationships:**

```
Parent-Child Hierarchy:

Company
  ↓
Department
  ↓
Employee
  ↓
Projects

Many dependencies and relations
```

**SQL Example:**

```
┌─────────────┐
│  Company    │
└─────────────┘
       ↓ (1 to many)
┌─────────────┐
│ Department  │
└─────────────┘
       ↓ (1 to many)
┌─────────────┐
│  Employee   │
└─────────────┘
       ↓ (many to many)
┌─────────────┐
│  Projects   │
└─────────────┘

SQL maintains these relations naturally
```

**NoSQL:**
```
Data is independent
No tight coupling
Flat structure preferred
```

**Decision:**
```
Relational data with hierarchy → SQL ✓
Independent data → NoSQL ✓
```

---

### 3. Data Integrity Required

**ACID Properties Ensure:**
```
✓ No transaction loss
✓ Strong consistency
✓ Data integrity
```

**Critical Use Cases:**

**Financial Institutions:**
```
Bank Transfer:
Account A: -$100
Account B: +$100

Cannot afford:
❌ Partial transaction
❌ Data loss
❌ Inconsistency

Must use SQL (ACID)
```

**Other Examples:**
- Banking systems
- Payment gateways
- Stock trading
- Healthcare records
- Legal documents

**Decision:**
```
Data integrity critical → SQL ✓
Can tolerate some inconsistency → NoSQL ✓
```

---

### 4. Known Query Patterns

**If you know in advance:**
```
Always query by:
- User ID
- Email
- Date range
- Specific 4-5 columns

Fixed query patterns → Can use NoSQL
```

**If queries change frequently:**
```
Today: Query by name
Tomorrow: Query by department + salary
Next week: Query by project + client + region

Flexible query needs → Use SQL
```

---

## When to Use NoSQL

### 1. High Availability Required

**NoSQL Provides:**
```
✓ Distributed nature
✓ Data replication
✓ Multiple nodes
✓ Rarely down
```

**Example:**

```
Social Media Platform:
- Must be always available
- Millions of users
- Cannot afford downtime

Use NoSQL for high availability
```

---

### 2. High Performance Needed

**Fast Searching:**

```
NoSQL:
- Data in distributed nodes
- Direct key lookup
- Very fast retrieval

Example: DynamoDB
Key lookup: < 10ms
```

**When Performance Critical:**
```
✓ Real-time applications
✓ High-traffic systems
✓ Low-latency requirements
```

---

### 3. Can Afford Some Inconsistency

**Eventual Consistency Acceptable:**

```
Social Media Post:
User posts → May take few seconds to appear to friends
Acceptable delay ✓

E-commerce Product View:
Product count may be slightly off
Acceptable ✓

News Feed:
May show slightly old content
Acceptable ✓
```

**NOT Acceptable:**
```
Bank balance ❌
Payment transactions ❌
Stock prices ❌
```

**Decision:**
```
Can tolerate inconsistency → NoSQL ✓
Need strong consistency → SQL ✓
```

---

### 4. Handling Big Data

**Large Scale:**

```
Millions/Billions of records
- Easy horizontal scaling
- Add nodes as needed
- Distributed storage

NoSQL handles big data naturally
```

**Example:**
```
IoT sensor data: Billions of records
User activity logs: Millions per day
Social media posts: Massive scale

Use NoSQL
```

---

### 5. Simple Query Requirements

**Basic Operations:**

```
✓ Get by key
✓ Simple filters
✓ No complex joins
✓ No multi-table queries

NoSQL sufficient
```

**Example:**
```
Session storage:
- Get session by session_id
- Simple lookup
- No joins needed

Use NoSQL (Key-Value)
```

---

## Decision Framework

### Step-by-Step Decision Process

```
Step 1: Analyze Data Nature
├─ Relational with hierarchy? → SQL
└─ Independent, flat data? → NoSQL

Step 2: Query Requirements
├─ Complex, flexible queries? → SQL
└─ Simple, known queries? → NoSQL

Step 3: Consistency Needs
├─ Strong consistency required? → SQL
└─ Eventual consistency OK? → NoSQL

Step 4: Scalability Needs
├─ Vertical scaling acceptable? → SQL
└─ Need horizontal scaling? → NoSQL

Step 5: Availability Requirements
├─ Can tolerate some downtime? → SQL
└─ Must be always available? → NoSQL

Step 6: Data Integrity
├─ Cannot lose any transaction? → SQL
└─ Can tolerate rare data loss? → NoSQL
```

---

### Decision Matrix

```
┌─────────────────────────┬─────────┬─────────┐
│       Requirement       │   SQL   │  NoSQL  │
├─────────────────────────┼─────────┼─────────┤
│ Complex Queries         │    ✓    │    ✗    │
│ Flexible Queries        │    ✓    │    ✗    │
│ Relational Data         │    ✓    │    ✗    │
│ Data Integrity (ACID)   │    ✓    │    ✗    │
│ Strong Consistency      │    ✓    │    ✗    │
├─────────────────────────┼─────────┼─────────┤
│ High Availability       │    ✗    │    ✓    │
│ High Performance        │    ✗    │    ✓    │
│ Horizontal Scaling      │    ✗    │    ✓    │
│ Big Data                │    ✗    │    ✓    │
│ Simple Queries          │    ✓    │    ✓    │
│ Eventual Consistency OK │    ✗    │    ✓    │
└─────────────────────────┴─────────┴─────────┘
```

---

## Summary

### SQL Overview

```
Structure: Tables, Rows, Columns, Predetermined Schema
Nature: Centralized/Concentrated
Scalability: Vertical (Better fit)
Property: ACID (Strong consistency)

Use When:
✓ Complex queries needed
✓ Relational data
✓ Data integrity critical
✓ Strong consistency required

Examples: MySQL, PostgreSQL, Oracle
```

### NoSQL Overview

```
Structure: 4 Types
  1. Key-Value (DynamoDB)
  2. Document (MongoDB)
  3. Column-Wise (Cassandra)
  4. Graph (Neo4j)

Nature: Distributed
Scalability: Horizontal (Natural fit)
Property: BASE (Eventual consistency)

Use When:
✓ High availability needed
✓ High performance required
✓ Big data handling
✓ Horizontal scaling needed
✓ Eventual consistency acceptable

Examples: MongoDB, Cassandra, DynamoDB, Neo4j
```

### Quick Comparison

```
┌──────────────┬─────────────────┬─────────────────┐
│   Aspect     │      SQL        │     NoSQL       │
├──────────────┼─────────────────┼─────────────────┤
│ Structure    │ Table/Row/Col   │ 4 types         │
│ Schema       │ Predetermined   │ Flexible        │
│ Relations    │ Yes (FK/PK)     │ No              │
│ Nature       │ Centralized     │ Distributed     │
│ Scaling      │ Vertical        │ Horizontal      │
│ Property     │ ACID            │ BASE            │
│ Consistency  │ Strong          │ Eventual        │
│ Availability │ Lower           │ Very High       │
│ Queries      │ Complex/Flex    │ Simple/Basic    │
└──────────────┴─────────────────┴─────────────────┘
```

---

## Interview Tips

### Do's ✅

**1. Always Provide Reasoning**
```
❌ "I will use SQL"
✅ "I will use SQL because:
    - Data is relational (users, orders, products)
    - Need ACID properties for transactions
    - Require complex queries across tables"
```

**2. Consider Trade-offs**
```
Mention both pros and cons:
"Using SQL gives us ACID properties,
but we may need to consider vertical scaling limits
if data grows significantly"
```

**3. Match to Use Case**
```
E-commerce:
- Product catalog → NoSQL (MongoDB)
- User orders → SQL (PostgreSQL)
- Session data → NoSQL (Redis)

Show you understand different needs
```

**4. Know the 4 NoSQL Types**
```
Be ready to explain:
- Key-Value: DynamoDB
- Document: MongoDB
- Column-Wise: Cassandra
- Graph: Neo4j

And when to use each
```

### Don'ts ❌

**1. Don't Say "Anything Works"**
```
❌ "We can use SQL or NoSQL, both work"

Shows lack of understanding
```

**2. Don't Ignore Data Nature**
```
❌ Using NoSQL for highly relational data
❌ Using SQL for simple key-value lookups

Consider data structure first
```

**3. Don't Forget Consistency Requirements**
```
Financial app with NoSQL:
❌ Wrong choice (need ACID)

Social media with SQL:
❌ May not scale well
```

**4. Don't Overlook Scale**
```
Billions of records:
❌ SQL may struggle with horizontal scaling
✓ NoSQL better choice
```

### Common Interview Scenarios

**Scenario 1: Design Twitter**
```
Question: Which database?

Good Answer:
"I'll use a hybrid approach:
- User profiles: NoSQL (MongoDB)
  - Flexible schema for user data
  - High read performance
  
- Tweets: NoSQL (Cassandra)
  - Massive scale (billions of tweets)
  - Horizontal scaling needed
  
- User relationships: Graph DB (Neo4j)
  - Followers/following relationships
  - Fast traversal for recommendations"
```

**Scenario 2: Design Banking System**
```
Question: Which database?

Good Answer:
"I'll use SQL (PostgreSQL):
- ACID properties essential
- Cannot lose transactions
- Strong consistency required
- Relational data (accounts, transactions, users)
- Complex queries for reports

Trade-off: May need careful vertical scaling
but data integrity is non-negotiable"
```

**Scenario 3: Design Analytics Platform**
```
Question: Which database?

Good Answer:
"I'll use Column-Wise NoSQL (Cassandra):
- Time-series data (billions of events)
- High write throughput
- Horizontal scaling needed
- Column-based queries efficient
- Eventual consistency acceptable for analytics"
```

### Key Points to Remember

```
1. Structure matters
   - Relational → SQL
   - Flexible → NoSQL

2. Consistency is critical
   - ACID needed → SQL
   - Eventual OK → NoSQL

3. Scale appropriately
   - Vertical limits → SQL
   - Horizontal unlimited → NoSQL

4. Query complexity
   - Complex/Flexible → SQL
   - Simple/Known → NoSQL

5. Availability requirements
   - Can tolerate downtime → SQL
   - Must be always up → NoSQL
```

---

**End of Lecture**

Understanding when to use SQL vs NoSQL is crucial for system design interviews. Always justify your database choice based on data nature, consistency requirements, scalability needs, and query patterns. Remember: there's no one-size-fits-all solution - the right choice depends on your specific use case and requirements.

**Key Takeaway:** Don't just choose a database - explain WHY you're choosing it based on structure, nature, scalability, and properties (ACID vs BASE).
