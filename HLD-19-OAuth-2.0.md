# HLD-19: OAuth 2.0

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [Four Actors in OAuth](#four-actors-in-oauth)
3. [Five Grant Types](#five-grant-types)
4. [Authorization Code Grant](#authorization-code-grant)
5. [Implicit Grant](#implicit-grant)
6. [Resource Owner Password Credentials Grant](#resource-owner-password-credentials-grant)
7. [Client Credentials Grant](#client-credentials-grant)
8. [Security: CSRF Attack Prevention](#security-csrf-attack-prevention)
9. [Token Types](#token-types)
10. [Summary](#summary)
11. [Interview Tips](#interview-tips)

---

## Introduction

### What is OAuth?

```
OAuth = Open Authorization

OAuth 2.0:
- Authorization framework
- Enables secure third-party access
- To user's protected data
- Without sharing credentials
```

---

### Real-World Example

**Scenario:**

```
User (You):
- Already logged into Gmail
- Has protected data (profile, email, etc.)
- Wants to log into XYZ website

XYZ Website:
- Offers "Sign in with Gmail"
- Needs your Gmail data
- But you don't want to share Gmail password

OAuth Solution:
- User authorizes XYZ to access Gmail data
- Gmail provides limited access (token)
- XYZ gets data without password
- User stays secure
```

**Visual:**

```
┌──────────────┐
│     User     │
│  (Logged in  │
│   to Gmail)  │
└──────┬───────┘
       │
       │ Wants to login
       ▼
┌──────────────┐         ┌──────────────┐
│ XYZ Website  │         │    Gmail     │
│              │◄────────│  (Protected  │
│ "Sign in     │ OAuth   │    Data)     │
│  with Gmail" │  Flow   │              │
└──────────────┘         └──────────────┘

User authorizes XYZ to access Gmail data
Gmail provides token to XYZ
XYZ uses token to get user's data
```

---

### Key Points

```
1. Third-party access:
   - XYZ website = Third party
   - Needs access to Gmail data

2. User authorization:
   - User explicitly authorizes
   - "Yes, XYZ can access my Gmail data"

3. No password sharing:
   - XYZ never sees Gmail password
   - Only gets limited access token

4. Secure:
   - Token-based
   - Limited scope
   - Time-limited
```

---

## Four Actors in OAuth

### Overview

```
OAuth 2.0 has 4 main actors:
1. Resource Owner
2. Client
3. Authorization Server
4. Resource Server
```

---

### 1. Resource Owner

**Definition:**

```
Resource Owner = User who owns the data

Example:
- You (Shreyansh)
- Logged into Gmail
- Owns profile, email, contacts
- Can authorize access
```

**Visual:**

```
┌──────────────────┐
│ Resource Owner   │
│   (Shreyansh)    │
├──────────────────┤
│ Owns:            │
│ - Name           │
│ - Email          │
│ - Profile pic    │
│ - Date of birth  │
└──────────────────┘
```

---

### 2. Client

**Definition:**

```
Client = Application requesting access to protected data

Example:
- Instagram
- Facebook
- XYZ website
- Initiates OAuth flow
```

**Visual:**

```
┌──────────────────┐
│     Client       │
│   (Instagram)    │
├──────────────────┤
│ Wants:           │
│ - User's name    │
│ - User's email   │
│ - Profile data   │
└──────────────────┘
```

---

### 3. Authorization Server

**Definition:**

```
Authorization Server = Issues tokens after authentication

Example:
- Gmail Authorization Server
- Authenticates user
- Issues access tokens
- Issues refresh tokens
```

**Visual:**

```
┌──────────────────────┐
│ Authorization Server │
│  (Gmail Auth Server) │
├──────────────────────┤
│ Responsibilities:    │
│ - Authenticate user  │
│ - Get user consent   │
│ - Issue tokens       │
│ - Validate tokens    │
└──────────────────────┘
```

---

### 4. Resource Server

**Definition:**

```
Resource Server = Hosts protected data

Example:
- Gmail Resource Server
- Stores user's data
- Validates tokens
- Returns requested data
```

**Visual:**

```
┌──────────────────────┐
│   Resource Server    │
│ (Gmail Data Server)  │
├──────────────────────┤
│ Stores:              │
│ - User profiles      │
│ - Emails             │
│ - Contacts           │
│ - Settings           │
└──────────────────────┘
```

---

### Complete Flow Overview

```
┌─────────────┐
│  Resource   │
│   Owner     │ (1) Wants to login to Instagram
│ (Shreyansh) │     using Gmail
└──────┬──────┘
       │
       ▼
┌─────────────┐         ┌──────────────────┐
│   Client    │────────►│  Authorization   │
│ (Instagram) │ (2) Ask │     Server       │
│             │◄────────│  (Gmail Auth)    │
└──────┬──────┘ (3) Token└──────────────────┘
       │
       │ (4) Use token
       ▼
┌─────────────┐
│  Resource   │
│   Server    │
│(Gmail Data) │
└─────────────┘

Flow:
1. User wants to login
2. Instagram requests authorization
3. Gmail Auth issues token
4. Instagram uses token to get data
```

---

### Note: Combined Roles

```
Sometimes Authorization Server and Resource Server
are the same service (e.g., Gmail)

For clarity, we treat them as separate components
```

---

## Five Grant Types

### Overview

```
Grant Type = Mechanism to obtain access token

5 Grant Types:
1. Authorization Code Grant ★ (Most important)
2. Implicit Grant
3. Resource Owner Password Credentials Grant
4. Client Credentials Grant
5. Refresh Token Grant
```

---

### Grant Types Summary

```
┌────────────────────────┬─────────────┬──────────────┐
│      Grant Type        │  Use Case   │ Recommended? │
├────────────────────────┼─────────────┼──────────────┤
│ Authorization Code     │ Web apps    │     ★★★      │
│                        │ Most secure │              │
│                        │             │              │
│ Implicit               │ SPAs (old)  │      ✗       │
│                        │ Less secure │  Deprecated  │
│                        │             │              │
│ Password Credentials   │ Trusted apps│      ✗       │
│                        │ Legacy      │  Not advised │
│                        │             │              │
│ Client Credentials     │ M2M (API)   │      ✓       │
│                        │ No user     │              │
│                        │             │              │
│ Refresh Token          │ Token renew │      ✓       │
│                        │ All types   │              │
└────────────────────────┴─────────────┴──────────────┘

★ Authorization Code Grant is MOST IMPORTANT
```

---

## Authorization Code Grant

### Overview

```
Authorization Code Grant:
- Most secure
- Most widely used
- Two-step process:
  1. Get authorization code
  2. Exchange code for token
```

---

### Complete Flow

```
┌─────────────┐
│  Resource   │
│   Owner     │
└──────┬──────┘
       │
       │ (1) Click "Sign in with Gmail"
       ▼
┌─────────────┐
│   Client    │
│ (Instagram) │
└──────┬──────┘
       │
       │ (2) Redirect to Auth Server
       ▼
┌──────────────────┐
│  Authorization   │
│     Server       │
│  (Gmail Auth)    │
└──────┬───────────┘
       │
       │ (3) User authenticates & consents
       │ (4) Return authorization code
       ▼
┌─────────────┐
│   Client    │
│ (Instagram) │
└──────┬──────┘
       │
       │ (5) Exchange code for token
       ▼
┌──────────────────┐
│  Authorization   │
│     Server       │
└──────┬───────────┘
       │
       │ (6) Return access token + refresh token
       ▼
┌─────────────┐
│   Client    │
│ (Instagram) │
└──────┬──────┘
       │
       │ (7) Use token to access data
       ▼
┌─────────────┐
│  Resource   │
│   Server    │
│(Gmail Data) │
└─────────────┘
```

---

### Step-by-Step Flow

**Step 0: Registration (One-time)**

```
Client (Instagram) registers with Authorization Server

Request:
POST /register
{
  "client_name": "Instagram",
  "redirect_uris": [
    "https://instagram.com/callback",
    "https://instagram.com/oauth/callback",
    "https://instagram.com/auth/return"
  ]
}

Response:
{
  "client_id": "abc123",
  "client_secret": "xyz789secret"
}

Notes:
- client_id: Public identifier
- client_secret: CONFIDENTIAL (only client + auth server)
- redirect_uris: Max 3 URIs for callbacks
```

---

**Step 1: User Clicks "Sign in with Gmail"**

```
User action:
- On Instagram website
- Clicks "Sign in with Gmail" button
- Instagram initiates OAuth flow
```

---

**Step 2: Redirect to Authorization Server**

```
Instagram redirects user to Gmail Auth Server

Request:
GET /authorize?
  response_type=code&
  client_id=abc123&
  redirect_uri=https://instagram.com/callback&
  scope=email profile address&
  state=SJ111

Parameters:
- response_type: "code" (want authorization code)
- client_id: Instagram's ID from registration
- redirect_uri: Where to send code (optional)
- scope: Requested permissions (space-separated)
- state: Random value for CSRF protection
```

**Parameter Details:**

```
response_type:
- "code" = Authorization Code Grant
- Tells server what client expects

client_id:
- From registration step
- Identifies the client

redirect_uri:
- Optional
- If not provided, uses one from registration
- If provided, MUST match registration URIs

scope:
- Space-separated permissions
- Examples: "email", "profile", "address"
- What data client wants to access

state:
- Random unique value
- CSRF protection (explained later)
- Recommended but not mandatory
```

---

**Step 3: User Authentication & Consent**

```
At Gmail Authorization Server:

1. User Authentication:
   - If not logged in: Show login page
   - User enters Gmail credentials
   - Gmail authenticates user

2. User Consent:
   - Show consent screen:
     "Instagram wants to access:
      - Your email
      - Your profile
      - Your address
      
      Do you authorize?"
   
   - User clicks "Yes" (provides consent)
```

**Visual:**

```
┌────────────────────────────────┐
│   Gmail Authorization Server   │
├────────────────────────────────┤
│                                │
│  Login to Gmail:               │
│  Email: shreyansh@gmail.com    │
│  Password: ********            │
│  [Login]                       │
│                                │
│  ─────────────────────────     │
│                                │
│  Instagram wants to access:    │
│  ✓ Your email                  │
│  ✓ Your profile                │
│  ✓ Your address                │
│                                │
│  [Deny]  [Authorize]           │
│                                │
└────────────────────────────────┘
```

---

**Step 4: Return Authorization Code**

```
After user authorizes, Gmail redirects back to Instagram

Response (Redirect):
GET https://instagram.com/callback?
  code=AUTH_CODE_12345&
  state=SJ111

Parameters:
- code: Authorization code (one-time use)
- state: Same value Instagram sent

Instagram receives:
- Authorization code
- Can verify state matches
```

---

**Step 5: Exchange Code for Token**

```
Instagram exchanges authorization code for access token

Request:
POST /token
{
  "grant_type": "authorization_code",
  "code": "AUTH_CODE_12345",
  "redirect_uri": "https://instagram.com/callback",
  "client_id": "abc123",
  "client_secret": "xyz789secret"
}

Parameters:
- grant_type: "authorization_code"
- code: Authorization code from step 4
- redirect_uri: Optional (must match if provided)
- client_id: Instagram's ID
- client_secret: Instagram's secret (authentication)
```

**Why client_secret?**

```
Client must authenticate itself:
- Accessing protected data
- Must prove it's legitimate Instagram
- client_secret is proof
- Only Instagram and Gmail Auth know it
```

---

**Step 6: Receive Access Token & Refresh Token**

```
Gmail Auth Server returns tokens

Response:
{
  "access_token": "ACCESS_TOKEN_ABC",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "REFRESH_TOKEN_XYZ"
}

Fields:
- access_token: Use to access protected data
- token_type: "Bearer" (use in Authorization header)
- expires_in: 3600 seconds (1 hour)
- refresh_token: Get new access token when expired
```

**Token Details:**

```
access_token:
- Short-lived (e.g., 15 min, 1 hour)
- Used to access protected data
- Can be JWT or plain string

token_type: "Bearer"
- Tells client how to use token
- Bearer = Include in Authorization header
- Format: "Authorization: Bearer ACCESS_TOKEN_ABC"

expires_in:
- Lifetime in seconds
- 3600 = 1 hour
- After expiry, use refresh token

refresh_token:
- Long-lived (days, weeks, months)
- Used to get new access token
- Only used with Auth Server
```

---

**Step 7: Access Protected Data**

```
Instagram uses access token to get user data

Request to Resource Server:
GET /userinfo
Headers:
  Authorization: Bearer ACCESS_TOKEN_ABC

Resource Server:
1. Receives request with token
2. Validates token with Auth Server
3. If valid, returns requested data
4. If invalid, returns 401 Unauthorized
```

**Detailed Flow:**

```
┌─────────────┐         ┌─────────────┐
│  Instagram  │         │Gmail Resource│
│             │────────►│   Server     │
│             │ (1) GET │              │
│             │  +token │              │
└─────────────┘         └──────┬───────┘
                               │
                               │ (2) Validate token
                               ▼
                        ┌──────────────┐
                        │Gmail Auth    │
                        │   Server     │
                        └──────┬───────┘
                               │
                               │ (3) Token valid/invalid
                               ▼
                        ┌──────────────┐
                        │Gmail Resource│
                        │   Server     │
                        └──────┬───────┘
                               │
                               │ (4) Return data or 401
                               ▼
                        ┌─────────────┐
                        │  Instagram  │
                        └─────────────┘

If valid: Return user data (name, email, etc.)
If invalid: Return 401 Unauthorized
```

---

### Refresh Token Flow

**When Access Token Expires:**

```
Instagram's access token expired
Need new access token
Use refresh token

Request:
POST /token
{
  "grant_type": "refresh_token",
  "refresh_token": "REFRESH_TOKEN_XYZ",
  "client_id": "abc123",
  "client_secret": "xyz789secret",
  "redirect_uri": "https://instagram.com/callback"
}

Response:
{
  "access_token": "NEW_ACCESS_TOKEN_DEF",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "NEW_REFRESH_TOKEN_UVW"
}

Notes:
- New access token issued
- New refresh token issued (old one invalidated)
- Use new refresh token for next renewal
```

---

### Authorization Code Grant Summary

```
Flow:
1. Registration (one-time)
   → Get client_id, client_secret

2. User clicks "Sign in with Gmail"
   → Redirect to /authorize

3. User authenticates & consents
   → Get authorization code

4. Exchange code for token
   → POST /token with code

5. Receive tokens
   → access_token + refresh_token

6. Access protected data
   → Use access_token

7. Renew token when expired
   → Use refresh_token

Security:
✓ Two-step process
✓ client_secret required
✓ Authorization code one-time use
✓ CSRF protection (state parameter)
```

---

## Implicit Grant

### Overview

```
Implicit Grant:
- Single-step process
- Token returned directly (no code)
- Less secure
- NOT RECOMMENDED (deprecated)
- Used for SPAs (old approach)
```

---

### Flow

```
┌─────────────┐
│  Resource   │
│   Owner     │
└──────┬──────┘
       │
       │ (1) Click "Sign in"
       ▼
┌─────────────┐
│   Client    │
│    (SPA)    │
└──────┬──────┘
       │
       │ (2) Redirect to Auth Server
       ▼
┌──────────────────┐
│  Authorization   │
│     Server       │
└──────┬───────────┘
       │
       │ (3) User authenticates & consents
       │ (4) Return access token DIRECTLY
       ▼
┌─────────────┐
│   Client    │
│    (SPA)    │
└──────┬──────┘
       │
       │ (5) Use token to access data
       ▼
┌─────────────┐
│  Resource   │
│   Server    │
└─────────────┘

No authorization code step!
Token returned directly
```

---

### API Flow

**Request:**

```
GET /authorize?
  response_type=token&
  client_id=abc123&
  redirect_uri=https://spa.com/callback&
  scope=email profile&
  state=XYZ456

Key difference:
- response_type=token (not "code")
- Expects token directly
```

**Response:**

```
Redirect:
https://spa.com/callback#
  access_token=ACCESS_TOKEN_ABC&
  token_type=Bearer&
  expires_in=3600&
  state=XYZ456

Notes:
- Token in URL fragment (#)
- No refresh token
- No authorization code
- Less secure (token exposed in URL)
```

---

### Why Not Recommended?

```
Problems:
✗ Token exposed in URL
✗ Browser history may store token
✗ No refresh token
✗ Less secure than Authorization Code

Modern approach:
Use Authorization Code Grant with PKCE
(Proof Key for Code Exchange)
```

---

## Resource Owner Password Credentials Grant

### Overview

```
Password Credentials Grant:
- User provides credentials directly to client
- Client sends credentials to Auth Server
- Gets access token
- NOT RECOMMENDED (security risk)
- Only for highly trusted apps
```

---

### Flow

```
┌─────────────┐
│  Resource   │
│   Owner     │
└──────┬──────┘
       │
       │ (1) Enter Gmail username & password
       │     directly in Instagram app
       ▼
┌─────────────┐
│   Client    │
│ (Instagram) │
└──────┬──────┘
       │
       │ (2) Send credentials to Auth Server
       ▼
┌──────────────────┐
│  Authorization   │
│     Server       │
└──────┬───────────┘
       │
       │ (3) Return access token + refresh token
       ▼
┌─────────────┐
│   Client    │
│ (Instagram) │
└──────┬──────┘
       │
       │ (4) Use token to access data
       ▼
┌─────────────┐
│  Resource   │
│   Server    │
└─────────────┘

No authorization code
No redirect to Auth Server
Direct credential exchange
```

---

### API Flow

**Request:**

```
POST /token
{
  "grant_type": "password",
  "username": "shreyansh@gmail.com",
  "password": "myGmailPassword",
  "client_id": "abc123",
  "client_secret": "xyz789secret",
  "scope": "email profile"
}

Parameters:
- grant_type: "password"
- username: User's Gmail username
- password: User's Gmail password
- client_id: Client identifier
- client_secret: Client secret
- scope: Requested permissions
```

**Response:**

```
{
  "access_token": "ACCESS_TOKEN_ABC",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "REFRESH_TOKEN_XYZ"
}

Notes:
- Includes refresh token
- Can renew without re-entering password
```

---

### Refresh Token (Password Grant)

```
When access token expires:

Request:
POST /token
{
  "grant_type": "refresh_token",
  "refresh_token": "REFRESH_TOKEN_XYZ",
  "client_id": "abc123",
  "client_secret": "xyz789secret"
}

Note:
- NO username/password needed
- Just refresh token
- Get new access token
```

---

### Why Not Recommended?

```
Problems:
✗ Client sees user's password
✗ Violates OAuth principle (no password sharing)
✗ User must trust client completely
✗ If client compromised, password exposed

Use only when:
- Client is HIGHLY trusted
- Legacy systems
- Migration scenarios
```

---

## Client Credentials Grant

### Overview

```
Client Credentials Grant:
- Machine-to-Machine (M2M)
- No user involved
- Client IS the resource owner
- Used for API access
```

---

### Use Case

```
Scenario:
- Instagram backend service
- Needs to access Gmail API
- No specific user
- Service-to-service communication

Example:
Instagram's server wants to:
- Send emails via Gmail API
- Access shared resources
- Automated tasks
```

---

### Flow

```
┌─────────────┐
│   Client    │
│ (Instagram  │
│   Service)  │
└──────┬──────┘
       │
       │ (1) Request token with credentials
       ▼
┌──────────────────┐
│  Authorization   │
│     Server       │
└──────┬───────────┘
       │
       │ (2) Return access token
       ▼
┌─────────────┐
│   Client    │
│ (Instagram  │
│   Service)  │
└──────┬──────┘
       │
       │ (3) Use token to access API
       ▼
┌─────────────┐
│  Resource   │
│   Server    │
│ (Gmail API) │
└─────────────┘

No user authentication
No authorization code
No refresh token (can request new easily)
```

---

### API Flow

**Request:**

```
POST /token
{
  "grant_type": "client_credentials",
  "client_id": "abc123",
  "client_secret": "xyz789secret",
  "scope": "api.send_email"
}

Parameters:
- grant_type: "client_credentials"
- client_id: Service identifier
- client_secret: Service secret
- scope: API permissions
```

**Response:**

```
{
  "access_token": "ACCESS_TOKEN_ABC",
  "token_type": "Bearer",
  "expires_in": 3600
}

Notes:
- No refresh token
- When expired, request new token
- Same request, new token
- Simple renewal
```

---

### Why No Refresh Token?

```
Refresh token not needed because:
- Can easily request new token
- Just send client_id + client_secret
- No user interaction required
- No authorization code step

If token expires:
- Send same request again
- Get new access token
- Simple and straightforward
```

---

## Security: CSRF Attack Prevention

### What is CSRF?

```
CSRF = Cross-Site Request Forgery

Attack where:
- Attacker tricks user
- Into executing unwanted actions
- On a trusted site
```

---

### CSRF Attack in OAuth (Without State)

**Setup:**

```
Actors:
- Attacker
- Legitimate User (Shreyansh)
- Instagram (Client)
- Gmail Auth Server
```

**Attack Flow:**

```
Step 1: Attacker's Preparation
┌──────────┐         ┌──────────────┐
│ Attacker │────────►│ Gmail Auth   │
│          │ /authorize│   Server     │
│          │◄────────│              │
└──────────┘ code=ATK└──────────────┘

Attacker:
- Calls /authorize
- Gets authorization code: "ATK_CODE"
- SAVES it (doesn't use it)
- Authorization codes are one-time use

Step 2: Legitimate User's Request
┌────────────┐         ┌──────────────┐
│ Shreyansh  │────────►│  Instagram   │
│            │ Click   │              │
│            │ "Sign in│              │
│            │  Gmail" │              │
└────────────┘         └──────┬───────┘
                              │
                              ▼
                       ┌──────────────┐
                       │ Gmail Auth   │
                       │   Server     │
                       └──────────────┘

Shreyansh:
- Clicks "Sign in with Gmail" on Instagram
- Instagram redirects to Gmail Auth
- Waiting for authorization code

Step 3: Attacker Intercepts
┌──────────┐         ┌──────────────┐
│ Attacker │────────►│  Instagram   │
│          │ Sends   │              │
│          │ ATK_CODE│              │
└──────────┘         └──────────────┘

Attacker:
- Intercepts response
- Sends HIS code (ATK_CODE)
- Instagram thinks it's Shreyansh's code

Step 4: Instagram Uses Attacker's Code
┌──────────────┐         ┌──────────────┐
│  Instagram   │────────►│ Gmail Auth   │
│              │ Exchange│   Server     │
│              │ ATK_CODE│              │
│              │◄────────│              │
└──────────────┘  Token  └──────────────┘

Instagram:
- Exchanges ATK_CODE for token
- Gets access token
- Token is for ATTACKER's account

Step 5: Shreyansh Logged into Attacker's Account
┌────────────┐         ┌──────────────┐
│ Shreyansh  │────────►│  Instagram   │
│            │ Logged  │ (Attacker's  │
│            │   in    │   account)   │
└────────────┘         └──────────────┘

Result:
- Shreyansh thinks he's logged into HIS account
- Actually logged into ATTACKER's account
- Uploads photos → Go to attacker's account
- Shares data → Goes to attacker
```

**Visual Timeline:**

```
Time    Attacker            Shreyansh           Instagram
────────────────────────────────────────────────────────
T0:     /authorize
        Get ATK_CODE
        SAVE it
        
T1:                         Click "Sign in"
                            
T2:                         Redirect to
                            Gmail Auth
                            
T3:     INTERCEPT!
        Send ATK_CODE ──────────────────────►
        
T4:                                         Exchange
                                            ATK_CODE
                                            Get TOKEN
                                            (Attacker's)
                                            
T5:                         Logged in
                            (Attacker's
                             account!)

PROBLEM: Shreyansh in attacker's account!
```

---

### Solution: State Parameter

**How State Prevents CSRF:**

```
State = Random unique value
- Client generates random value
- Sends in /authorize request
- Auth Server returns same value
- Client verifies match
```

**Protected Flow:**

```
Step 1: Instagram Generates State
┌──────────────┐
│  Instagram   │
├──────────────┤
│ Generate:    │
│ state=SJ111  │
│ (Random,     │
│  unique)     │
└──────────────┘

Step 2: Send to Auth Server
GET /authorize?
  response_type=code&
  client_id=abc123&
  state=SJ111

Instagram sends state=SJ111

Step 3: Auth Server Returns Same State
Redirect:
https://instagram.com/callback?
  code=LEGIT_CODE&
  state=SJ111

Auth Server echoes back state=SJ111

Step 4: Instagram Validates
┌──────────────┐
│  Instagram   │
├──────────────┤
│ Sent: SJ111  │
│ Received:    │
│   SJ111      │
│ Match? ✓     │
│ Accept code  │
└──────────────┘
```

**Attack Fails with State:**

```
Time    Attacker            Shreyansh           Instagram
────────────────────────────────────────────────────────
T0:     /authorize
        state=ATK999
        Get code=ATK_CODE
        
T1:                         Click "Sign in"
                            
T2:                                         Generate
                                            state=SJ111
                                            
T3:                                         /authorize
                                            state=SJ111
                                            
T4:     INTERCEPT!
        Send:
        code=ATK_CODE
        state=ATK999 ───────────────────────►
        
T5:                                         Validate:
                                            Sent: SJ111
                                            Received: ATK999
                                            MISMATCH! ✗
                                            REJECT!

ATTACK BLOCKED! ✓
```

**Validation Logic:**

```
Instagram receives response:
{
  code: "SOME_CODE",
  state: "SOME_STATE"
}

Validation:
1. Check if state exists
   - If missing → REJECT
   
2. Check if state matches
   - Compare with sent state
   - If mismatch → REJECT
   
3. If match → ACCEPT
   - Use authorization code
   - Proceed with token exchange

Example:
Sent state: SJ111
Received state: SJ111
Match? YES ✓ → Accept

Sent state: SJ111
Received state: ATK999
Match? NO ✗ → Reject (CSRF attack!)
```

---

### State Parameter Summary

```
State Parameter:
- Random unique value
- Hard to guess
- Sent in /authorize
- Returned in callback
- Client validates match

Purpose:
✓ CSRF protection
✓ Ensures response matches request
✓ Prevents attacker code injection

Recommendation:
- HIGHLY RECOMMENDED
- Not mandatory but essential
- Always use in production
```

---

## Token Types

### Access Token

```
Access Token:
- Used to access protected resources
- Short-lived (15 min - 1 hour)
- Sent in Authorization header
- Can be JWT or plain string
```

**Format:**

```
JWT Token:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Plain String:
abc123xyz789token
```

**Usage:**

```
HTTP Request:
GET /userinfo
Headers:
  Authorization: Bearer ACCESS_TOKEN_ABC

"Bearer" indicates token type
```

---

### Refresh Token

```
Refresh Token:
- Used to get new access token
- Long-lived (days, weeks, months)
- Only used with Auth Server
- Not sent to Resource Server
```

**Characteristics:**

```
Lifetime:
- Access token: 15 min - 1 hour
- Refresh token: Days to months

Security:
- More sensitive than access token
- Stored securely
- Can be revoked

Usage:
- When access token expires
- Exchange for new access token
- Get new refresh token
```

---

### Token Type: Bearer

```
token_type: "Bearer"

Meaning:
- How to use the token
- Include in Authorization header
- Format: "Bearer <token>"

Example:
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Summary

### OAuth 2.0 Overview

```
OAuth 2.0 = Open Authorization Framework

Purpose:
- Secure third-party access
- To user's protected data
- Without sharing credentials
- Token-based authorization
```

---

### Four Actors

```
1. Resource Owner:
   - User who owns data
   - Grants authorization

2. Client:
   - Application requesting access
   - Uses tokens to access data

3. Authorization Server:
   - Authenticates user
   - Issues tokens
   - Validates tokens

4. Resource Server:
   - Hosts protected data
   - Validates tokens
   - Returns requested data
```

---

### Five Grant Types

```
1. Authorization Code Grant ★:
   - Most secure
   - Two-step (code → token)
   - Web applications
   - RECOMMENDED

2. Implicit Grant:
   - Single-step (direct token)
   - Less secure
   - DEPRECATED

3. Password Credentials Grant:
   - Direct credential exchange
   - NOT RECOMMENDED
   - Only for trusted apps

4. Client Credentials Grant:
   - Machine-to-Machine
   - No user involved
   - API access

5. Refresh Token Grant:
   - Renew access token
   - Using refresh token
   - All grant types
```

---

### Authorization Code Grant Flow

```
1. Registration:
   - Client registers
   - Gets client_id, client_secret

2. Authorization Request:
   - /authorize with state
   - User authenticates & consents
   - Get authorization code

3. Token Exchange:
   - /token with code + client_secret
   - Get access_token + refresh_token

4. Access Data:
   - Use access_token
   - In Authorization header

5. Renew Token:
   - Use refresh_token
   - Get new access_token
```

---

### Security Features

```
1. State Parameter:
   - CSRF protection
   - Random unique value
   - Validate match

2. Client Secret:
   - Client authentication
   - Confidential
   - Only client + auth server

3. Authorization Code:
   - One-time use
   - Short-lived
   - Exchanged for token

4. Token Expiry:
   - Access token: Short-lived
   - Refresh token: Long-lived
   - Limited exposure
```

---

### Key Differences

```
┌──────────────┬──────────┬──────────┬──────────┐
│ Grant Type   │   Steps  │  Secure? │   Use    │
├──────────────┼──────────┼──────────┼──────────┤
│ Auth Code    │    2     │   ★★★    │ Web apps │
│ Implicit     │    1     │    ✗     │ Deprecated│
│ Password     │    1     │    ✗     │ Legacy   │
│ Client Creds │    1     │    ✓     │ M2M/API  │
└──────────────┴──────────┴──────────┴──────────┘
```

---

## Interview Tips

### Common Questions

**Q1: "What is OAuth 2.0 and why is it used?"**

```
Answer:
"OAuth 2.0 is an authorization framework that enables
secure third-party access to user's protected data
without sharing credentials.

Example:
User wants to login to Instagram using Gmail.

Without OAuth:
- User shares Gmail password with Instagram
- Security risk
- Instagram has full access

With OAuth:
- User authorizes Instagram via Gmail
- Gmail issues limited access token
- Instagram gets only requested data
- No password sharing

Benefits:
✓ Secure (token-based)
✓ Limited scope (specific permissions)
✓ Revocable (user can revoke anytime)
✓ Time-limited (tokens expire)"
```

**Q2: "Explain the four actors in OAuth 2.0"**

```
Answer:
"Four actors in OAuth 2.0:

1. Resource Owner (User):
   - Owns the protected data
   - Example: You with Gmail account
   - Grants authorization

2. Client (Application):
   - Wants to access protected data
   - Example: Instagram
   - Requests authorization

3. Authorization Server:
   - Authenticates user
   - Issues access tokens
   - Example: Gmail Auth Server

4. Resource Server:
   - Hosts protected data
   - Validates tokens
   - Returns data
   - Example: Gmail Data Server

Flow:
Resource Owner authorizes Client
→ Client requests from Auth Server
→ Auth Server issues token
→ Client uses token with Resource Server"
```

**Q3: "What is Authorization Code Grant and why is it most secure?"**

```
Answer:
"Authorization Code Grant is the most secure OAuth flow.

Two-step process:
1. Get authorization code
2. Exchange code for token

Flow:
1. Client redirects user to Auth Server
2. User authenticates & consents
3. Auth Server returns authorization code
4. Client exchanges code + client_secret for token
5. Auth Server returns access token + refresh token

Why most secure:
✓ Two-step verification
✓ client_secret required (client authentication)
✓ Authorization code one-time use
✓ Code never exposed to browser
✓ CSRF protection (state parameter)
✓ Token not in URL

Contrast with Implicit Grant:
- Implicit: Token in URL (less secure)
- Auth Code: Token via backend (more secure)

Used for: Web applications, mobile apps (with PKCE)"
```

**Q4: "What is the state parameter and why is it important?"**

```
Answer:
"State parameter is for CSRF attack prevention.

How it works:
1. Client generates random unique value
2. Sends in /authorize request
3. Auth Server returns same value in callback
4. Client validates match

Example:
Request:  state=SJ111
Response: state=SJ111
Match? YES → Accept

CSRF Attack Scenario (without state):
1. Attacker gets authorization code
2. Intercepts user's OAuth flow
3. Injects attacker's code
4. User logged into attacker's account
5. User's data goes to attacker

With State:
1. Client sends state=SJ111
2. Attacker sends code with state=ATK999
3. Client checks: SJ111 ≠ ATK999
4. Mismatch → Reject
5. Attack blocked ✓

Recommendation:
- HIGHLY RECOMMENDED
- Always use in production
- Random, unique, hard to guess"
```

**Q5: "Explain access token vs refresh token"**

```
Answer:
"Two types of tokens in OAuth:

Access Token:
- Purpose: Access protected resources
- Lifetime: Short (15 min - 1 hour)
- Usage: Sent to Resource Server
- Format: JWT or plain string
- Header: Authorization: Bearer <token>

Refresh Token:
- Purpose: Get new access token
- Lifetime: Long (days - months)
- Usage: Sent to Auth Server only
- Never sent to Resource Server
- More sensitive

Flow:
1. Client gets both tokens initially
2. Uses access token to access data
3. Access token expires
4. Client uses refresh token
5. Gets new access token + new refresh token
6. Repeat

Why two tokens:
- Access token: Frequently used, short-lived
  (If compromised, limited damage)
- Refresh token: Rarely used, long-lived
  (Less exposure, more secure)

Security:
- Access token: Less sensitive (short-lived)
- Refresh token: More sensitive (can get new access tokens)
- Both should be stored securely"
```

**Q6: "What are the different OAuth grant types?"**

```
Answer:
"Five OAuth 2.0 grant types:

1. Authorization Code Grant ★:
   - Most secure, recommended
   - Two-step: code → token
   - Use: Web apps, mobile apps
   - Has refresh token

2. Implicit Grant:
   - Single-step, token directly
   - Less secure, deprecated
   - Use: SPAs (old approach)
   - No refresh token

3. Password Credentials Grant:
   - User gives credentials to client
   - Not recommended
   - Use: Highly trusted apps only
   - Has refresh token

4. Client Credentials Grant:
   - Machine-to-Machine
   - No user involved
   - Use: API access, backend services
   - No refresh token (easy to renew)

5. Refresh Token Grant:
   - Renew access token
   - Using refresh token
   - Works with grants that provide refresh token

Recommendation:
- Use Authorization Code Grant for most cases
- Use Client Credentials for M2M
- Avoid Implicit and Password grants"
```

**Q7: "How does token validation work?"**

```
Answer:
"Token validation flow:

1. Client requests data from Resource Server:
   GET /userinfo
   Authorization: Bearer ACCESS_TOKEN

2. Resource Server receives request

3. Resource Server validates token:
   - Calls Authorization Server
   - Sends token for validation
   - Auth Server checks:
     * Is token valid?
     * Is token expired?
     * What scopes does it have?

4. Auth Server responds:
   - Valid: Return token info + scopes
   - Invalid: Return error

5. Resource Server decides:
   - If valid: Return requested data
   - If invalid: Return 401 Unauthorized

Alternative (JWT):
- If token is JWT (self-contained)
- Resource Server can validate locally
- Check signature, expiry, claims
- No need to call Auth Server
- Faster but less control

Security:
- Token validation ensures only authorized access
- Expired tokens rejected
- Revoked tokens rejected
- Scope-based access control"
```

### Key Points to Remember

```
1. OAuth 2.0 = Authorization framework
   - Not authentication (that's OpenID Connect)
   - Token-based access

2. Four actors:
   - Resource Owner (user)
   - Client (app)
   - Authorization Server (issues tokens)
   - Resource Server (hosts data)

3. Authorization Code Grant:
   - Most secure ★
   - Two-step process
   - client_secret required
   - CSRF protection (state)

4. Tokens:
   - Access token: Short-lived, access data
   - Refresh token: Long-lived, renew access

5. Security:
   - State parameter (CSRF)
   - client_secret (client auth)
   - Token expiry (limited exposure)
   - One-time authorization code
```

### Do's ✅

**1. Explain with Real Examples**
```
"Instagram login with Gmail:
User authorizes Instagram to access Gmail data
Gmail issues token to Instagram
Instagram uses token to get user's email"
```

**2. Mention Security Benefits**
```
"Authorization Code Grant is most secure because:
- Two-step verification
- client_secret required
- CSRF protection with state
- Token not exposed in URL"
```

**3. Discuss Grant Type Selection**
```
"Use Authorization Code Grant for web apps
Use Client Credentials for M2M APIs
Avoid Implicit and Password grants"
```

### Don'ts ❌

**1. Don't Confuse Authentication and Authorization**
```
❌ "OAuth is for authentication"
✓ "OAuth is for authorization. OpenID Connect (built on OAuth) is for authentication"
```

**2. Don't Forget State Parameter**
```
❌ "State is optional, not important"
✓ "State is highly recommended for CSRF protection"
```

**3. Don't Recommend Deprecated Grants**
```
❌ "Use Implicit Grant for SPAs"
✓ "Use Authorization Code Grant with PKCE for SPAs"
```

---

**End of Lecture**

OAuth 2.0 is an authorization framework with four actors (Resource Owner, Client, Authorization Server, Resource Server) and five grant types. Authorization Code Grant is most secure and widely used, featuring two-step process (code → token), client_secret authentication, and CSRF protection via state parameter. Access tokens are short-lived for data access, while refresh tokens are long-lived for token renewal.

**Key Takeaway:** OAuth 2.0 enables secure third-party access without password sharing. Always use Authorization Code Grant for web apps, implement state parameter for CSRF protection, and understand the difference between access tokens (short-lived, data access) and refresh tokens (long-lived, token renewal)!

---

**Note:** For exact API specifications and parameters, refer to official OAuth 2.0 RFCs (RFC 6749, RFC 6750). The APIs shown here are simplified for educational purposes.
