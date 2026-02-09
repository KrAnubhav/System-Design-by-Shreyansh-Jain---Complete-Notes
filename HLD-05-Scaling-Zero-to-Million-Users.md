# HLD-05: Scaling from Zero to 1 Million Users

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Step 1: Single Server](#step-1-single-server)
3. [Step 2: Application and DB Server Separation](#step-2-application-and-db-server-separation)
4. [Step 3: Load Balancer + Multiple App Servers](#step-3-load-balancer--multiple-app-servers)
5. [Step 4: Database Replication (Master-Slave)](#step-4-database-replication-master-slave)
6. [Step 5: Use of Cache](#step-5-use-of-cache)
7. [Step 6: CDN (Content Delivery Network)](#step-6-cdn-content-delivery-network)
8. [Step 7: Multiple Data Centers](#step-7-multiple-data-centers)
9. [Step 8: Messaging Queue](#step-8-messaging-queue)
10. [Step 9: Database Scaling](#step-9-database-scaling)
11. [Summary](#summary)

---

## Introduction

This lecture covers **how to scale a system from 0 users to 1 million+ users** through 9 progressive steps.

**Key Concepts Covered:**
- CDN (Content Delivery Network)
- Horizontal and Vertical Scaling
- Caching
- Load Balancer
- Database Replication
- Messaging Queue
- Sharding

**Goal:** Set up the system to handle 1 million+ requests efficiently

---

## Step 1: Single Server

### Architecture

**Scenario:** Starting a new project, zero users

```
┌─────────────┐
│   Client    │
│ (Web/Mobile)│
└─────────────┘
       │
       │ Direct connection
       ▼
┌─────────────────────────┐
│    Single Server        │
├─────────────────────────┤
│                         │
│  • Application Server   │
│  • Database             │
│                         │
│  (Both in same server)  │
└─────────────────────────┘
```

### Characteristics

**What's included:**
- Application + DB both on same server
- DB hosted inside the server
- Client directly connects to server

**Example:**
- College projects
- Small personal projects
- MVP (Minimum Viable Product)

**Problems:**
- Cannot scale independently
- Single point of failure
- Limited resources

---

## Step 2: Application and DB Server Separation

### Architecture

```
┌─────────────┐
│   Client    │
│ (Web/Mobile)│
└─────────────┘
       │
       ▼
┌─────────────────────────┐
│     Mid-Tier Layer      │
├─────────────────────────┤
│  Application Server     │
│                         │
│  • Business Logic Only  │
└─────────────────────────┘
       │
       ▼
┌─────────────────────────┐
│      DB Tier            │
├─────────────────────────┤
│     DB Server           │
│                         │
│  • Data Storage Only    │
└─────────────────────────┘
```

### Why Separation?

**Benefits:**
✅ **Independent Growth**
- Application and DB can scale independently
- Remove dependency between them

✅ **Independent Servers**
- Application has its own server
- Database has its own server

✅ **Better Resource Management**
- Allocate resources based on specific needs
- App server: More CPU
- DB server: More memory/storage

---

## Step 3: Load Balancer + Multiple App Servers

### Architecture

```
┌─────────────┐
│   Client    │
│ (Web/Mobile)│
└─────────────┘
       │
       ▼
┌─────────────────────────┐
│    Load Balancer        │
│                         │
│  • Traffic Distribution │
│  • Security Layer       │
└─────────────────────────┘
       │
       ├──────────┬──────────┬──────────┐
       │          │          │          │
       ▼          ▼          ▼          ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│   App    │ │   App    │ │   App    │ │   App    │
│ Server 1 │ │ Server 2 │ │ Server 3 │ │ Server N │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
       │          │          │          │
       └──────────┴──────────┴──────────┘
                  │
                  ▼
           ┌─────────────┐
           │  DB Server  │
           └─────────────┘
```

### Load Balancer Benefits

**1. Traffic Distribution**
- Equally divides traffic among app servers
- Prevents overloading single server

**2. Security & Privacy**
- Load balancer and app servers communicate on **private IPs**
- Client cannot directly call app servers
- Extra layer of security

**3. High Availability**
- If one app server fails, load balancer routes to others
- System remains operational

### Why Multiple App Servers?

**Problem with Single Server:**

```
Single App Server:
GET /api endpoint → Can handle 1000 requests/minute
After 1000 → Starts dropping requests ❌
```

**Solution with Multiple Servers:**

```
Multiple App Servers:
Server 1 → 1000 requests/minute
Server 2 → 1000 requests/minute
Server 3 → 1000 requests/minute
Total: 3000 requests/minute ✅
```

### Communication

**Private IPs:**
```
Internet (Public) → Load Balancer (Public IP)
                         ↓
            App Servers (Private IPs)
            
Client cannot access app servers directly
Only through load balancer
```

---

## Step 4: Database Replication (Master-Slave)

### Architecture

```
┌──────────┐ ┌──────────┐ ┌──────────┐
│   App    │ │   App    │ │   App    │
│ Server 1 │ │ Server 2 │ │ Server 3 │
└──────────┘ └──────────┘ └──────────┘
     │            │            │
     └────────────┴────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
┌──────────────┐    ┌──────────────┐
│  Master DB   │───►│  Slave DB 1  │
│              │    │              │
│ Write Ops    │    │  Read Ops    │
└──────────────┘    └──────────────┘
        │                   
        │ Replication       
        ▼                   
┌──────────────┐    
│  Slave DB 2  │    
│              │    
│  Read Ops    │    
└──────────────┘    
```

### Master-Slave Concept

**Master DB:**
- Handles all **write operations** (Create, Update, Delete)
- Single master database

**Slave DB:**
- Handles all **read operations** (Select)
- Multiple slave databases possible
- Replicate data from master

**Replication:**
```
Master DB changes → Automatically replicated to Slave DBs
```

### Benefits

✅ **Fault Tolerance**

**Scenario 1: Slave DB Fails**
```
Master DB: ✓ (Still up)
Slave DB 1: ✗ (Failed)
Slave DB 2: ✓ (Still up)

Result: Read requests go to Slave DB 2
        System continues to work
```

**Scenario 2: Master DB Fails**
```
Master DB: ✗ (Failed)
Slave DB 1: ✓ (Promoted to Master)
Slave DB 2: ✓ (Continues as Slave)

Result: One slave gets promoted to master
        System continues to work
```

✅ **High Availability**
- System doesn't go down if one DB fails
- Automatic failover

✅ **Load Distribution**
- Write load on master
- Read load distributed across slaves

---

## Step 5: Use of Cache

### Architecture

```
┌──────────────┐
│ App Server   │
└──────────────┘
       │
       ▼
   Check Cache?
       │
   ┌───┴───┐
   │       │
   ▼       ▼
Cache    Cache
Hit      Miss
   │       │
   │       ▼
   │   ┌──────────┐
   │   │    DB    │
   │   └──────────┘
   │       │
   │       ▼
   │   Write to Cache
   │       │
   └───────┴──────► Return Response
```

### How Cache Works

**Cache Hit:**
```
1. App Server → Cache: "Do you have data for key X?"
2. Cache → App Server: "Yes, here's the data"
3. App Server → Client: Return data
   (No DB call needed!)
```

**Cache Miss:**
```
1. App Server → Cache: "Do you have data for key X?"
2. Cache → App Server: "No, I don't have it"
3. App Server → DB: "Get data for key X"
4. DB → App Server: Return data
5. App Server → Cache: "Store this data"
6. App Server → Client: Return data
```

### Why Cache?

**Problem:**
- DB operations are **expensive**
- Network call to DB takes time
- Every request hitting DB is slow

**Solution:**
- Cache stores frequently accessed data
- In-memory storage (very fast)
- Reduces DB load

### Performance Improvement

```
Without Cache:
Request → App Server → DB (10ms) → Response
Total: ~10-20ms

With Cache (Cache Hit):
Request → App Server → Cache (1ms) → Response
Total: ~1-2ms

Performance: 10x faster!
```

### Time to Live (TTL)

**What is TTL?**
- Time to Live = How long data stays in cache
- After TTL expires, data is purged

**Example:**
```
Set TTL = 24 hours

Day 1, 10:00 AM: Data stored in cache
Day 2, 10:00 AM: TTL expires, data purged
Day 2, 10:01 AM: Cache miss, fetch from DB again
```

**TTL Values:**
- 24 hours
- 48 hours
- 7 days
- Depends on application requirements

---

## Step 6: CDN (Content Delivery Network)

### What is CDN?

**CDN** = Content Delivery Network

**Important Distinction:**
```
CDN does caching ✓
But not all caching is CDN ✗

Example:
- Redis does caching → Redis is NOT CDN
- CDN does caching → CDN has additional functionality
```

### The Latency Problem

**Scenario: Data Center in India**

```
┌─────────────────────────────────────────────────┐
│         Users Accessing from Different          │
│              Locations                          │
├─────────────────────────────────────────────────┤
│                                                 │
│  India User  ──────► Data Center (India)       │
│  Latency: 1ms                                   │
│                                                 │
│  Saudi Arabia ─────► Data Center (India)       │
│  Latency: 2ms                                   │
│                                                 │
│  USA User   ───────► Data Center (India)       │
│  Latency: 3ms                                   │
│                                                 │
│  Japan User ───────► Data Center (India)       │
│  Latency: 4ms                                   │
│                                                 │
└─────────────────────────────────────────────────┘

Problem: Users far from data center have high latency
```

### CDN Solution

**Architecture:**

```
                ┌─────────────────┐
                │  Original Server│
                │    (India)      │
                └─────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  CDN Node    │ │  CDN Node    │ │  CDN Node    │
│   (USA)      │ │  (France)    │ │  (Japan)     │
└──────────────┘ └──────────────┘ └──────────────┘
        ▲               ▲               ▲
        │               │               │
   USA Users      France Users     Japan Users
```

### How CDN Works

**Step 1: Initial Setup**
```
Original Server stores static data in all CDN nodes:
- HTML pages
- CSS files
- JavaScript files
- Images
- Videos
```

**Step 2: User Request (Cache Hit)**
```
1. USA User → Nearest CDN (USA): "Get website X"
2. CDN (USA) → USA User: "Here's the cached data"
   (No need to go to India!)
```

**Step 3: User Request (Cache Miss)**
```
1. France User → CDN (France): "Get website X"
2. CDN (France): "I don't have it"
3. CDN (France) → Neighbor CDN: "Do you have it?"
4. If not found → CDN (France) → Original Server (India)
5. Original Server → CDN (France): Return data
6. CDN (France) caches data
7. CDN (France) → France User: Return data
```

### CDN Flow Diagram

```
User Request
     │
     ▼
Nearest CDN Node
     │
  ┌──┴──┐
  │     │
  ▼     ▼
 Has   Doesn't
Data   Have
  │     │
  │     ▼
  │  Check Neighbor CDN
  │     │
  │  ┌──┴──┐
  │  │     │
  │  ▼     ▼
  │ Has   Doesn't
  │Data   Have
  │  │     │
  │  │     ▼
  │  │  Original Server
  │  │     │
  │  └─────┴──────┐
  │               │
  └───────────────┴──► Return to User
```

### What CDN Caches

**Static Data:**
- HTML pages
- CSS files
- JavaScript files
- Images
- Videos
- Files that don't change frequently

**Storage Method:**
- Stored by URL
- URL acts as key

### CDN Benefits

✅ **Reduced Latency**
```
Without CDN:
Japan → India (4ms)

With CDN:
Japan → Japan CDN (0.5ms)
8x faster!
```

✅ **Improved Performance**
- Serve content from nearest location
- Faster response times

✅ **Increased Security**
- Protection against DDoS attacks
- Intelligent bot detection
- Request filtering before reaching original server

✅ **Reduced Load on Original Server**
- Most requests served by CDN
- Original server handles fewer requests

✅ **Cost Savings**
- Don't need as many DB servers
- Reduced bandwidth costs

### CDN Placement in Architecture

```
┌─────────────┐
│   Client    │
└─────────────┘
       │
       ▼
┌─────────────┐
│     CDN     │ ← First stop (before load balancer)
└─────────────┘
       │
       ▼ (If CDN doesn't have data)
┌─────────────┐
│Load Balancer│
└─────────────┘
       │
       ▼
┌─────────────┐
│ App Servers │
└─────────────┘
```

**Flow:**
1. Client → CDN (check for static content)
2. If CDN has data → Return to client
3. If CDN doesn't have data → Check neighbor CDN
4. If still not found → Request goes to original server

---

## Step 7: Multiple Data Centers

### Architecture

```
┌─────────────┐
│Load Balancer│
│             │
│ Geo-based   │
│  Routing    │
└─────────────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────────────────┐    ┌─────────────────┐
│  Data Center 1  │    │  Data Center 2  │
│    (India)      │    │     (USA)       │
├─────────────────┤    ├─────────────────┤
│                 │    │                 │
│ ┌─────┐ ┌─────┐│    │ ┌─────┐ ┌─────┐│
│ │App 1│ │App 2││    │ │App 1│ │App 2││
│ └─────┘ └─────┘│    │ └─────┘ └─────┘│
│       │        │    │       │        │
│       ▼        │    │       ▼        │
│ ┌───────────┐  │    │ ┌───────────┐  │
│ │Master DB  │  │    │ │Master DB  │  │
│ └───────────┘  │    │ └───────────┘  │
│       │        │    │       │        │
│       ▼        │    │       ▼        │
│ ┌───────────┐  │    │ ┌───────────┐  │
│ │ Slave DB  │  │    │ │ Slave DB  │  │
│ └───────────┘  │    │ └───────────┘  │
│                 │    │                 │
│ ┌───────────┐  │    │ ┌───────────┐  │
│ │   Cache   │  │    │ │   Cache   │  │
│ └───────────┘  │    │ └───────────┘  │
└─────────────────┘    └─────────────────┘
         │                      │
         └──────────────────────┘
              DB Replication
```

### How It Works

**Geo-based Routing:**
```
Load Balancer receives request
    │
    ├─► Check user location
    │
    ├─► User in Asia → Route to India Data Center
    │
    └─► User in Americas → Route to USA Data Center
```

### Each Data Center Contains

**Complete Infrastructure:**
- Multiple App Servers
- Master DB
- Slave DB(s)
- Cache
- All necessary components

### Benefits

✅ **Reduced Latency**
```
India User → India Data Center (Low latency)
USA User → USA Data Center (Low latency)
```

✅ **High Availability**

**Scenario: India Data Center Fails**
```
India Data Center: ✗ (Down)
USA Data Center: ✓ (Up)

Load Balancer → Routes all traffic to USA Data Center
System remains operational!
```

✅ **Load Distribution**
- Traffic distributed across data centers
- Reduced load on single data center

✅ **DB Replication Between Data Centers**
```
India Master DB ←──────────→ USA Master DB
                Replication
```

---

## Step 8: Messaging Queue

### Why Messaging Queue?

**Problem:**
- Some operations are heavy and slow
- Don't want to block request thread
- Need asynchronous processing

**Examples of Heavy Operations:**
- Sending notifications
- Sending emails
- Processing videos
- Generating reports

### What is Messaging Queue?

**Messaging Queue** = Brings asynchronous nature to codebase

**Popular Technologies:**
- RabbitMQ
- Kafka

### Basic Architecture

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   Producer   │────────►│  Messaging   │────────►│  Subscriber  │
│              │  Push   │    Queue     │  Pull   │  (Consumer)  │
└──────────────┘ Message └──────────────┘ Message └──────────────┘
```

### How It Works

**Async Flow:**

```
Step 1: Producer pushes message
┌──────────────┐
│ App Server   │ "Send email to user@example.com"
│ (Producer)   │
└──────────────┘
       │
       ▼ Push message
┌──────────────┐
│ Message Queue│ [Message 1, Message 2, ...]
└──────────────┘
       │
       ▼ Subscriber listens
┌──────────────┐
│ Email Service│ Process message and send email
│ (Subscriber) │
└──────────────┘
```

**Without Messaging Queue:**
```
User Request → App Server → Send Email (5 seconds)
                          → Wait...
                          → Response to user
Total: 5+ seconds (Bad!)
```

**With Messaging Queue:**
```
User Request → App Server → Push to Queue (10ms)
                          → Response to user
Total: 10ms (Good!)

Meanwhile:
Queue → Email Service → Send Email (5 seconds)
(Happens asynchronously)
```

### RabbitMQ Architecture

```
┌──────────────┐
│   Producer   │
└──────────────┘
       │
       │ Routing Key + Message
       ▼
┌─────────────────────────────────────┐
│           Exchange                  │
└─────────────────────────────────────┘
       │
       │ Binding
       ├────────────┬────────────┐
       │            │            │
       ▼            ▼            ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│  Queue 1 │  │  Queue 2 │  │  Queue 3 │
└──────────┘  └──────────┘  └──────────┘
       │            │            │
       ▼            ▼            ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│Subscriber│  │Subscriber│  │Subscriber│
│    1     │  │    2     │  │    3     │
└──────────┘  └──────────┘  └──────────┘
```

### RabbitMQ Components

**1. Producer**
- Sends messages to exchange
- Includes routing key

**2. Exchange**
- Receives messages from producer
- Routes to appropriate queue based on routing key

**3. Binding**
- Links exchange to queue
- Has binding key

**4. Queue**
- Stores messages
- Multiple queues possible

**5. Subscriber (Consumer)**
- Listens to queue
- Processes messages

### Exchange Types

#### 1. Direct Exchange

```
Producer sends: Routing Key = "order"

┌──────────┐
│ Exchange │
└──────────┘
       │
       │ Compare: Routing Key == Binding Key
       │
   ┌───┴───┐
   │       │
   ▼       ▼
Queue 1   Queue 2
Binding   Binding
Key:      Key:
"order"   "payment"
   ▲
   │
Matches! Send here
```

**Logic:**
- Routing Key **exactly matches** Binding Key
- Send to that queue only

#### 2. Fan-out Exchange

```
┌──────────┐
│ Exchange │
└──────────┘
       │
       ├────────┬────────┬────────┐
       │        │        │        │
       ▼        ▼        ▼        ▼
   Queue 1  Queue 2  Queue 3  Queue 4

All queues receive the same message!
```

**Logic:**
- Send message to **all queues**
- Subscribers decide whether to process or ignore

#### 3. Topic Exchange

```
Producer sends: Routing Key = "order.created"

┌──────────┐
│ Exchange │
└──────────┘
       │
       │ Wildcard matching
       │
   ┌───┴───┐
   │       │
   ▼       ▼
Queue 1   Queue 2
Binding   Binding
Key:      Key:
"order.*" "*.created"
   │       │
   └───┬───┘
       │
Both match! Send to both
```

**Logic:**
- Wildcard comparison (not exact match)
- Can send to **multiple queues**
- Pattern matching: `order.*`, `*.created`, etc.

### Retry Mechanism

**Dead Letter Queue:**

```
┌──────────────┐
│ Main Queue   │
└──────────────┘
       │
       ▼ Processing fails
┌──────────────┐
│ Dead Letter  │ Failed messages go here
│    Queue     │
└──────────────┘
       │
       ▼ Retry after fixing issue
┌──────────────┐
│ Subscriber   │ Process again
└──────────────┘
```

**Benefits:**
- If processing fails → Message goes to dead letter queue
- Fix the issue
- Retry processing
- Don't lose messages

### Messaging Queue in Architecture

```
┌──────────────┐         ┌──────────────┐
│ App Server 1 │────────►│  Messaging   │
└──────────────┘  Publish│    Queue     │
                         └──────────────┘
┌──────────────┐         ┌──────────────┐
│ App Server 2 │────────►│  Messaging   │
└──────────────┘  Publish│    Queue     │
                         └──────────────┘
                                │
                                │ Subscribe
                                ▼
                         ┌──────────────┐
                         │Other Services│
                         │ (Consumers)  │
                         └──────────────┘
```

### Benefits

✅ **Asynchronous Processing**
- Don't block request thread
- Fast response to user

✅ **Decoupling**
- Producer and consumer are independent
- Can scale separately

✅ **Reliability**
- Messages stored in queue
- Retry mechanism
- No data loss

✅ **Scalability**
- Add more subscribers to process faster
- Add more queues for different tasks

---

## Step 9: Database Scaling

### Two Types of Scaling

1. **Vertical Scaling**
2. **Horizontal Scaling**

---

### Vertical Scaling

**Definition:** Increase capacity of existing DB server

```
Before Vertical Scaling:
┌──────────────┐
│  Master DB   │
│  CPU: 4 core │
│  RAM: 16 GB  │
└──────────────┘

After Vertical Scaling:
┌──────────────┐
│  Master DB   │
│  CPU: 16 core│ ← Increased
│  RAM: 64 GB  │ ← Increased
└──────────────┘
```

**What to Increase:**
- CPU capability
- RAM capacity
- Storage

**Advantages:**
✅ Simple to implement
✅ No code changes needed

**Disadvantages:**
❌ **Has limits**
```
Cannot keep increasing forever:
- Maximum CPU cores available
- Maximum RAM available
- Physical hardware limitations
```

---

### Horizontal Scaling

**Definition:** Add more database nodes

```
Before Horizontal Scaling:
┌──────────────┐
│  Master DB   │
└──────────────┘
       │
       ▼
┌──────────────┐
│  Slave DB    │
└──────────────┘

After Horizontal Scaling:
┌──────────────┐
│  Master DB   │
└──────────────┘
       │
   ┌───┴───┬───────┬───────┐
   │       │       │       │
   ▼       ▼       ▼       ▼
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│Slave│ │Slave│ │Slave│ │Slave│
│ DB1 │ │ DB2 │ │ DB3 │ │ DB4 │
└─────┘ └─────┘ └─────┘ └─────┘

Add more nodes instead of increasing capacity
```

**Implementation:** Sharding

---

### Sharding

**Sharding** = Dividing data across multiple database nodes

**Two Types:**
1. Horizontal Sharding
2. Vertical Sharding

---

### Horizontal Sharding

**Definition:** Divide data **row-wise**

**Example: User Table with 1000 rows**

```
Original Table:
┌─────────────────────────────┐
│         Users Table         │
├─────────────────────────────┤
│ Row 1: User A (Name: Alice) │
│ Row 2: User B (Name: Bob)   │
│ Row 3: User C (Name: Carol) │
│ ...                         │
│ Row 1000: User Z            │
└─────────────────────────────┘

After Horizontal Sharding:
┌─────────────────┐    ┌─────────────────┐
│   Shard 1 (T1)  │    │   Shard 2 (T2)  │
├─────────────────┤    ├─────────────────┤
│ Row 1-500       │    │ Row 501-1000    │
│                 │    │                 │
│ Users A-M       │    │ Users N-Z       │
└─────────────────┘    └─────────────────┘
```

### Sharding Logic Examples

**Example 1: Range-based**
```
Shard 1: Rows 1-500
Shard 2: Rows 501-1000
```

**Example 2: Name-based**
```
Shard 1: Names A-M
Shard 2: Names N-Z
```

**Example 3: Alphabetical**
```
Shard 1: Names A-P
Shard 2: Names Q-Z
```

### How Horizontal Sharding Works

**Request Flow:**

```
User Request: "Get user with name starting with 'S'"
       │
       ▼
Application checks sharding logic:
"S is between Q-Z"
       │
       ▼
Route to Shard 2
       │
       ▼
┌─────────────────┐
│   Shard 2       │
│   Names Q-Z     │
└─────────────────┘
```

---

### Vertical Sharding

**Definition:** Divide data **column-wise**

**Example: User Table with 10 columns**

```
Original Table:
┌────────────────────────────────────────────┐
│              Users Table                   │
├────────────────────────────────────────────┤
│ C1: ID | C2: Name | C3: Email | C4: Phone │
│ C5: Address | C6: City | C7: State        │
│ C8: Country | C9: Age | C10: Gender       │
└────────────────────────────────────────────┘
        All rows, all columns

After Vertical Sharding:
┌─────────────────────┐    ┌─────────────────────┐
│   Shard 1 (T1)      │    │   Shard 2 (T2)      │
├─────────────────────┤    ├─────────────────────┤
│ C1: ID              │    │ C6: City            │
│ C2: Name            │    │ C7: State           │
│ C3: Email           │    │ C8: Country         │
│ C4: Phone           │    │ C9: Age             │
│ C5: Address         │    │ C10: Gender         │
├─────────────────────┤    ├─────────────────────┤
│ All rows            │    │ All rows            │
│ Columns 1-5         │    │ Columns 6-10        │
└─────────────────────┘    └─────────────────────┘
```

**Characteristics:**
- Divided **column-wise**
- Each shard contains **all rows**
- Different columns in different shards

---

### Horizontal vs Vertical Sharding

| Aspect | Horizontal Sharding | Vertical Sharding |
|--------|---------------------|-------------------|
| **Division** | Row-wise | Column-wise |
| **Rows** | Different rows in different shards | All rows in each shard |
| **Columns** | All columns in each shard | Different columns in different shards |
| **Use Case** | Large number of rows | Large number of columns |
| **Common** | More commonly used | Less commonly used |

---

### Sharding Drawbacks

#### 1. Uneven Distribution

**Problem:**

```
Sharding Logic: Names A-P → Shard 1, Names Q-Z → Shard 2

Reality:
┌─────────────────┐    ┌─────────────────┐
│   Shard 1       │    │   Shard 2       │
│   Names A-P     │    │   Names Q-Z     │
├─────────────────┤    ├─────────────────┤
│ 800 users       │    │ 200 users       │
│ (Very full!)    │    │ (Less full)     │
└─────────────────┘    └─────────────────┘

Problem: Shard 1 fills up quickly!
         Need to re-shard
```

**Solution: Re-sharding**

```
Original:
┌─────────────┐    ┌─────────────┐
│  Shard 1    │    │  Shard 2    │
│  A-P (800)  │    │  Q-Z (200)  │
└─────────────┘    └─────────────┘

After Re-sharding:
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│Shard 1.1│  │Shard 1.2│  │Shard 2.1│  │Shard 2.2│
│ A-F     │  │ G-P     │  │ Q-T     │  │ U-Z     │
│ (400)   │  │ (400)   │  │ (100)   │  │ (100)   │
└─────────┘  └─────────┘  └─────────┘  └─────────┘
```

**Tree Structure:**

```
         Original Shard
              │
        ┌─────┴─────┐
        │           │
    Shard 1     Shard 2
        │           │
    ┌───┴───┐   ┌───┴───┐
    │       │   │       │
Shard 1.1 1.2 2.1     2.2

Can keep sharding further as needed
```

#### 2. Cannot Join Across Shards

**Problem:**

```
┌─────────────┐    ┌─────────────┐
│  Shard 1    │    │  Shard 2    │
│  Rows 1-500 │    │  Rows 501-  │
│             │    │  1000       │
└─────────────┘    └─────────────┘

Need to join data from both shards:
❌ Cannot directly join across shards
```

**Solution: Denormalization**

```
Instead of joining:
- Store redundant data
- Avoid need for joins
- Trade storage for performance

Example:
Instead of:
  Users table + Orders table (need join)
  
Use:
  Orders table with user info embedded
  (No join needed)
```

### Solutions to Sharding Problems

**1. Consistent Hashing**
- Solves uneven distribution
- Minimizes re-sharding
- Will be covered in next lecture

**2. Denormalization**
- Avoid joins by storing redundant data
- Trade-off: More storage, less joins

---

## Summary

### The 9 Steps to Scale from 0 to 1 Million Users

```
Step 1: Single Server
        ↓
Step 2: Separate App and DB
        ↓
Step 3: Load Balancer + Multiple App Servers
        ↓
Step 4: Database Replication (Master-Slave)
        ↓
Step 5: Add Cache
        ↓
Step 6: Add CDN
        ↓
Step 7: Multiple Data Centers
        ↓
Step 8: Messaging Queue
        ↓
Step 9: Database Scaling (Sharding)
```

### Complete Architecture

```
                    ┌─────────────┐
                    │   Client    │
                    └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │     CDN     │
                    └─────────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │Load Balancer│
                    └─────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Data Center 1│  │ Data Center 2│  │ Data Center N│
├──────────────┤  ├──────────────┤  ├──────────────┤
│              │  │              │  │              │
│ App Servers  │  │ App Servers  │  │ App Servers  │
│      │       │  │      │       │  │      │       │
│      ▼       │  │      ▼       │  │      ▼       │
│   Cache      │  │   Cache      │  │   Cache      │
│      │       │  │      │       │  │      │       │
│      ▼       │  │      ▼       │  │      ▼       │
│  Master DB   │  │  Master DB   │  │  Master DB   │
│      │       │  │      │       │  │      │       │
│      ▼       │  │      ▼       │  │      ▼       │
│  Slave DBs   │  │  Slave DBs   │  │  Slave DBs   │
│      │       │  │      │       │  │      │       │
│      ▼       │  │      ▼       │  │      ▼       │
│ Msg Queue    │  │ Msg Queue    │  │ Msg Queue    │
└──────────────┘  └──────────────┘  └──────────────┘
```

### Key Concepts Importance

**Very Important ⭐⭐⭐:**
- CDN
- Database Replication (Master-Slave)
- Messaging Queue
- Database Scaling (Sharding)
- Cache

**Important ⭐⭐:**
- Load Balancer
- Multiple App Servers
- Multiple Data Centers

### Interview Preparation

**Most Commonly Asked:**
1. How to scale from 0 to 1 million users?
2. Explain CDN and its benefits
3. Master-Slave replication
4. Horizontal vs Vertical scaling
5. Sharding strategies
6. Messaging queue use cases

**Be Ready to Explain:**
- Each step and why it's needed
- Trade-offs at each step
- When to apply each technique
- Real-world examples

### Key Takeaways

1. **Start Simple:** Single server → Gradually add complexity

2. **Separation of Concerns:** Separate app and DB early

3. **High Availability:** Load balancer, multiple servers, replication

4. **Performance:** Cache, CDN reduce latency

5. **Scalability:** Horizontal scaling, sharding for growth

6. **Async Processing:** Messaging queue for heavy operations

7. **Global Reach:** Multiple data centers, CDN for worldwide users

8. **Fault Tolerance:** Replication, multiple nodes prevent downtime

---

**End of Lecture**

This systematic approach to scaling ensures your system can handle millions of users while maintaining performance, availability, and reliability. Each step builds upon the previous one, creating a robust, scalable architecture.
