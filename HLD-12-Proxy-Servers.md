# HLD-12: Proxy Servers (Forward & Reverse)

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What is Proxy Server](#what-is-proxy-server)
3. [Forward Proxy](#forward-proxy)
4. [Reverse Proxy](#reverse-proxy)
5. [Key Comparisons](#key-comparisons)
6. [Summary](#summary)
7. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Proxy Servers

**Coverage:**
- ✅ What is proxy server
- ✅ Forward proxy (client-side)
- ✅ Reverse proxy (server-side)
- ✅ Proxy vs VPN
- ✅ Proxy vs Load Balancer
- ✅ Proxy vs Firewall

**Why These Comparisons Matter:**
Understanding differences prevents confusion in interviews!

---

## What is Proxy Server

### Real-World Analogy

```
Child wants chocolate from shop:

┌───────┐                    ┌──────┐
│ Child │───────────────────►│ Shop │
└───────┘   Direct request   └──────┘
              ✗ Not allowed

WITH Proxy (Mom):

┌───────┐      ┌─────┐      ┌──────┐
│ Child │─────►│ Mom │─────►│ Shop │
└───────┘      └─────┘      └──────┘
               Proxy

Child → Mom: "I want chocolate"
Mom → Shop: Gets chocolate
Mom → Child: Gives chocolate

Mom = Proxy (acts on behalf of child)
```

**Key Concept:**
```
Proxy = Acting on someone's behalf
```

---

### Technical Definition

```
┌────────┐      ┌───────┐      ┌────────┐
│ Client │─────►│ Proxy │─────►│ Server │
└────────┘      └───────┘      └────────┘

Proxy Server:
- Sits between client and server
- Acts as intermediary
- All requests pass through proxy
- No direct client-server communication
```

**Multiple Clients:**

```
┌──────────┐
│ Client 1 │──┐
└──────────┘  │
              │
┌──────────┐  │    ┌───────┐      ┌────────┐
│ Client 2 │──┼───►│ Proxy │─────►│ Server │
└──────────┘  │    └───────┘      └────────┘
              │
┌──────────┐  │
│ Client 3 │──┘
└──────────┘

Proxy handles multiple clients
```

---

## Forward Proxy

### Definition

**Forward Proxy** = Proxy on client side (also called "simple proxy")

```
Internal Network          Internet
┌─────────────────┐
│  ┌──────────┐   │
│  │ Client 1 │   │
│  │172.1.0.1 │   │
│  └──────────┘   │
│       │         │
│  ┌──────────┐   │    ┌─────────┐      ┌────────┐
│  │ Client 2 │   │    │ Forward │      │ Server │
│  │172.2.0.1 │───┼───►│  Proxy  │─────►│        │
│  └──────────┘   │    │192.3.0.1│      └────────┘
│       │         │    └─────────┘
│  ┌──────────┐   │
│  │ Client 3 │   │
│  │172.3.0.1 │   │
│  └──────────┘   │
│                 │
│ Intranet/       │
│ Private Network │
└─────────────────┘
```

---

### How It Works

**IP Address Masking:**

```
Client 1 makes request:
Real IP: 172.1.0.1
Request: GET google.com

Forward Proxy:
- Receives request from 172.1.0.1
- Sends request with own IP: 192.3.0.1
- Server sees: 192.3.0.1 requesting google.com

Server Response:
- Server responds to 192.3.0.1
- Proxy receives response
- Proxy forwards to 172.1.0.1

Server never knows about 172.1.0.1!
```

**Key Point:**
```
Forward Proxy hides CLIENT from server
Server only knows about proxy
```

---

### Advantages

#### 1. Anonymity

```
Without Proxy:
Server knows: Client IP, Location, Identity

With Forward Proxy:
Server knows: Only proxy IP
Client hidden ✓
```

**Benefit:** Privacy protection

---

#### 2. Request Grouping

```
Scenario:
Client 1 → google.com
Client 2 → google.com
Client 3 → google.com

Without Proxy:
3 separate requests to google.com

With Forward Proxy:
Proxy groups requests
Sends 1 combined request
More efficient ✓
```

**Benefit:** Reduced network traffic

---

#### 3. Access Restricted Content

```
Scenario: Website blocked in India

Without Proxy:
India User → Blocked Website ✗
Request blocked (geo-restriction)

With Forward Proxy (in UK):
India User → UK Proxy → Blocked Website ✓
Server thinks request from UK
Access granted ✓
```

**Benefit:** Bypass geo-restrictions

---

#### 4. Security & Control

```
Company Network:
100 employees → Forward Proxy → Internet

Proxy Rules:
✗ Block facebook.com
✗ Block youtube.com
✓ Allow work-related sites

Employee tries facebook.com:
Request → Proxy → Blocked by rule ✗
```

**Benefit:** Centralized access control

---

#### 5. Caching

```
First Request:
Client 1 → Proxy: "Get website-a.com"
Proxy checks cache: Not found
Proxy → Server: Fetch website-a.com
Server → Proxy: Returns content
Proxy: Stores in cache
Proxy → Client 1: Returns content

Second Request:
Client 2 → Proxy: "Get website-a.com"
Proxy checks cache: Found! ✓
Proxy → Client 2: Returns from cache
(No server request needed)
```

**Benefit:** Faster response, reduced load

---

### Disadvantage

**Application-Level Setup:**

```
Network Layers:
├─ Application Layer ← Proxy works here
├─ Transport Layer
├─ Network Layer
├─ Data Link Layer
└─ Physical Layer

Problem:
App 1 → Need Proxy 1
App 2 → Need Proxy 2
App 3 → Need Proxy 3
...
App 10 → Need Proxy 10

Must configure proxy for EACH application
```

**Drawback:** Not packet-level, application-specific

---

### Summary: Forward Proxy

```
┌────────────────────────────────────────────┐
│         Forward Proxy                      │
├────────────────────────────────────────────┤
│ Position: Client-side                      │
│ Protects: Clients                          │
│ Hides: Client IP from server               │
│                                            │
│ Advantages:                                │
│ ✓ Anonymity                                │
│ ✓ Request grouping                         │
│ ✓ Access restricted content                │
│ ✓ Security & control                       │
│ ✓ Caching                                  │
│                                            │
│ Disadvantage:                              │
│ ✗ Application-level (not packet-level)    │
│                                            │
│ Use Cases:                                 │
│ - Corporate networks                       │
│ - Bypass geo-restrictions                 │
│ - Content filtering                        │
└────────────────────────────────────────────┘
```

---

## Reverse Proxy

### Definition

**Reverse Proxy** = Proxy on server side

```
Internet                  Internal Network
                          ┌─────────────────┐
┌──────────┐              │  ┌──────────┐   │
│ Client 1 │──┐           │  │ Server 1 │   │
└──────────┘  │           │  └──────────┘   │
              │           │                 │
┌──────────┐  │  ┌────────┐  ┌──────────┐   │
│ Client 2 │──┼─►│Reverse │─►│ Server 2 │   │
└──────────┘  │  │ Proxy  │  └──────────┘   │
              │  └────────┐                 │
┌──────────┐  │           │  ┌──────────┐   │
│ Client 3 │──┘           │  │ Server N │   │
└──────────┘              │  └──────────┘   │
                          └─────────────────┘
```

**Key Difference:**
```
Forward Proxy: Works on behalf of CLIENT
Reverse Proxy: Works on behalf of SERVER
```

---

### How It Works

```
Client makes request:
Client → Reverse Proxy: "Get data"

Reverse Proxy:
- Receives request
- Selects appropriate server
- Forwards request to server

Server Response:
Server → Reverse Proxy: Returns data
Reverse Proxy → Client: Forwards data

Client never knows which server handled request!
```

**Key Point:**
```
Reverse Proxy hides SERVER from client
Client only knows about proxy
```

---

### Advantages

#### 1. Security (DDoS Protection)

```
Without Reverse Proxy:
Attacker → Server (direct attack)
Server IP exposed ✗
Server overwhelmed ✗

With Reverse Proxy:
Attacker → Reverse Proxy (attack absorbed)
Server IP hidden ✓
Server protected ✓

Reverse proxy has resources to handle DDoS
```

**Benefit:** Server protection

---

#### 2. Caching (CDN)

```
Global Setup:

Paris CDN ←─┐
            │
US CDN ←────┼─── Original Server (Singapore)
            │
India CDN ←─┘

Paris User Request:
User → Paris CDN: Check cache
       ├─ Found → Return from cache (fast!)
       └─ Not found → Fetch from Singapore
                      Store in cache
                      Return to user
```

**CDN = Reverse Proxy with caching**

**Benefit:** Reduced latency, faster response

---

#### 3. Reduced Latency

```
Without Reverse Proxy:
Paris User → Singapore Server
Latency: 200ms

With Reverse Proxy (CDN):
Paris User → Paris CDN → (cache hit)
Latency: 10ms

90% faster! ✓
```

**Benefit:** Geo-distributed caching

---

#### 4. Load Balancing

```
High Traffic:
1000 requests/sec

Reverse Proxy distributes:
Server 1 ← 333 req/sec
Server 2 ← 333 req/sec
Server 3 ← 334 req/sec

Load balanced ✓
```

**Benefit:** Distribute traffic, prevent overload

---

### Popular Example: CDN

```
CDN (Content Delivery Network) = Reverse Proxy

Examples:
- Cloudflare
- Akamai
- AWS CloudFront

Features:
✓ Geo-distributed
✓ Caching
✓ DDoS protection
✓ Load balancing
```

---

### Summary: Reverse Proxy

```
┌────────────────────────────────────────────┐
│         Reverse Proxy                      │
├────────────────────────────────────────────┤
│ Position: Server-side                      │
│ Protects: Servers                          │
│ Hides: Server IP from client               │
│                                            │
│ Advantages:                                │
│ ✓ Security (DDoS protection)              │
│ ✓ Caching                                  │
│ ✓ Reduced latency                          │
│ ✓ Load balancing                           │
│                                            │
│ Popular Example:                           │
│ - CDN (Content Delivery Network)          │
│                                            │
│ Use Cases:                                 │
│ - Protect servers                          │
│ - Distribute content globally              │
│ - Load balancing                           │
└────────────────────────────────────────────┘
```

---

## Key Comparisons

### 1. Proxy vs VPN

#### Proxy

```
┌────────┐      ┌───────┐      ┌────────┐
│ Client │─────►│ Proxy │─────►│ Server │
└────────┘      └───────┘      └────────┘

Features:
✓ IP address masking
✓ Caching
✓ Logging
✗ No encryption
✗ No secure tunnel
```

#### VPN

```
┌────────┐      ┌────────────┐      ┌────────────┐      ┌────────┐
│ Client │─────►│VPN Client  │─────►│ VPN Server │─────►│ Server │
└────────┘      └────────────┘      └────────────┘      └────────┘
                      │                    │
                      └────────────────────┘
                         VPN Tunnel
                      (Encrypted data)

Features:
✓ IP address masking
✓ Data encryption
✓ Secure tunnel
✓ End-to-end security
✗ No caching
```

#### Comparison

```
┌─────────────────┬─────────────┬─────────────┐
│    Feature      │    Proxy    │     VPN     │
├─────────────────┼─────────────┼─────────────┤
│ IP Masking      │     ✓       │      ✓      │
│ Encryption      │     ✗       │      ✓      │
│ Secure Tunnel   │     ✗       │      ✓      │
│ Caching         │     ✓       │      ✗      │
│ Logging         │     ✓       │      ✗      │
│ Speed           │   Faster    │   Slower    │
│ Security        │    Lower    │    Higher   │
└─────────────────┴─────────────┴─────────────┘
```

**Key Difference:**
```
Proxy: IP anonymity only
VPN: Full encryption + secure tunnel
```

---

### 2. Reverse Proxy vs Load Balancer

#### Reverse Proxy

```
Capabilities:
✓ IP anonymity
✓ Caching
✓ Logging
✓ Load balancing
✓ Security

Can work with:
- Single server (for caching, security)
- Multiple servers (for load balancing too)
```

#### Load Balancer

```
Capabilities:
✓ Load balancing ONLY

Requires:
- Multiple servers (no point with single server)
```

#### Comparison

```
┌─────────────────┬─────────────┬─────────────┐
│    Feature      │Rev. Proxy   │Load Balancer│
├─────────────────┼─────────────┼─────────────┤
│ IP Anonymity    │     ✓       │      ✗      │
│ Caching         │     ✓       │      ✗      │
│ Logging         │     ✓       │      ✗      │
│ Load Balancing  │     ✓       │      ✓      │
│ Single Server   │     ✓       │      ✗      │
│ Multi Server    │     ✓       │      ✓      │
└─────────────────┴─────────────┴─────────────┘
```

**Key Differences:**

```
1. Reverse Proxy can act as Load Balancer
   Load Balancer cannot act as Reverse Proxy

2. Reverse Proxy useful even with 1 server
   Load Balancer needs multiple servers

3. Reverse Proxy = Load Balancing + More features
   Load Balancer = Load Balancing only
```

---

### 3. Proxy vs Firewall

#### Firewall

```
┌─────────────────────────────────────┐
│          Firewall                   │
│                                     │
│  ┌─────┐  ┌─────┐  ┌─────┐         │
│  │Hole1│  │Hole2│  │Hole3│         │
│  │Rule1│  │Rule2│  │Rule3│         │
│  └─────┘  └─────┘  └─────┘         │
│                                     │
│  Rules based on:                    │
│  - Source IP                        │
│  - Destination IP                   │
│  - Port numbers                     │
│  - Protocol                         │
│                                     │
│  Works at: Packet level             │
│  (Network/Transport layer)          │
└─────────────────────────────────────┘

Packet Scanning:
- Checks packet headers
- Source/Destination IP
- Port numbers
- Allows/Blocks based on rules
```

#### Proxy

```
┌─────────────────────────────────────┐
│          Proxy                      │
│                                     │
│  Works at: Application level        │
│                                     │
│  Access to:                         │
│  - IP addresses                     │
│  - Port numbers                     │
│  - Application data                 │
│                                     │
│  Can inspect:                       │
│  - HTTP headers                     │
│  - Request/Response content         │
│  - Application-specific data        │
└─────────────────────────────────────┘
```

#### Comparison

```
┌─────────────────┬─────────────┬─────────────┐
│    Feature      │  Firewall   │    Proxy    │
├─────────────────┼─────────────┼─────────────┤
│ Works At        │ Packet level│ App level   │
│ Inspects        │ Headers only│ Full data   │
│ IP Filtering    │     ✓       │      ✓      │
│ Port Filtering  │     ✓       │      ✓      │
│ Content Filter  │     ✗       │      ✓      │
│ Caching         │     ✗       │      ✓      │
│ Anonymity       │     ✗       │      ✓      │
│ Setup           │ Network-wide│ Per-app     │
└─────────────────┴─────────────┴─────────────┘
```

**Proxy Firewall:**

```
Modern proxies can act as firewalls:
- Block based on rules (like firewall)
- Work at application level (like proxy)
- Content inspection (proxy advantage)

Example:
Block all requests to facebook.com
Block requests with specific keywords
```

**Key Differences:**

```
Traditional Firewall:
- Packet-level inspection
- Network/Transport layer
- Fast, lightweight

Proxy:
- Application-level inspection
- Can inspect content
- More features (caching, anonymity)

Proxy Firewall:
- Combines both
- Application-level blocking
- Content filtering
```

---

## Summary

### Forward vs Reverse Proxy

```
┌──────────────────┬─────────────────┬─────────────────┐
│    Aspect        │ Forward Proxy   │ Reverse Proxy   │
├──────────────────┼─────────────────┼─────────────────┤
│ Position         │ Client-side     │ Server-side     │
│ Protects         │ Clients         │ Servers         │
│ Hides            │ Client IP       │ Server IP       │
│ Direction        │ Outbound        │ Inbound         │
│ Use Case         │ Access control  │ Load balancing  │
│                  │ Bypass geo-rest │ DDoS protection │
│                  │ Privacy         │ Caching (CDN)   │
└──────────────────┴─────────────────┴─────────────────┘
```

### Visual Summary

```
FORWARD PROXY (Client-side):
Internal Network → Forward Proxy → Internet
Protects: Clients
Hides: Client identity from servers

REVERSE PROXY (Server-side):
Internet → Reverse Proxy → Internal Servers
Protects: Servers
Hides: Server identity from clients
```

### Key Takeaways

```
1. Proxy = Intermediary between client and server

2. Forward Proxy:
   - Client-side
   - Anonymity, caching, access control
   - Example: Corporate proxy

3. Reverse Proxy:
   - Server-side
   - Load balancing, DDoS protection, caching
   - Example: CDN

4. Proxy vs VPN:
   - Proxy: IP masking only
   - VPN: Full encryption + tunnel

5. Reverse Proxy vs Load Balancer:
   - Reverse Proxy can be load balancer
   - Load balancer cannot be reverse proxy

6. Proxy vs Firewall:
   - Firewall: Packet-level
   - Proxy: Application-level
   - Proxy firewall: Combines both

7. CDN = Reverse Proxy
   - Always remember this!
```

---

## Interview Tips

### Common Questions

**Q1: "What is the difference between forward and reverse proxy?"**

```
Answer:
"Direction and what they protect:

Forward Proxy:
- Client-side proxy
- Protects clients
- Hides client IP from server
- Use case: Corporate network, bypass restrictions

Reverse Proxy:
- Server-side proxy
- Protects servers
- Hides server IP from client
- Use case: Load balancing, CDN, DDoS protection

Example:
Forward: Company employees → Proxy → Internet
Reverse: Internet users → CDN → Company servers"
```

**Q2: "When would you use a proxy vs VPN?"**

```
Answer:
"Depends on security needs:

Proxy:
- Need: IP anonymity, caching
- Use when: Accessing geo-restricted content, corporate filtering
- Limitation: No encryption

VPN:
- Need: Full security, encrypted tunnel
- Use when: Sensitive data, remote work, public WiFi
- Limitation: Slower, no caching

Example:
Proxy: Watch region-locked video
VPN: Access company network remotely"
```

**Q3: "Can reverse proxy act as load balancer?"**

```
Answer:
"Yes, reverse proxy can act as load balancer.

Reverse Proxy capabilities:
✓ Load balancing
✓ Caching
✓ IP anonymity
✓ Logging
✓ Security

Load Balancer:
✓ Load balancing only

Key point:
- Reverse proxy = Load balancer + more features
- Load balancer ≠ Reverse proxy
- Reverse proxy useful even with 1 server (caching)
- Load balancer needs multiple servers"
```

**Q4: "What is CDN and how does it relate to proxy?"**

```
Answer:
"CDN is a reverse proxy with geo-distribution.

CDN (Content Delivery Network):
- Reverse proxy servers worldwide
- Caches content close to users
- Reduces latency
- DDoS protection

Example:
Server in Singapore
CDNs in: US, Europe, India

India user:
- Request → India CDN (10ms)
- Not → Singapore server (200ms)

CDN = Reverse Proxy + Geo-distribution + Caching"
```

**Q5: "Proxy vs Firewall - which provides better security?"**

```
Answer:
"Different types of security:

Firewall:
- Packet-level filtering
- Fast, lightweight
- Network/Transport layer
- Blocks based on IP, port, protocol

Proxy:
- Application-level filtering
- Can inspect content
- Application layer
- Blocks based on content, URLs

Best approach: Use both
- Firewall: First line of defense (packet filtering)
- Proxy: Second line (content filtering)

Modern solution: Proxy Firewall
- Combines both
- Application-level blocking + content inspection"
```

### Do's ✅

**1. Explain Direction**
```
"Forward proxy sits on client side, reverse on server side.
Forward protects clients, reverse protects servers."
```

**2. Use Real Examples**
```
"Forward proxy: Corporate network filtering
Reverse proxy: CDN like Cloudflare"
```

**3. Mention CDN**
```
"Always remember: CDN is a reverse proxy.
It's the most common real-world example."
```

**4. Explain Trade-offs**
```
"Proxy gives anonymity but no encryption.
VPN gives encryption but slower.
Choose based on needs."
```

### Don'ts ❌

**1. Don't Confuse Direction**
```
❌ "Forward proxy protects servers"
✓ "Forward proxy protects clients"
```

**2. Don't Say They're Same**
```
❌ "Reverse proxy and load balancer are same"
✓ "Reverse proxy can act as load balancer, but has more features"
```

**3. Don't Forget Application Level**
```
❌ "Proxy works at packet level"
✓ "Proxy works at application level"
```

### Key Points to Remember

```
1. Proxy = Intermediary (acts on behalf)

2. Forward = Client-side, Reverse = Server-side

3. Forward hides client, Reverse hides server

4. Proxy advantages:
   - Anonymity
   - Caching
   - Security
   - Load balancing (reverse)

5. Proxy vs VPN:
   - Proxy: IP masking
   - VPN: Encryption + tunnel

6. Reverse Proxy ⊃ Load Balancer
   (Reverse proxy includes load balancing)

7. Proxy vs Firewall:
   - Firewall: Packet-level
   - Proxy: Application-level

8. CDN = Reverse Proxy (always!)
```

---

**End of Lecture**

Understanding proxy servers is fundamental for system design. Forward proxy protects clients (anonymity, access control), while reverse proxy protects servers (load balancing, DDoS protection, CDN). Remember: CDN is always a reverse proxy. Know the differences between proxy vs VPN, load balancer, and firewall for interviews.

**Key Takeaway:** Forward proxy for client-side (corporate networks, geo-bypass), Reverse proxy for server-side (CDN, load balancing, security). Direction matters!
