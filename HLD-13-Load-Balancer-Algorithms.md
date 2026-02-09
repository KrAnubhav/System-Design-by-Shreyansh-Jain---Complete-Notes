# HLD-13: Load Balancer & Algorithms

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What is Load Balancer](#what-is-load-balancer)
3. [Types of Load Balancers](#types-of-load-balancers)
4. [Static Load Balancing Algorithms](#static-load-balancing-algorithms)
5. [Dynamic Load Balancing Algorithms](#dynamic-load-balancing-algorithms)
6. [Summary](#summary)
7. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Load Balancer & Algorithms

**Coverage:**
- ✅ What is load balancer
- ✅ L4 vs L7 load balancers
- ✅ Static algorithms (Round Robin, Weighted Round Robin, IP Hash)
- ✅ Dynamic algorithms (Least Connection, Weighted Least Connection, Least Response Time)

---

## What is Load Balancer

### Basic Concept

```
┌──────────┐
│ Client 1 │──┐
└──────────┘  │
              │
┌──────────┐  │    ┌─────────────┐
│ Client 2 │──┼───►│    Load     │
└──────────┘  │    │  Balancer   │
              │    └─────────────┘
┌──────────┐  │           │
│ Client N │──┘           │
└──────────┘              │
                          ├──────────┬──────────┐
                          ▼          ▼          ▼
                     ┌────────┐ ┌────────┐ ┌────────┐
                     │Server 1│ │Server 2│ │Server N│
                     └────────┘ └────────┘ └────────┘
```

**Purpose:**
```
Distribute traffic across multiple servers
→ No single server gets overburdened
→ Better resource utilization
→ High availability
```

**Main Function:**
- Distribute traffic appropriately
- Prevent server overload
- Ensure even distribution

**Additional Capabilities:**
- Logging
- Caching (L7)
- Health checks
- SSL termination

---

## Types of Load Balancers

### L4 Load Balancer (Network/Transport Layer)

```
OSI Model:
├─ Layer 7: Application
├─ Layer 6: Presentation
├─ Layer 5: Session
├─ Layer 4: Transport ← L4 Load Balancer
├─ Layer 3: Network
├─ Layer 2: Data Link
└─ Layer 1: Physical
```

**What L4 Can Access:**
```
✓ Source IP address
✓ Destination IP address
✓ Source port
✓ Destination port
✓ TCP/UDP protocol

✗ Cannot read headers
✗ Cannot read data
✗ Cannot read response
```

**Characteristics:**
- Works at transport layer
- Fast (less processing)
- Simple routing decisions
- Based on network information only

---

### L7 Load Balancer (Application Layer)

```
OSI Model:
├─ Layer 7: Application ← L7 Load Balancer
├─ Layer 6: Presentation
├─ Layer 5: Session
├─ Layer 4: Transport
├─ Layer 3: Network
├─ Layer 2: Data Link
└─ Layer 1: Physical
```

**What L7 Can Access:**
```
✓ HTTP headers
✓ Session data
✓ Cookies
✓ Request data
✓ Response data
✓ URLs
✓ Content type

Plus all L4 capabilities
```

**Characteristics:**
- Works at application layer
- Advanced routing decisions
- Can do caching (reads response)
- Slower (more processing)
- More features

---

### L4 vs L7 Comparison

```
┌─────────────────┬─────────────┬─────────────┐
│    Feature      │     L4      │     L7      │
├─────────────────┼─────────────┼─────────────┤
│ Layer           │ Transport   │ Application │
│ Speed           │ Faster      │ Slower      │
│ Complexity      │ Simple      │ Advanced    │
│ IP/Port         │     ✓       │      ✓      │
│ Headers         │     ✗       │      ✓      │
│ Data            │     ✗       │      ✓      │
│ Caching         │     ✗       │      ✓      │
│ Content-based   │     ✗       │      ✓      │
│ routing         │             │             │
└─────────────────┴─────────────┴─────────────┘
```

---

## Static Load Balancing Algorithms

**Static** = No dynamic computation, uses predefined rules

### 1. Round Robin

#### How It Works

```
Requests: 1, 2, 3, 4, 5, 6, 7, 8

┌──────────┐
│ Requests │
│ 1,2,3,4, │
│ 5,6,7,8  │
└──────────┘
      │
      ▼
┌─────────────┐
│    Load     │
│  Balancer   │
└─────────────┘
      │
      ├──────────────┬──────────────┐
      ▼              ▼              ▼
┌─────────┐    ┌─────────┐    ┌─────────┐
│Server 1 │    │Server 2 │    │Server 3 │
│Req: 1,4,7│    │Req: 2,5,8│    │Req: 3,6 │
└─────────┘    └─────────┘    └─────────┘

Distribution:
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2
Request 6 → Server 3
Request 7 → Server 1
Request 8 → Server 2
```

**Pattern:** Cyclic distribution

---

#### Advantages

```
1. Very easy to implement
   - Simple algorithm
   - No complex logic

2. Equal load distribution
   - 10 requests → 5 to S1, 5 to S2
   - Guaranteed equal distribution
   - Predictable
```

---

#### Disadvantages

```
Problem: Treats all servers equally

Scenario:
Server 1: 10x capacity (powerful)
Server 2: 1x capacity (weak)

Round Robin:
10 requests → 5 to S1, 5 to S2

Result:
Server 1: Underutilized (can handle more)
Server 2: Overloaded (5 requests too much)
         May crash ✗

Issue: Doesn't consider server capacity
```

**Visual:**

```
┌─────────────┐         ┌─────────────┐
│  Server 1   │         │  Server 2   │
│             │         │             │
│ Capacity:   │         │ Capacity:   │
│   10x       │         │    1x       │
│             │         │             │
│ Requests: 5 │         │ Requests: 5 │
│ Status: OK  │         │ Status: ✗   │
│ (Can handle │         │ (Overloaded)│
│  more)      │         │             │
└─────────────┘         └─────────────┘
```

---

### 2. Weighted Round Robin

#### How It Works

```
Server 1: Weight = 3 (3x capacity)
Server 2: Weight = 1 (1x capacity)

Weight represents server capacity

Requests: 1, 2, 3, 4, 5, 6, 7, 8

Distribution:
Request 1 → Server 1 (weight 3, count 1)
Request 2 → Server 1 (weight 3, count 2)
Request 3 → Server 1 (weight 3, count 3)
Request 4 → Server 2 (weight 1, count 1)
Request 5 → Server 1 (weight 3, count 1)
Request 6 → Server 1 (weight 3, count 2)
Request 7 → Server 1 (weight 3, count 3)
Request 8 → Server 2 (weight 1, count 1)

Pattern: 3 to S1, 1 to S2, repeat
```

**Visual:**

```
┌──────────┐
│ Requests │
│1,2,3,4,5,│
│ 6,7,8    │
└──────────┘
      │
      ▼
┌─────────────┐
│    Load     │
│  Balancer   │
│             │
│ S1: Weight=3│
│ S2: Weight=1│
└─────────────┘
      │
      ├────────────────┬────────────────┐
      ▼                ▼                ▼
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Server 1 │     │ Server 2 │     │          │
│Weight: 3 │     │Weight: 1 │     │          │
│          │     │          │     │          │
│Req: 1,2,3│     │Req: 4    │     │          │
│    5,6,7 │     │    8     │     │          │
│          │     │          │     │          │
│6 requests│     │2 requests│     │          │
└──────────┘     └──────────┘     └──────────┘

Ratio: 3:1 (matches capacity)
```

---

#### Advantages

```
1. Low capacity servers protected
   - Get fewer requests
   - Won't be overloaded
   - Based on capacity

2. Easy to implement
   - Weights are static
   - No dynamic computation
   - Configured at startup
```

---

#### Disadvantages

```
Problem: Doesn't consider request processing time

Scenario:
Server 1 (Weight 3): Gets requests 1,2,3
Server 2 (Weight 1): Gets request 4

Request processing times:
Request 1: 50ms  (fast)
Request 2: 30ms  (fast)
Request 3: 40ms  (fast)
Request 4: 10 seconds (slow!)

Next round:
Request 5 → Server 1 (50ms)
Request 6 → Server 1 (30ms)
Request 7 → Server 1 (40ms)
Request 8 → Server 2 (20 seconds!)

Server 2 status:
- Still processing request 4 (10s)
- Now gets request 8 (20s)
- Queue building up
- Overloaded! ✗

Server 1 status:
- All requests done quickly
- Idle most of the time
- Underutilized
```

**Visual:**

```
Timeline:

Server 1:
[Req1:50ms][Req2:30ms][Req3:40ms][Idle...][Req5:50ms][Req6:30ms][Req7:40ms]

Server 2:
[Req4: 10 seconds..................][Req8: 20 seconds....................]
                                    ↑
                                Queue building
                                Overloaded!
```

**Issue:** High processing requests can overload low capacity servers

---

### 3. IP Hash

#### How It Works

```
Each client has source IP address

Client 1: 192.168.1.10
Client 2: 192.168.1.20
Client 3: 192.168.1.30

Load Balancer:
1. Takes source IP
2. Computes hash
3. Maps to server

Hash(192.168.1.10) % 2 = 0 → Server 1
Hash(192.168.1.20) % 2 = 1 → Server 2
Hash(192.168.1.30) % 2 = 0 → Server 1

Same IP → Same hash → Same server
```

**Visual:**

```
┌──────────────┐
│  Client 1    │
│192.168.1.10  │
└──────────────┘
      │
      ▼
┌─────────────────┐
│  Load Balancer  │
│                 │
│ Hash(IP) % 2    │
│ = 0             │
└─────────────────┘
      │
      ▼
┌──────────────┐
│  Server 1    │
└──────────────┘

Every request from Client 1 → Server 1
```

---

#### Advantages

```
Use case: Session persistence

Scenario: Shopping cart

Client 1 adds items:
Request 1 → Server 1 (add item A)
Request 2 → Server 1 (add item B)
Request 3 → Server 1 (checkout)

All requests to same server ✓
Session maintained ✓
Cart data consistent ✓
```

**Benefit:** Same client → Same server (session affinity)

---

#### Disadvantages

##### 1. Proxy Problem

```
WITHOUT Proxy:
┌──────────┐  ┌──────────┐  ┌──────────┐
│Client 1  │  │Client 2  │  │Client 3  │
│192.1.1.10│  │192.1.1.20│  │192.1.1.30│
└──────────┘  └──────────┘  └──────────┘
      │             │             │
      └─────────────┼─────────────┘
                    ▼
              ┌─────────────┐
              │Load Balancer│
              └─────────────┘
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
    ┌────────┐            ┌────────┐
    │Server 1│            │Server 2│
    └────────┘            └────────┘

Different IPs → Different servers ✓


WITH Proxy (Forward Proxy):
┌──────────┐  ┌──────────┐  ┌──────────┐
│Client 1  │  │Client 2  │  │Client 3  │
│192.1.1.10│  │192.1.1.20│  │192.1.1.30│
└──────────┘  └──────────┘  └──────────┘
      │             │             │
      └─────────────┼─────────────┘
                    ▼
              ┌──────────┐
              │  Proxy   │
              │162.1.0.1 │ ← All requests from this IP
              └──────────┘
                    │
                    ▼
              ┌─────────────┐
              │Load Balancer│
              │             │
              │Hash(162.1.0.1)│
              │Always same  │
              └─────────────┘
                    │
                    ▼
              ┌────────┐
              │Server 1│ ← All requests here!
              └────────┘
              
              ┌────────┐
              │Server 2│ ← Never used ✗
              └────────┘

Problem: All clients appear as same IP
Result: All requests to one server
Server 1 overloaded ✗
```

##### 2. Unequal Distribution

```
Hash function doesn't guarantee equal distribution

Example:
10 clients with different IPs

After hashing:
Server 1: 7 requests
Server 2: 3 requests

Unequal! ✗
```

---

### Summary: Static Algorithms

```
┌──────────────┬────────────┬────────────┬────────────┐
│  Algorithm   │Round Robin │  Weighted  │  IP Hash   │
│              │            │Round Robin │            │
├──────────────┼────────────┼────────────┼────────────┤
│ Distribution │ Equal      │ By weight  │ By hash    │
│ Capacity     │ Ignored    │ Considered │ Ignored    │
│ Session      │ No         │ No         │ Yes        │
│ Persistence  │            │            │            │
│ Complexity   │ Very low   │ Low        │ Low        │
│ Use Case     │ Equal      │ Different  │ Session    │
│              │ servers    │ capacity   │ affinity   │
└──────────────┴────────────┴────────────┴────────────┘
```

---

## Dynamic Load Balancing Algorithms

**Dynamic** = Computes metrics in real-time

### 1. Least Connection

#### How It Works

```
Tracks active connections per server

Server 1: 2 active connections
Server 2: 1 active connection
Server 3: 3 active connections

New request arrives:
Load balancer checks: Which has least connections?
Answer: Server 2 (1 connection)
Route request to Server 2 ✓
```

**Visual:**

```
Current State:
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│  Server 1    │         │  Server 2    │         │  Server 3    │
│              │         │              │         │              │
│ Connections: │         │ Connections: │         │ Connections: │
│      2       │         │      1       │         │      3       │
│              │         │              │         │              │
│ Client 1 ──┐ │         │ Client 2 ─┐  │         │ Client 4 ──┐ │
│ Client 3 ──┘ │         │           │  │         │ Client 5 ──┤ │
│              │         │           │  │         │ Client 6 ──┘ │
└──────────────┘         └──────────────┘         └──────────────┘

New request arrives:

Load Balancer:
- Server 1: 2 connections
- Server 2: 1 connection ← Least!
- Server 3: 3 connections

Route to Server 2 ✓

After routing:
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│  Server 1    │         │  Server 2    │         │  Server 3    │
│ Connections: │         │ Connections: │         │ Connections: │
│      2       │         │      2       │         │      3       │
└──────────────┘         └──────────────┘         └──────────────┘
```

---

#### Advantages

```
1. Dynamic load consideration
   - Checks real-time load
   - Not static rules
   - Adapts to current state

2. Less chance of overload (equal capacity)
   - Distributes based on actual load
   - Balances connections
   - Fair distribution
```

---

#### Disadvantages

##### 1. Active Connection ≠ Traffic

```
Problem: TCP connection can be active but idle

Scenario:
Server 1: 2 active connections
  - Client 1: 100 requests/sec
  - Client 3: 100 requests/sec
  - Total: 200 requests/sec

Server 2: 1 active connection
  - Client 2: 500 requests/sec
  - Total: 500 requests/sec

Load Balancer sees:
Server 1: 2 connections (looks busy)
Server 2: 1 connection (looks free)

Reality:
Server 1: 200 req/sec (less load)
Server 2: 500 req/sec (more load!)

New request routes to Server 2 ✗
Server 2 gets more overloaded!
```

**Visual:**

```
┌──────────────────────┐         ┌──────────────────────┐
│     Server 1         │         │     Server 2         │
│                      │         │                      │
│ Connections: 2       │         │ Connections: 1       │
│                      │         │                      │
│ Client 1: 100 req/s  │         │ Client 2: 500 req/s  │
│ Client 3: 100 req/s  │         │                      │
│ Total: 200 req/s     │         │ Total: 500 req/s     │
│                      │         │                      │
│ Actual Load: Low     │         │ Actual Load: High    │
│ LB thinks: High      │         │ LB thinks: Low       │
└──────────────────────┘         └──────────────────────┘
                                          ↑
                                   New request goes here ✗
                                   (Wrong decision!)
```

##### 2. Doesn't Consider Capacity

```
Server 1: 10x capacity, 2 connections
Server 2: 1x capacity, 1 connection

Least Connection routes to Server 2
But Server 1 can handle much more!

Result: Suboptimal distribution
```

---

### 2. Weighted Least Connection

#### How It Works

```
Combines weights + active connections

Server 1: Weight = 10, Active connections = 2
Server 2: Weight = 1, Active connections = 1

Formula: Ratio = Active Connections / Weight

Server 1: 2 / 10 = 0.2
Server 2: 1 / 1 = 1.0

Route to minimum ratio → Server 1 (0.2) ✓
```

**Detailed Example:**

```
Initial State:
Server 1: Weight=10, Connections=2
Server 2: Weight=1, Connections=1

New request arrives:

Calculate ratios:
Server 1: 2/10 = 0.2
Server 2: 1/1 = 1.0

Minimum: 0.2 (Server 1)
Route to Server 1 ✓

After routing:
Server 1: Weight=10, Connections=3
Server 2: Weight=1, Connections=1

Next request:

Calculate ratios:
Server 1: 3/10 = 0.3
Server 2: 1/1 = 1.0

Minimum: 0.3 (Server 1)
Route to Server 1 ✓

After routing:
Server 1: Weight=10, Connections=4
Server 2: Weight=1, Connections=1

Next request:

Calculate ratios:
Server 1: 4/10 = 0.4
Server 2: 1/1 = 1.0

Minimum: 0.4 (Server 1)
Route to Server 1 ✓
```

**Benefits:**
- Considers both capacity and load
- Better distribution
- Protects weak servers

---

### 3. Least Response Time

#### Concept: TTFB (Time To First Byte)

```
TTFB = Time interval between:
  - Sending request
  - Receiving first byte of response

Example:
T0: Send request
T1: Receive first byte
TTFB = T1 - T0
```

**Visual:**

```
Client                    Server
  │                         │
  │──── Request ───────────►│ T0
  │                         │
  │                         │ Processing...
  │                         │
  │◄─── First byte ─────────│ T1
  │                         │
  │◄─── Rest of data ───────│
  │                         │

TTFB = T1 - T0
```

---

#### How It Works

```
Each server tracks:
1. Active connections
2. TTFB (response time)

Formula: Score = Active Connections × TTFB

Route to server with minimum score
```

**Detailed Example:**

```
Initial State:
Server 1: Connections=3, TTFB=2ms
Server 2: Connections=1, TTFB=4ms
Server 3: Connections=0, TTFB=2ms

Request 1 arrives:

Calculate scores:
Server 1: 3 × 2 = 6
Server 2: 1 × 4 = 4
Server 3: 0 × 2 = 0 ← Minimum!

Route to Server 3 ✓

After routing:
Server 1: Connections=3, TTFB=2ms
Server 2: Connections=1, TTFB=4ms
Server 3: Connections=1, TTFB=2ms

Request 2 arrives:

Calculate scores:
Server 1: 3 × 2 = 6
Server 2: 1 × 4 = 4
Server 3: 1 × 2 = 2 ← Minimum!

Route to Server 3 ✓

After routing:
Server 1: Connections=3, TTFB=2ms
Server 2: Connections=1, TTFB=4ms
Server 3: Connections=2, TTFB=2ms

Request 3 arrives:

Calculate scores:
Server 1: 3 × 2 = 6
Server 2: 1 × 4 = 4 ← Minimum!
Server 3: 2 × 2 = 4 ← Also minimum!

Tie! Use Round Robin:
Route to Server 2 (first in tie)

After routing:
Server 1: Connections=3, TTFB=2ms
Server 2: Connections=2, TTFB=4ms
Server 3: Connections=2, TTFB=2ms

Request 4 arrives:

Calculate scores:
Server 1: 3 × 2 = 6
Server 2: 2 × 4 = 8
Server 3: 2 × 2 = 4 ← Minimum!

Route to Server 3 ✓
```

---

#### Tie-Breaking Rule

```
If multiple servers have same score:
→ Use Round Robin

Example:
Server 2: Score = 4
Server 3: Score = 4

First tie: Route to Server 2
Next tie: Route to Server 3
Next tie: Route to Server 2
...
```

---

#### How TTFB is Measured

```
Load Balancer actively monitors:

1. Send health check request
2. Measure response time
3. Update TTFB dynamically

Example:
LB → Server 1: Health check (T0)
Server 1 → LB: Response (T1)
TTFB = T1 - T0 = 2ms

LB → Server 2: Health check (T0)
Server 2 → LB: Response (T1)
TTFB = T1 - T0 = 4ms

Continuous monitoring
Real-time updates
```

---

### Summary: Dynamic Algorithms

```
┌──────────────┬────────────┬────────────┬────────────┐
│  Algorithm   │   Least    │  Weighted  │   Least    │
│              │ Connection │   Least    │  Response  │
│              │            │ Connection │    Time    │
├──────────────┼────────────┼────────────┼────────────┤
│ Metric       │ Connections│Connections │Connections │
│              │            │ + Weight   │ + TTFB     │
│              │            │            │            │
│ Dynamic      │    Yes     │    Yes     │    Yes     │
│              │            │            │            │
│ Capacity     │    No      │    Yes     │    No      │
│ Aware        │            │            │            │
│              │            │            │            │
│ Response     │    No      │    No      │    Yes     │
│ Time         │            │            │            │
│              │            │            │            │
│ Complexity   │    Low     │   Medium   │    High    │
│              │            │            │            │
│ Best For     │Equal       │Different   │Performance │
│              │servers     │capacity    │sensitive   │
└──────────────┴────────────┴────────────┴────────────┘
```

---

## Summary

### Algorithm Categories

```
STATIC ALGORITHMS:
├─ Round Robin
│  └─ Equal distribution, ignores capacity
├─ Weighted Round Robin
│  └─ Distribution by weight, ignores processing time
└─ IP Hash
   └─ Session persistence, proxy issues

DYNAMIC ALGORITHMS:
├─ Least Connection
│  └─ Based on active connections, ignores traffic
├─ Weighted Least Connection
│  └─ Connections + capacity, better distribution
└─ Least Response Time
   └─ Connections + TTFB, performance-aware
```

### Decision Matrix

```
Choose Algorithm Based On:

Equal Servers + Simple:
→ Round Robin

Different Capacity + Simple:
→ Weighted Round Robin

Session Persistence Needed:
→ IP Hash

Equal Servers + Dynamic:
→ Least Connection

Different Capacity + Dynamic:
→ Weighted Least Connection

Performance Critical:
→ Least Response Time
```

### Key Concepts

```
1. Load Balancer Purpose:
   - Distribute traffic
   - Prevent overload
   - High availability

2. L4 vs L7:
   - L4: Fast, network-level
   - L7: Advanced, application-level

3. Static vs Dynamic:
   - Static: Predefined rules
   - Dynamic: Real-time metrics

4. Trade-offs:
   - Simple ↔ Advanced
   - Fast ↔ Accurate
   - Static ↔ Dynamic
```

---

## Interview Tips

### Common Questions

**Q1: "Explain Round Robin and its limitations"**

```
Answer:
"Round Robin distributes requests in cyclic order.

How it works:
- Request 1 → Server 1
- Request 2 → Server 2
- Request 3 → Server 1
- Repeats...

Advantages:
✓ Simple to implement
✓ Equal distribution

Limitations:
✗ Ignores server capacity
✗ Treats 10x and 1x servers equally
✗ Low capacity server may crash

Example:
Server 1: 10x capacity → Gets 5 requests (underutilized)
Server 2: 1x capacity → Gets 5 requests (overloaded)

Solution: Use Weighted Round Robin"
```

**Q2: "When would you use IP Hash?"**

```
Answer:
"IP Hash when session persistence is critical.

Use case: Shopping cart
- User adds items across multiple requests
- All requests must go to same server
- Session data maintained

How it works:
- Hash(Client IP) → Server
- Same IP → Same hash → Same server

Limitation:
- Proxy problem: All clients appear as same IP
- Unequal distribution possible

Alternative: Session cookies with any algorithm"
```

**Q3: "Difference between Least Connection and Least Response Time?"**

```
Answer:
"Both are dynamic, but different metrics:

Least Connection:
- Metric: Active connections
- Routes to server with fewest connections
- Problem: Connection ≠ Traffic
  - 1 connection with 500 req/s > 2 connections with 100 req/s

Least Response Time:
- Metric: Connections × TTFB
- Considers both load and performance
- More accurate
- Higher complexity

Example:
Server 1: 3 conn, 2ms TTFB → Score: 6
Server 2: 1 conn, 4ms TTFB → Score: 4
Route to Server 2 (lower score)

Least Response Time is better for performance-critical apps"
```

**Q4: "L4 vs L7 load balancer - which to choose?"**

```
Answer:
"Depends on requirements:

L4 (Network Load Balancer):
- Fast (transport layer)
- Simple routing (IP, port)
- Cannot read content
- Use when: Speed critical, simple routing

L7 (Application Load Balancer):
- Slower (application layer)
- Advanced routing (headers, URLs, content)
- Can cache responses
- Use when: Content-based routing, caching needed

Example:
L4: Route based on port 80 vs 443
L7: Route /api/* to API servers, /static/* to CDN

Modern systems often use both:
- L4 for initial distribution
- L7 for content-based routing"
```

**Q5: "How does Weighted Least Connection work?"**

```
Answer:
"Combines server capacity and current load.

Formula: Ratio = Active Connections / Weight

Example:
Server 1: Weight=10, Connections=2
Server 2: Weight=1, Connections=1

Ratios:
Server 1: 2/10 = 0.2
Server 2: 1/1 = 1.0

Route to minimum ratio (Server 1)

Why better than Least Connection:
- Considers capacity (weight)
- Protects weak servers
- Better distribution

Server 1 can handle 10x load, so ratio 0.2 is actually low
Server 2 at ratio 1.0 is at full capacity"
```

### Key Points to Remember

```
1. Load Balancer = Traffic distributor

2. L4 (fast) vs L7 (advanced)
   - L4: IP/port only
   - L7: Headers/content

3. Static vs Dynamic
   - Static: Predefined rules
   - Dynamic: Real-time metrics

4. Round Robin: Simple, equal distribution

5. Weighted: Considers capacity

6. IP Hash: Session persistence

7. Least Connection: Dynamic load

8. Least Response Time: Performance-aware

9. Trade-offs always exist:
   - Simple ↔ Accurate
   - Fast ↔ Advanced
```

### Do's ✅

**1. Explain Trade-offs**
```
"Round Robin is simple but ignores capacity.
Weighted Round Robin solves this but ignores processing time."
```

**2. Use Examples**
```
"Shopping cart needs IP Hash for session persistence.
API gateway needs L7 for content-based routing."
```

**3. Mention Formulas**
```
"Weighted Least Connection: Ratio = Connections / Weight
Least Response Time: Score = Connections × TTFB"
```

### Don'ts ❌

**1. Don't Confuse Static and Dynamic**
```
❌ "Round Robin is dynamic"
✓ "Round Robin is static (no real-time metrics)"
```

**2. Don't Ignore Limitations**
```
❌ "IP Hash is perfect for load balancing"
✓ "IP Hash good for sessions, but has proxy problem"
```

**3. Don't Forget L4 vs L7**
```
❌ "All load balancers can read HTTP headers"
✓ "Only L7 can read headers, L4 works at transport layer"
```

---

**End of Lecture**

Load balancers are critical for distributing traffic and ensuring high availability. Understanding the difference between L4 and L7, static and dynamic algorithms, and their trade-offs is essential for system design interviews. Choose algorithms based on server capacity, session requirements, and performance needs.

**Key Takeaway:** Static algorithms (Round Robin, Weighted, IP Hash) use predefined rules. Dynamic algorithms (Least Connection, Weighted Least Connection, Least Response Time) use real-time metrics. L4 is fast but simple, L7 is advanced but slower. Always consider trade-offs!
