# HLD-03: Monolithic vs Microservices & Decomposition Patterns

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Types of Architectures](#types-of-architectures)
3. [Monolithic Architecture](#monolithic-architecture)
   - [What is Monolithic?](#what-is-monolithic)
   - [Disadvantages of Monolithic](#disadvantages-of-monolithic)
4. [Microservices Architecture](#microservices-architecture)
   - [What are Microservices?](#what-are-microservices)
   - [Advantages of Microservices](#advantages-of-microservices)
   - [Disadvantages of Microservices](#disadvantages-of-microservices)
5. [Microservices Phases and Patterns](#microservices-phases-and-patterns)
6. [Decomposition Patterns](#decomposition-patterns)
   - [Pattern 1: Decompose by Business Capability](#pattern-1-decompose-by-business-capability)
   - [Pattern 2: Decompose by Subdomain (DDD)](#pattern-2-decompose-by-subdomain-ddd)
7. [Summary](#summary)

---

## Introduction

This lecture covers:
- Monolithic vs Microservices architectures
- Important microservices patterns
- How to properly decompose applications into microservices

**Common Interview Question:**
"What are the advantages and disadvantages of microservices?"

---

## Types of Architectures

There are **two types** of architectures:

1. **Monolithic**
2. **Microservices**

```
┌─────────────────────────────────────────────────┐
│              Architecture Types                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─────────────────┐   ┌─────────────────┐    │
│  │   Monolithic    │   │  Microservices  │    │
│  │    (Legacy)     │   │    (Modern)     │    │
│  └─────────────────┘   └─────────────────┘    │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## Monolithic Architecture

### What is Monolithic?

**Monolithic** = Single application containing all functionalities

**Also called:** Legacy applications

**Common scenario in companies:**
> "This is a legacy application that needs to be migrated to microservices."

### Example: Online Store

```
┌─────────────────────────────────────────────┐
│       Monolithic Application (Single)       │
├─────────────────────────────────────────────┤
│                                             │
│  • Order Generation                         │
│  • Product Inventory Management             │
│  • Login Management                         │
│  • Billing                                  │
│  • Payment                                  │
│  • All Backend Functionalities              │
│                                             │
└─────────────────────────────────────────────┘
         ▲
         │
    Everything in ONE application
```

**Characteristics:**
- All functionalities in **one application**
- Single codebase
- Single deployment unit
- Tightly coupled components

---

## Disadvantages of Monolithic

### 1. Overloaded IDE

**Problem:**
- Application becomes very large (sometimes in **gigabytes**)
- IDE takes a long time to load
- Sometimes doesn't load at all
- Very difficult for developers to work

```
┌─────────────────────────────────────┐
│  Monolithic Application: 10 GB      │
│                                     │
│  Loading in IDE...                  │
│  ████████░░░░░░░░░░░░░░ 35%         │
│  (Takes forever!)                   │
└─────────────────────────────────────┘
```

### 2. Scaling is Very Hard

**What is scaling?**
- Scaling = Ability to grow
- Fast CI/CD operations
- Easy management of components

**Why is scaling hard in monolithic?**

#### Problem 1: Tightly Coupled Code

```
┌─────────────────────────────────────────┐
│      Monolithic Application             │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐│
│  │ Order   │  │ Product │  │ Payment ││
│  │  Code   │  │  Code   │  │  Code   ││
│  └─────────┘  └─────────┘  └─────────┘│
│       │            │            │      │
│       └────────────┴────────────┘      │
│         All using same code            │
│         (Tightly Coupled)              │
└─────────────────────────────────────────┘
```

**Impact:**
- Change **one line** → Impacts multiple domains
- Order code uses same function
- Product code uses same function
- Payment code uses same function
- Must test **everything** when changing anything

#### Problem 2: Deploy Entire Application

**Workflow for one-line bug fix:**

```
1. Fix one line of code
   ↓
2. Run regression on ENTIRE application
   ↓
3. Deploy ENTIRE application (10 GB)
   ↓
4. Monitor ENTIRE application
   ↓
Takes a LOT of time!
```

#### Problem 3: Cannot Scale Individual Components

**Scenario:**
- Order functionality has **high traffic**
- Other functionalities have normal traffic

**Monolithic approach:**

```
┌──────────────────────┐         ┌──────────────────────┐
│   Server 1 (10 GB)   │         │   Server 2 (10 GB)   │
├──────────────────────┤         ├──────────────────────┤
│ • Order (High Load)  │         │ • Order (High Load)  │
│ • Product (Normal)   │  Add    │ • Product (Normal)   │
│ • Payment (Normal)   │ ──────► │ • Payment (Normal)   │
│ • Billing (Normal)   │ Server  │ • Billing (Normal)   │
│ • Login (Normal)     │         │ • Login (Normal)     │
└──────────────────────┘         └──────────────────────┘

Problem: Must scale ENTIRE 10 GB application
         just to handle Order traffic!
         Very costly and inefficient!
```

**Issues:**
- Cannot scale **only** the Order functionality
- Must add entire 10 GB application on new server
- Very **costly**
- Very **inefficient**

### Summary of Monolithic Disadvantages

❌ **Overloaded IDE** - Difficult to load and work with
❌ **Tightly Coupled** - One change impacts multiple areas
❌ **Hard to Debug** - Large codebase
❌ **Slow CI/CD** - Regression, deployment takes long time
❌ **Cannot Scale Individual Components** - Must scale entire application
❌ **Costly** - Scaling requires duplicating entire application

---

## Microservices Architecture

### What are Microservices?

**Microservices** = Dividing a large application into **small, independent services**

### Example: Online Store

```
┌─────────────────────────────────────────────────────────┐
│         Microservices Architecture                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ Product  │  │  Order   │  │ Billing  │             │
│  │  Mgmt    │  │   Mgmt   │  │          │             │
│  └──────────┘  └──────────┘  └──────────┘             │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ Payment  │  │ Account  │  │  Login   │             │
│  │          │  │   Mgmt   │  │          │             │
│  └──────────┘  └──────────┘  └──────────┘             │
│                                                         │
│        Each box = Independent Service                   │
└─────────────────────────────────────────────────────────┘
```

**Characteristics:**
- Large application divided into **small services**
- Each service is **independent**
- Each service handles specific functionality

---

## Advantages of Microservices

### 1. Easy to Manage and Debug

**Bug fix workflow:**

```
Find bug in Order Service
   ↓
Debug ONLY Order Service (not entire app)
   ↓
Fix ONLY Order Service
   ↓
Deploy ONLY Order Service
   ↓
Much faster and easier!
```

**Before (Monolithic):** Work on entire application
**After (Microservices):** Work on just one service

### 2. Easy to Scale

**Scenario:** High traffic on Order Service

```
┌──────────┐                    ┌──────────┐  ┌──────────┐
│  Order   │                    │  Order   │  │  Order   │
│  Service │  Scale only  ────► │  Service │  │  Service │
│          │  what's needed     │          │  │          │
└──────────┘                    └──────────┘  └──────────┘

┌──────────┐                    ┌──────────┐
│ Product  │                    │ Product  │
│ Service  │  No need to  ────► │ Service  │
│          │  scale this        │          │
└──────────┘                    └──────────┘

Result: Scale ONLY Order Service
        Cost-effective and efficient!
```

**Benefits:**
- Scale **only** the component that needs it
- **Pocket-friendly** - No need to scale everything
- **Efficient** - Add resources where needed

### 3. Loosely Coupled

**Characteristics:**
- Services are independent
- Change in one service doesn't impact others
- Can deploy independently

### Summary of Microservices Advantages

✅ **Easy to Manage** - Work on individual services
✅ **Easy to Debug** - Debug only the affected service
✅ **Easy to Scale** - Scale only what's needed
✅ **Cost-effective** - Pay only for what you scale
✅ **Loosely Coupled** - Independent services
✅ **Fast Deployment** - Deploy individual services

**Key Point:**
> All disadvantages of Monolithic are advantages of Microservices!

---

## Disadvantages of Microservices

### 1. Proper Decomposition Required

**Problem:**
- Must properly break monolithic into microservices
- If not done properly → Can be very costly

**Bad Decomposition Example:**

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Service  │────►│ Service  │────►│ Service  │────►│ Service  │
│    1     │◄────│    2     │◄────│    3     │◄────│    4     │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                │                │                │
     └────────────────┴────────────────┴────────────────┘
              Tightly Coupled Dependencies!
```

**Issues with bad decomposition:**
- Services become **tightly coupled** (not loosely coupled)
- One request requires communication with multiple services
- **Latency increases** due to inter-service communication
- Changing one service requires changing others

**Impact on Latency:**

```
Monolithic:
One API call → 5 milliseconds

Bad Microservices:
Service 1 → Service 2 → Service 3 → Service 4
(Network calls add latency)
Total: 10+ milliseconds
```

**Solution:**
- Services should be **loosely coupled**
- Minimal dependencies between services
- Each service should be independently scalable and changeable

### 2. Monitoring and Debugging Complexity

**Problem:**
- When one service deploys new code, it can break its clients

**Example Scenario:**

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│ Service  │────────►│ Service  │────────►│ Service  │
│    1     │  calls  │    2     │  calls  │    3     │
└──────────┘         └──────────┘         └──────────┘
                           │
                           │ (New deployment)
                           ▼
                     Changed response
                     format/behavior
                           │
                           ▼
                     ┌──────────┐
                     │ Service  │
                     │    2     │ ← Breaks!
                     └──────────┘
                           │
                           ▼
                     ┌──────────┐
                     │ Service  │
                     │    1     │ ← Also breaks!
                     └──────────┘
```

**Debugging becomes confusing:**

```
1. S1 engineer: "My service is failing because of S2"
   ↓
2. S2 engineer: "My service is failing because of S3"
   ↓
3. S3 engineer: "My service is running perfectly!"
   ↓
Problem: S3 changed response format, breaking its clients
```

**Challenges:**
- Difficult to monitor which components to check before deployment
- Service might run fine, but its clients break
- Hard to identify the culprit
- Requires good monitoring and observability

### 3. Transaction Management

**Problem:**
- In monolithic: One database, one transaction
- In microservices: Multiple databases, complex transaction management

**Monolithic Transaction:**

```
┌─────────────────────────────────┐
│   Monolithic Application        │
├─────────────────────────────────┤
│                                 │
│  Start Transaction              │
│    ↓                            │
│  DB Operation 1                 │
│  DB Operation 2                 │
│  DB Operation 3                 │
│    ↓                            │
│  All Success → Commit           │
│  Any Failure → Rollback         │
│                                 │
└─────────────────────────────────┘
         │
         ▼
   ┌──────────┐
   │    DB    │
   └──────────┘
```

**Microservices Transaction:**

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

Problem: Cannot create a common transaction across multiple DBs!
```

**Scenario:**

```
Request needs Service 1 and Service 2:

Service 1:
  Start Transaction
  DB1 Operation → Success ✓
  Commit

Service 2:
  Start Transaction
  DB2 Operation → Failed ✗
  Rollback

Problem: DB1 committed, but DB2 failed!
         How to rollback DB1?
```

**Challenges:**
- Cannot create common transaction across services
- If Service 1 succeeds and Service 2 fails → Inconsistent state
- Need distributed transaction patterns (Saga pattern, etc.)

### Summary of Microservices Disadvantages

❌ **Proper Decomposition Required** - Must design carefully to avoid tight coupling
❌ **Latency Issues** - Inter-service communication adds latency
❌ **Monitoring Complexity** - Hard to track which service caused the issue
❌ **Debugging Difficulty** - Error could be in any service in the chain
❌ **Transaction Management** - Complex to maintain consistency across services

**Important:**
> Microservices have disadvantages, but we have solutions (patterns) for them!

---

## Microservices Phases and Patterns

**Microservices = Micro + Service**

**Question:** How small should "micro" be?

**Answer:** There are **patterns** to guide us!

### The Five Phases

```
┌─────────────────────────────────────────────────────────┐
│         Microservices Phases and Patterns               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Decomposition                                       │
│     └─ How to break monolithic into small services?    │
│                                                         │
│  2. Database                                            │
│     └─ Separate DB per service or shared DB?           │
│                                                         │
│  3. Communication                                       │
│     └─ How will services communicate?                  │
│                                                         │
│  4. Integration                                         │
│     └─ How to integrate with UI and other apps?        │
│                                                         │
│  5. Observability                                       │
│     └─ How to monitor and debug?                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Phase Details

#### 1. Decomposition
**Question:** How to divide a large service into smaller ones?

**Patterns:**
- Decompose by Business Capability
- Decompose by Subdomain (DDD)

#### 2. Database
**Question:** Should each service have its own database?

**Patterns:**
- Database per Service
- Shared Database

#### 3. Communication
**Question:** How will services communicate with each other?

**Patterns:**
- API Communication
- Event-based Communication
- Message Queues

#### 4. Integration
**Question:** How to integrate microservices with UI and external apps?

**Patterns:**
- API Gateway
- Backend for Frontend (BFF)

#### 5. Observability
**Question:** How to monitor and debug distributed systems?

**Patterns:**
- Logging
- Monitoring
- Distributed Tracing

### How Microservices are Formed

```
Decomposition Pattern (Choose one)
        +
Database Pattern (Choose one)
        +
Communication Pattern (Choose one)
        +
Integration Pattern (Choose one)
        +
Observability Pattern (Choose one)
        ║
        ▼
Your Microservices Architecture
```

**Key Point:**
> At each phase, you choose a pattern. Mixing all patterns together forms your microservices architecture.

**Complexity:**
- Microservices involve many decisions
- Each phase has multiple patterns
- Must choose appropriate patterns for your use case

---

## Decomposition Patterns

**Focus of this lecture:** Decomposition Phase

**Question:** How to divide monolithic into microservices? How small should each service be?

### Two Decomposition Patterns

1. **Decompose by Business Capability**
2. **Decompose by Subdomain (Domain-Driven Design - DDD)**

---

## Pattern 1: Decompose by Business Capability

### Definition

**Decompose by Business Capability** = Create services based on business functions

### Example: Online Order Application

**Identify Business Capabilities:**

```
┌─────────────────────────────────────────────────────────┐
│         Business Capabilities (Functions)               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │      Order       │  │     Product      │           │
│  │   Management     │  │   Management     │           │
│  │                  │  │                  │           │
│  │ Job: Manage      │  │ Job: Manage      │           │
│  │ incoming orders  │  │ inventory        │           │
│  └──────────────────┘  └──────────────────┘           │
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │     Account      │  │      Login       │           │
│  │   Management     │  │                  │           │
│  │                  │  │                  │           │
│  │ Job: Manage      │  │ Job: Handle      │           │
│  │ user accounts    │  │ authentication   │           │
│  └──────────────────┘  └──────────────────┘           │
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │     Billing      │  │     Payment      │           │
│  │                  │  │                  │           │
│  │ Job: Generate    │  │ Job: Process     │           │
│  │ bills            │  │ payments         │           │
│  └──────────────────┘  └──────────────────┘           │
│                                                         │
│      Each box = One Microservice                       │
└─────────────────────────────────────────────────────────┘
```

### How to Apply This Pattern

**Step 1:** Identify all business functions in your application

**Step 2:** Create one service per business function

| Business Function | Microservice | Job |
|-------------------|--------------|-----|
| Order handling | Order Management Service | Manage incoming orders |
| Product handling | Product Management Service | Manage inventory |
| User accounts | Account Management Service | Manage user accounts |
| Authentication | Login Service | Handle authentication |
| Billing | Billing Service | Generate bills |
| Payments | Payment Service | Process payments |

### Key Principle

**One business capability = One microservice**

```
Business Capability: Order Management
         ↓
Create Service: Order Management Service
         ↓
Responsibility: ONLY manage orders
```

### Challenge

**Requirement:** Good knowledge of business functionalities

**Problem:**
- If you don't have clarity on business functions → Difficult to decide services
- Must understand the business domain well

**Solution:**
- Work with business stakeholders
- Understand all business capabilities
- Document business functions clearly

### What is "Micro"?

**Important Point:** There is **no fixed definition** of "micro"

```
┌─────────────────────────────────────────────────┐
│  What is "Micro"? (Context-dependent)           │
├─────────────────────────────────────────────────┤
│                                                 │
│  Small Project:                                 │
│  Order Management = 100 lines → Micro          │
│                                                 │
│  Large Project:                                 │
│  Order Management = 10,000 lines → Still Micro │
│                                                 │
│  Enterprise Project:                            │
│  Order Management = 100,000 lines → Still Micro│
│                                                 │
└─────────────────────────────────────────────────┘
```

**Key Point:**
- "Micro" is **relative** to your project size
- For one project, Order Management might be small
- For another project, Order Management itself might be very large
- But in the context of that project, it's still considered a "microservice"

**Example:**
- Order Management itself is a very big application
- But in the context of the overall system, it's one microservice
- It's "micro" compared to the entire monolithic application

---

## Pattern 2: Decompose by Subdomain (DDD)

### Definition

**Decompose by Subdomain** = Divide domains into smaller subdomains, each becoming a microservice

**Also called:** Domain-Driven Design (DDD)

### Difference from Business Capability

**Business Capability Pattern:**
```
Order Management (Business Function)
        ↓
One Microservice
```

**Subdomain Pattern:**
```
Order Management (Domain)
        ↓
Multiple Microservices (Subdomains)
```

### Example: Payment Domain

**Domain:** Payment

**Subdomains:**

```
┌─────────────────────────────────────────────────┐
│           Payment Domain                        │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────────┐  ┌──────────────────┐   │
│  │     Forward      │  │     Reverse      │   │
│  │     Payment      │  │     Payment      │   │
│  │                  │  │                  │   │
│  │ User pays        │  │ Refund to user   │   │
│  │ another user     │  │                  │   │
│  └──────────────────┘  └──────────────────┘   │
│         ▲                      ▲               │
│         │                      │               │
│    Microservice 1         Microservice 2       │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Explanation:**
- **Payment** is one domain
- Within Payment domain:
  - **Forward Payment** = User making payment (subdomain/microservice)
  - **Reverse Payment** = Refund capability (subdomain/microservice)

### Example: Order Management Domain

**Domain:** Order Management

**Subdomains:**

```
┌─────────────────────────────────────────────────┐
│        Order Management Domain                  │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────────┐  ┌──────────────────┐   │
│  │     Order        │  │     Order        │   │
│  │     Placing      │  │    Tracking      │   │
│  │                  │  │                  │   │
│  │ Place new orders │  │ Track order      │   │
│  │                  │  │ status           │   │
│  └──────────────────┘  └──────────────────┘   │
│         ▲                      ▲               │
│         │                      │               │
│    Microservice 1         Microservice 2       │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Explanation:**
- **Order Management** is one domain
- Within Order Management domain:
  - **Order Placing** = Placing new orders (subdomain/microservice)
  - **Order Tracking** = Tracking order status (subdomain/microservice)

### How to Apply This Pattern

**Step 1:** Identify domains

**Step 2:** Break each domain into subdomains

**Step 3:** Each subdomain becomes a microservice

```
Domain: Payment
   ├─ Subdomain: Forward Payment → Microservice
   └─ Subdomain: Reverse Payment → Microservice

Domain: Order Management
   ├─ Subdomain: Order Placing → Microservice
   └─ Subdomain: Order Tracking → Microservice
```

### Key Principle

**One domain can have multiple microservices**

```
Domain (Large)
    ↓
Subdomains (Smaller)
    ↓
Microservices (Independent)
```

---

## Comparison: Business Capability vs Subdomain

### Visual Comparison

**Business Capability Pattern:**

```
┌──────────────────┐
│      Order       │
│   Management     │ ← One Business Function
│                  │ ← One Microservice
│ All order-related│
│  functionality   │
└──────────────────┘
```

**Subdomain Pattern:**

```
┌─────────────────────────────────────┐
│      Order Management Domain        │
├─────────────────────────────────────┤
│  ┌──────────┐    ┌──────────┐      │
│  │  Order   │    │  Order   │      │
│  │ Placing  │    │ Tracking │      │
│  └──────────┘    └──────────┘      │
│       ▲               ▲             │
│       │               │             │
│  Microservice    Microservice       │
└─────────────────────────────────────┘
```

### Key Differences

| Aspect | Business Capability | Subdomain (DDD) |
|--------|---------------------|-----------------|
| **Approach** | One function = One service | One domain = Multiple services |
| **Granularity** | Coarser | Finer |
| **Example** | Order Management = 1 service | Order Management = Multiple services |
| **Focus** | Business function | Domain and subdomains |
| **Result** | Fewer, larger services | More, smaller services |

### When to Use Which?

**Use Business Capability when:**
- Business functions are well-defined
- Each function is independent
- Simpler architecture needed

**Use Subdomain (DDD) when:**
- Domains are complex and large
- Need finer-grained services
- Want more flexibility in scaling specific capabilities

---

## Summary

### Monolithic vs Microservices

| Aspect | Monolithic | Microservices |
|--------|------------|---------------|
| **Structure** | Single application | Multiple independent services |
| **Coupling** | Tightly coupled | Loosely coupled |
| **Scaling** | Scale entire app | Scale individual services |
| **Deployment** | Deploy entire app | Deploy individual services |
| **Cost** | High (scale everything) | Low (scale what's needed) |
| **Complexity** | Simple architecture | Complex architecture |
| **IDE** | Overloaded | Manageable |

### Microservices Advantages

✅ Easy to manage and debug
✅ Easy to scale specific components
✅ Cost-effective scaling
✅ Loosely coupled
✅ Fast deployment
✅ Independent services

### Microservices Disadvantages

❌ Requires proper decomposition
❌ Latency due to inter-service communication
❌ Complex monitoring and debugging
❌ Complex transaction management

### Microservices Phases

1. **Decomposition** - How to break into services
2. **Database** - DB per service or shared
3. **Communication** - How services talk
4. **Integration** - How to integrate with UI
5. **Observability** - How to monitor

### Decomposition Patterns

**Pattern 1: Business Capability**
- One business function = One microservice
- Example: Order Management, Payment, Billing

**Pattern 2: Subdomain (DDD)**
- One domain = Multiple microservices
- Example: Payment domain → Forward Payment + Reverse Payment

### Key Takeaways

1. **Monolithic** is legacy, **Microservices** is modern
2. All disadvantages of monolithic are advantages of microservices
3. Microservices have disadvantages, but patterns solve them
4. **"Micro"** is relative to project size
5. Proper decomposition is critical for success
6. Choose decomposition pattern based on your use case

### Interview Tips

**Common Question:** "What are advantages and disadvantages of microservices?"

**Answer Structure:**
1. List advantages (opposite of monolithic disadvantages)
2. List disadvantages (proper decomposition, latency, monitoring, transactions)
3. Mention that patterns exist to solve disadvantages
4. Show understanding of when to use microservices vs monolithic

---

**End of Lecture**

This lecture covered the fundamentals of Monolithic vs Microservices architecture and decomposition patterns. Understanding these concepts is crucial for designing scalable distributed systems.
