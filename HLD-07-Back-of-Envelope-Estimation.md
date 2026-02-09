# HLD-07: Back of the Envelope Estimation

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What is Back of the Envelope Estimation?](#what-is-back-of-the-envelope-estimation)
3. [Why Do We Need It?](#why-do-we-need-it)
4. [Key Considerations](#key-considerations)
5. [Essential Cheat Sheets](#essential-cheat-sheets)
   - [Traffic and Storage Zeros](#traffic-and-storage-zeros)
   - [Data Type Sizes](#data-type-sizes)
   - [Quick Calculation Formula](#quick-calculation-formula)
6. [What to Compute](#what-to-compute)
7. [Facebook Estimation Example](#facebook-estimation-example)
   - [Step 1: Traffic Estimation](#step-1-traffic-estimation)
   - [Step 2: Storage Estimation](#step-2-storage-estimation)
   - [Step 3: RAM Estimation](#step-3-ram-estimation)
   - [Step 4: Number of Servers](#step-4-number-of-servers)
   - [Step 5: Trade-offs (CAP Theorem)](#step-5-trade-offs-cap-theorem)
8. [Complete Summary](#complete-summary)
9. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Back of the Envelope Estimation

**When to Use:**
- Every High Level Design (HLD) interview
- Before starting system design
- To validate design decisions

**Example:** Design Facebook

---

## What is Back of the Envelope Estimation?

### Definition

**Back of the Envelope Estimation** = Rough calculations to drive system design decisions

```
Purpose: Drive our decision for system design
         (Load balancer, CDN, servers, cache, etc.)
```

### The Problem Without Estimation

**Scenario: Design Facebook**

```
Direct approach (without estimation):
┌─────────────┐
│Load Balancer│
└─────────────┘
       │
       ▼
┌─────────────┐
│     CDN     │
└─────────────┘
       │
       ▼
┌─────────────┐
│   Servers   │
└─────────────┘
       │
       ▼
┌─────────────┐
│    Cache    │
└─────────────┘
       │
       ▼
┌─────────────┐
│  Database   │
└─────────────┘
```

**Interviewer Questions:**
- Do you really need a load balancer?
- Do you really need CDN?
- How many servers do you need?
- What should be the capacity?
- Do you really need cache?
- What is the cache storage size?

**Problem:**
```
If you cannot answer these questions:
❌ Design appears random
❌ No consideration of system constraints
❌ Could be under-utilized or over-utilized
❌ Wastage of resources
```

---

## Why Do We Need It?

### Purpose

✅ **Avoid Under-utilization**
```
Example:
Low traffic (100 requests/day)
→ Don't need load balancer
→ Don't need multiple servers
→ Wastage of resources
```

✅ **Avoid Over-utilization**
```
Example:
High traffic (1M requests/second)
→ Need load balancer
→ Need multiple servers
→ Need CDN
→ Need cache
```

✅ **Drive Design Decisions**
```
Based on numbers:
- Decide if load balancer needed
- Calculate number of servers
- Determine cache size
- Estimate storage capacity
```

### Key Principle

```
Back of the Envelope Estimation
         ↓
   Drives Decisions
         ↓
    System Design
```

---

## Key Considerations

### 1. Rough Numbers (T-Shirt Size Estimation)

**What it means:**
- High-level estimation
- Not exact or accurate
- Approximate values

**Example:**
```
Estimated: 250 million users
Real Facebook: Different actual number

Purpose: Get ballpark figures, not exact numbers
```

### 2. Don't Spend Much Time

**Why?**

```
Interview Reality:
- 95-98% of time → Interviewer expects scalable design
- Scalable design → Always needs CDN, cache, load balancer, multiple servers

Result:
- Numbers don't impact design much in interviews
- Same design components needed regardless
```

**Time Limit:**
```
⏰ Maximum: 10 minutes
⏰ Ideal: Less than 10 minutes

Ask interviewer first:
"Do you need back of the envelope estimation?"
```

### 3. Keep Assumption Values Simple

**Good Assumptions:**
```
✅ 10 million users
✅ 100 million users
✅ 500 million users
✅ 1 billion users
```

**Bad Assumptions:**
```
❌ 275 million users
❌ 435 million users
❌ 87.5 million users

Why bad?
- Difficult to compute
- Hard to remember
- Complex calculations
```

**Rule:**
```
Use multiples of 10:
- 10, 100, 1000
- 10M, 100M, 1B
- 500, 5000, 50000

Easy to compute and remember
```

---

## Essential Cheat Sheets

### Traffic and Storage Zeros

**MUST REMEMBER - Keep this handy during interviews!**

```
┌─────────────┬──────────┬─────────────┐
│   Zeros     │ Traffic  │   Storage   │
├─────────────┼──────────┼─────────────┤
│ 3 zeros     │ Thousand │ KB          │
│ 6 zeros     │ Million  │ MB          │
│ 9 zeros     │ Billion  │ GB          │
│ 12 zeros    │ Trillion │ TB          │
│ 15 zeros    │ Quadrill │ PB          │
└─────────────┴──────────┴─────────────┘

Pattern: Add 3 zeros each time
```

**Examples:**

```
Traffic:
1,000 = Thousand (K)
1,000,000 = Million (M)
1,000,000,000 = Billion (B)
1,000,000,000,000 = Trillion (T)

Storage:
1,000 bytes = KB (Kilobyte)
1,000,000 bytes = MB (Megabyte)
1,000,000,000 bytes = GB (Gigabyte)
1,000,000,000,000 bytes = TB (Terabyte)
1,000,000,000,000,000 bytes = PB (Petabyte)
```

---

### Data Type Sizes

**Assumptions for Estimation:**

```
┌──────────────────┬─────────────┐
│   Data Type      │    Size     │
├──────────────────┼─────────────┤
│ Character        │ 2 bytes     │
│ Long/Double      │ 8 bytes     │
│ Image (average)  │ 300 KB      │
│ Video (1 min)    │ ~10 MB      │
└──────────────────┴─────────────┘
```

**Character Details:**
```
ASCII: 1 byte
Unicode: 2 bytes

Assumption: Use 2 bytes (Unicode)
```

---

### Quick Calculation Formula

**VERY IMPORTANT - Memorize This!**

```
Formula:
X million × Y MB = (X × Y) TB

Why?
Million = 6 zeros
MB = 6 zeros
Total = 12 zeros = TB
```

### Examples

**Example 1:**
```
5 million users × 2 KB = ?

Step 1: Multiply values
5 × 2 = 10

Step 2: Count zeros
Million = 6 zeros
KB = 3 zeros
Total = 9 zeros

Step 3: Find unit
9 zeros = GB

Answer: 10 GB
```

**Example 2:**
```
10 million users × 3 MB = ?

Step 1: Multiply values
10 × 3 = 30

Step 2: Count zeros
Million = 6 zeros
MB = 6 zeros
Total = 12 zeros

Step 3: Find unit
12 zeros = TB

Answer: 30 TB
```

**Example 3:**
```
250 million users × 1 KB = ?

Step 1: Multiply values
250 × 1 = 250

Step 2: Count zeros
Million = 6 zeros
KB = 3 zeros
Total = 9 zeros

Step 3: Find unit
9 zeros = GB

Answer: 250 GB
```

---

## What to Compute

### Three Main Things

```
1. Number of Servers
   ↓
2. RAM (Memory)
   ↓
3. Storage Capacity
   ↓
4. Trade-offs (CAP Theorem)
```

**Why These Three?**
- Most important resources in system design
- Drive major design decisions
- Impact scalability

**Note:** You can compute more, but these are essential

---

## Facebook Estimation Example

### Step 1: Traffic Estimation

#### Assumptions

```
Total Users: 1 billion (1B)

Daily Active Users (DAU): 25% of total users
= 25% of 1B
= 250 million users
```

#### User Activity

```
Per user per day:
- Read operations: 5
- Write operations: 2
Total queries per user: 7
```

#### Queries Per Second (QPS)

```
Step 1: Total daily queries
250 million users × 7 queries = 1,750 million queries/day

Step 2: Seconds in a day
60 seconds × 60 minutes × 24 hours = 86,400 seconds

Round off to: 100,000 seconds (1 lakh seconds)
(Easier to compute)

Step 3: QPS calculation
1,750 million ÷ 100,000 = 17,500 queries/second

Round off to: 18K queries/second
```

**Result:**
```
Traffic Estimation: 18,000 queries per second (18K QPS)
```

---

### Step 2: Storage Estimation

#### Assumptions

```
Every user does:
- 2 posts per day
- Each post: 250 characters

10% of users:
- Upload 1 image
- Each image: 300 KB
```

#### Post Storage Calculation

**Step 1: Storage per post**
```
1 post = 250 characters
1 character = 2 bytes
1 post = 250 × 2 = 500 bytes
```

**Step 2: Storage for 2 posts**
```
2 posts = 500 bytes × 2 = 1,000 bytes = 1 KB
```

**Step 3: Daily storage for all users**
```
250 million users × 1 KB = ?

Using formula:
Million = 6 zeros
KB = 3 zeros
Total = 9 zeros = GB

250 × 1 = 250 GB per day
```

#### Image Storage Calculation

**Step 1: Users uploading images**
```
10% of 250 million = 25 million users
```

**Step 2: Daily storage for images**
```
25 million users × 300 KB = ?

Using formula:
Million = 6 zeros
KB = 3 zeros
Total = 9 zeros = GB

25 × 300 = 7,500 GB

Convert to TB:
7,500 GB = 7.5 TB ≈ 8 TB per day
```

#### Storage for 5 Years

**Assumptions:**
```
5 years ≈ 2,000 days
(Rough estimate: 365 × 5 = 1,825 ≈ 2,000)
```

**Post storage for 5 years:**
```
250 GB/day × 2,000 days = 500,000 GB = 500 TB
```

**Image storage for 5 years:**
```
8 TB/day × 2,000 days = 16,000 TB = 16 PB
```

**Result:**
```
Storage Capacity (5 years):
- Posts: 500 TB
- Images: 16 PB
```

---

### Step 3: RAM Estimation

#### Cache Assumptions

```
For each user:
- Cache last 5 posts
- Each post: 500 bytes
- Total per user: 5 × 500 = 2,500 bytes
```

**Round off:**
```
2,500 bytes ≈ 3,000 bytes = 3 KB
```

#### Total RAM Required

```
250 million users × 3 KB = ?

Using formula:
Million = 6 zeros
KB = 3 zeros
Total = 9 zeros = GB

250 × 3 = 750 GB
```

#### Number of Cache Machines

**Assumption:**
```
1 machine can hold: 75 GB of RAM
```

**Calculation:**
```
Total RAM needed: 750 GB
RAM per machine: 75 GB

Number of machines = 750 ÷ 75 = 10 machines
```

**Result:**
```
RAM Estimation:
- Total RAM: 750 GB
- Cache machines: 10
```

---

### Step 4: Number of Servers

#### Latency Assumption

```
Target: 95% of requests served in 500 milliseconds
```

#### Server Capacity

**Assumptions:**
```
1 server has: 50 threads
Latency: 500 ms per request

Calculation:
1 second = 2 × 500 ms
1 thread can handle: 2 requests/second
1 server (50 threads) can handle: 50 × 2 = 100 requests/second
```

#### Total Servers Needed

```
Total QPS: 18,000 queries/second
Per server capacity: 100 queries/second

Number of servers = 18,000 ÷ 100 = 180 servers
```

**Result:**
```
Application Servers: 180
```

---

### Step 5: Trade-offs (CAP Theorem)

#### CAP Theorem Recap

```
        C
       / \
      /   \
     /     \
    A ───── P

C = Consistency
A = Availability
P = Partition Tolerance

Rule: Can only achieve 2 out of 3
```

#### Possible Combinations

```
1. CA (Consistency + Availability)
   - Drop Partition Tolerance

2. CP (Consistency + Partition Tolerance)
   - Drop Availability

3. AP (Availability + Partition Tolerance)
   - Drop Consistency
```

#### Facebook's Choice: AP

**Selected:**
```
✅ Availability
✅ Partition Tolerance
❌ Consistency (eventual consistency)
```

**Reasoning:**

**Availability:**
```
Even if one node is down:
→ System should still serve requests
→ Users can still access Facebook
```

**Partition Tolerance:**
```
Even if network breaks between nodes:
→ System should still function
→ No complete downtime
```

**Consistency (Dropped):**
```
Eventual consistency is acceptable:
→ User posts may take time to propagate
→ Not critical if friend sees post after few seconds
→ Availability more important than immediate consistency
```

**Trade-off Decision:**
```
For Facebook:
- Availability > Consistency
- Better to show slightly stale data
- Than to show error page
```

---

## Complete Summary

### Final Estimates for Facebook

```
┌─────────────────────────┬──────────────────┐
│      Component          │      Value       │
├─────────────────────────┼──────────────────┤
│ Total Users             │ 1 Billion        │
│ Daily Active Users      │ 250 Million      │
│ Queries Per Second      │ 18K QPS          │
├─────────────────────────┼──────────────────┤
│ Storage (Posts/5 yrs)   │ 500 TB           │
│ Storage (Images/5 yrs)  │ 16 PB            │
├─────────────────────────┼──────────────────┤
│ RAM Required            │ 750 GB           │
│ Cache Machines          │ 10               │
├─────────────────────────┼──────────────────┤
│ Application Servers     │ 180              │
│ Latency (95%)           │ 500 ms           │
├─────────────────────────┼──────────────────┤
│ CAP Choice              │ AP               │
│ Trade-off               │ Drop Consistency │
└─────────────────────────┴──────────────────┘
```

### Calculation Flow

```
Step 1: Traffic Estimation
├─ Total users
├─ Daily active users
└─ Queries per second

Step 2: Storage Estimation
├─ Post storage (per day)
├─ Image storage (per day)
└─ Total storage (5 years)

Step 3: RAM Estimation
├─ Cache per user
├─ Total RAM needed
└─ Number of cache machines

Step 4: Server Estimation
├─ Server capacity
├─ Total QPS
└─ Number of servers

Step 5: Trade-offs
└─ CAP theorem decision
```

---

## Interview Tips

### Do's ✅

**1. Ask First**
```
"Do you need back of the envelope estimation for this design?"

If yes → Proceed
If no → Skip to design
```

**2. Keep It Simple**
```
Use round numbers:
✅ 10M, 100M, 1B
✅ 500, 5000
✅ 100K, 1M

Avoid:
❌ 87.5M
❌ 435K
❌ 2.75B
```

**3. Use Cheat Sheet**
```
Always remember:
- 3 zeros = K (Thousand/KB)
- 6 zeros = M (Million/MB)
- 9 zeros = B (Billion/GB)
- 12 zeros = T (Trillion/TB)
```

**4. Show Your Work**
```
Don't just say "180 servers"

Show:
- QPS = 18K
- Per server = 100 QPS
- Servers = 18K ÷ 100 = 180
```

**5. State Assumptions Clearly**
```
"I'm assuming:"
- 1 character = 2 bytes
- 1 image = 300 KB
- 25% daily active users
```

### Don'ts ❌

**1. Don't Spend Too Much Time**
```
❌ More than 10 minutes
✅ Less than 10 minutes

Why?
- Won't impact design much
- Same components needed anyway
```

**2. Don't Be Too Precise**
```
❌ "Exactly 17,361 queries per second"
✅ "Approximately 18K queries per second"

It's rough estimation, not exact calculation
```

**3. Don't Skip Trade-offs**
```
Always mention CAP theorem:
- Shows you understand distributed systems
- Demonstrates design thinking
```

**4. Don't Forget Units**
```
❌ "Storage is 500"
✅ "Storage is 500 TB"

Always specify units
```

### Quick Reference Formula

```
X million × Y MB = (X × Y) TB

Examples:
- 5M × 2KB = 10GB
- 10M × 3MB = 30TB
- 250M × 1KB = 250GB
```

### Time Breakdown

```
Total: < 10 minutes

Traffic: 2 minutes
Storage: 3 minutes
RAM: 2 minutes
Servers: 2 minutes
Trade-offs: 1 minute
```

### Common Interview Scenarios

**Scenario 1: Low Traffic**
```
100 requests/day
→ Don't need load balancer
→ Single server sufficient
→ No CDN needed
```

**Scenario 2: Medium Traffic**
```
10K requests/second
→ Need load balancer
→ Multiple servers (10-20)
→ Cache recommended
```

**Scenario 3: High Traffic**
```
100K+ requests/second
→ Need load balancer
→ Many servers (100+)
→ CDN essential
→ Cache essential
→ Multiple data centers
```

### Key Reminders

```
1. It's ROUGH estimation
   - Not exact numbers
   - Real numbers will differ
   - Purpose: Ballpark figures

2. It DRIVES decisions
   - Helps justify design choices
   - Shows you consider constraints
   - Demonstrates systematic thinking

3. It's NOT the design
   - Just preparation step
   - Real design comes after
   - Don't spend too much time

4. Always ROUND OFF
   - Makes calculations easier
   - Easier to remember
   - Acceptable in interviews
```

### What Interviewers Look For

```
✅ Systematic approach
✅ Clear assumptions
✅ Simple calculations
✅ Awareness of trade-offs
✅ Understanding of scale
✅ Justification for design choices

❌ Exact precision
❌ Complex calculations
❌ Too much time spent
❌ Random guesses
```

---

**End of Lecture**

Back of the envelope estimation is a crucial skill for system design interviews. It helps you make informed decisions about your architecture and demonstrates your ability to think about scale and constraints. Remember: keep it simple, quick, and use it to drive your design decisions!

**Key Takeaway:** This is rough estimation to validate design choices, not exact calculation. Spend less than 10 minutes, use simple numbers, and always state your assumptions clearly.
