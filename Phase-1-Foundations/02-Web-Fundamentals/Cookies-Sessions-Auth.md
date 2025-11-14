Cookies, Sessions & Authentication
======

## The Stateless Problem
**HTTP is stateless**: the server doesn't remember you between requests.

### The Problem Visualized
![stateless-problem](./images/stateless-problem.excalidraw.png)

**Solution**? We need a way to "remember" the user.

### Analogy
Imagine going to a theme park:
- **No memory system**: You buy a ticket, enter. Next ride, guard asks for ticket again (you already threw it away!).
- **With memory (wristband)**: You get a wristband at entrance. Show it at every ride -> guard knows you paid.

**Wristband = Cookie/Token** 


## Cookies (The Browser's Memory)
A **cookie** is a small piece of data stored in your browser, sent with every request to the same domain.

### How Cookies Work
#### Step 1: Server Sets Cookie
```http
HTTP/1.1 200 OK
Set-Cookie: user_id=12345; HttpOnly; Secure
Set-Cookie: theme=dark; Max-Age=86400

<html>...</html>
```

#### Step 2: Browser Stores Cookie
```http
Browser stores:
- user_id=12345
- theme=dark
```

#### Step 3: Browser Sends Cookie Automatically
```http
GET /profile HTTP/1.1
Host: example.com
Cookie: user_id=12345; theme=dark
```

### Visual Flow
![browser-cookie](./images/browser-cookie.excalidraw.png)

### Cookie Attributes (Security is Critical!)
|Attribute|Purpose|Example|
|:--------|:------|:------|
|HttpOnly|JavaScript can't access (prevents XSS)|`Set-Cookie: id=123; HttpOnly`|
|Secure|Only sent over HTTPS|`Set-Cookie: id=123; Secure`|
|SameSite|CSRF protection|`SameSite=Strict` or `Lax`|
|Max-Age|Expiration time (seconds)|`Max-Age=86400`(1 day)|
|Domain|Which domains can access|`Domain=.example.com`|
|Path|Which paths can access|`Path=/api`|

### Security Example (Must Know!)
```http
BAD (Vulnerable):
Set-Cookie: session_id=abc123

GOOD (Secure):
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict
```

### Real-World Cookies Uses
|Site|What Cookies Store|
|:---|:-----------------|
|Amazon|Shopping cart items (even when not logged in)|
|Youtube|Video watch history, preferences|
|Google|Login session, language preference|
|News sites|"Already read" article tracking|

### Cookie Trade-offs
|✅Pros|❌Cons|
|:------|:-----|
|Automatic (browser handles it)|Size limit (4KB per cookie)|
|Persistent (survives browser close)|Sent with EVERY request (overhead)|
|Server-side validation|Vulnerable if not secured properly|
|Works for traditional web apps|Privacy concerns (tracking)|


## Sessions (Server-Side Memory)
A **session** is server-side storage that remembers user state.

### How Sessions Work
![sessions](./images/Sessions.excalidraw.png)

### Session Storage Options
|Storage|Speed|Scalability|Persistence|
|:------|:----|:----------|:----------|
|In-Memory|⚡ Fastest|❌ Single server only|Lost on restart|
|Redis|Very Fast|✅ Shared across servers|✅ Persistent|
|Database|Slower|✅ Shared|✅ Persistent|
|Memcached|Fast|✅ Shared|❌ Not persistent|

### Real-World Session Example (E-commerce)
```javaScript
{
  // Server-side session (Redis)
  session_id: "xyz789",
  user: {
    id: 123,
    email: "bob@example.com",
    role: "customer"
  },
  cart: [
    { product_id: 456, quantity: 2 },
    { product_id: 789, quantity: 1 }
  ],
  created_at: "2024-01-15T10:00:00Z",
  expires_at: "2024-01-15T22:00:00Z"
}
```
#### Why this matters:
- User's cart persist across pages.
- Even if user closes browser, cart saved (until expiration).
- Multiple servers can access same session (via Redis).

### Session vs Cookies: What's the Difference?
|Aspect|Cookie|Session|
|:-----|:-----|:------|
|Storage Location|Client (browser)|Server (memory/Redis)|
|Security|Less secure (client-side)|More secure (server-side)|
|Size Limit|4KB|No limit|
|Access|JavaScript can read (unless HttpOnly)|Only server can read|
|Lifespan|Configurable (days/weeks)|Usually shorter (hours)|

### Analogy
- **Cookie**: Hotel key card (you carry it, shows room nubmer).
- **Session**: Hotel database (front desk knows your reservation details).

## Authentication vs Authorization

### Key Difference
|Concept|Question|Example|
|:------|:-------|:------|
|Authentication|"Who are you?"|Login with username/password|
|Authorization|"What can you do?"|Admin can delete posts, user cannot|

### Visual Flow
![auth-flow](./images/auth-flow.excalidraw.png)

### Real-World Example (Google Drive)
![auth-example](./images/auth-example.excalidraw.png)

## JWT (JSON Web Token) - Modern Authentication
**JWT** is a self-contained token that stores user info **inside the token itself**.

### Structure of JWT
![jwt-structure](./images/jwt-structure.excalidraw.png)

#### Decoded JWT Example
```json
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload (Your Data)
{
  "user_id": 123,
  "email": "alice@example.com",
  "role": "admin",
  "exp": 1705334400  // Expiration timestamp
}

// Signature (Prevents Tampering)
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

### How JWT Works
![how-jwt-works](./images/how-jwt-works.excalidraw.png)

### JWT vs Session Comparison
|Feature|Session + Cookie|JWT|
|:------|:---------------|:--|
|Storage|Server (Redis)|Client (localStorage)|
|Scalability|Needs shared Redis|✅ Stateless (no server storage)|
|Security|✅ Server controls|Can't revoke easily|
|Size|4KB cookie|Larger (1-2KB token)|
|Expiration|Server controls|Must wait for expiry|
|Use Case|Traditional web apps|APIs, microservices, SPAs|

#### notes
- SPAs: Single Page Applications (ex: Gmail, Google Docs, Facebook, Twitter Web, Notion).
- Size: JWTs are larger (around 1-2KB) because they contain an encoded header, payload, and signature. Unlike a simple session ID, which is only a few bytes, but they're still smaller than the 4KB cookie storage limit.

### JWT Security Best Practice

#### ✅DO
- Store JWT in HttpOnly cookie (best) or sessionStorage.
- Use HTTPS only.
- Set short expiration (15-30 min).
- Implement refresh tokens.

#### ❌DON'T
- Store JWT in localStorage (XSS vulnerable).
- Put sensitive data in payload (it's readable!).
- Use weak secret keys.
- Skip expiration check.

### Real-World JWT Users
|Company|Why JWT?|
|:------|:-------|
|Auth0|Authentication service (JWT-based)|
|Firebase|Mobile/web auth (JWT tokens)|
|Stripe|API authentication|
|GitHub API|OAuth flow uses JWT|
|Microservices|Stateless auth across services|

## OAuth 2.0 (Third-Party Login)

**OAuth** lets users log in with Google/Facebook/GitHub without sharing passwords.

### Analogy
You want to print photos at a shop:
- **Bad way**: Give shop your phone password (risky!).
- **Good way (OAuth)**: Shop asks permission to access only photos (limited access).

### OAuth Flow ("Sign in with Google")
![oauth-flow](./images/oauth-flow.excalidraw.png)

### Visual OAuth Flow
![visual-oauth-flow](./images/visual-oauth-flow.excalidraw.png)

### OAuth Roles
|Role|Description|Example|
|:---|:----------|:------|
|Resource Owner|The User|You|
|Client|App requesting access|Your website|
|Authorization Server|Grants tokens|Google/Facebook|
|Resource Server|Stores user data|Google's servers|

### Real-World OAuth Examples
|App|OAuth Provider|What Access?|
|:--|:------------|:-----------|
|Airbnb|Login with Facebook|Name, email, profile picture|
|Canva|Login with Google|Email, Profile picture|
|Medium|Login with Twitter|Username, email|
|Netlify|Login with GitHub|Repo access (for deployment)|

### OAuth Trade-offs
|Pros|Cons|
|:---|:---|
|User doesn't create new password|Dependency on third-party|
|Less friction (quick signup)|Privacy concerns (data shared)|
|Secure (Google handles auth)|Complexity to implement|
|Social features (import firends)|User locked out if provider down|

## Common Authentication Patterns

### Pattern 1: Session-Based (Traditional)
![session-based](./images/session-based.excalidraw.png)

### Pattern 2: JWT-Based (Modern APIs)
![jwt-based](./images/jwt-based.excalidraw.png)

### Pattern 3: Hybrid (Best of Both)
![hybrid](./images/hybrid.excalidraw.png)

## Security Best Practices (Critical!)

### 1. XSS (Cross-Site Scripting)
![xss](./images/xss.excalidraw.png)

### 2. CSRF (Cross-Site Request Forgery)
![csrf](./images/csrf.excalidraw.png)

### 3. Session Hijacking
![session-hijacking](./images/session-hijacking.excalidraw.png)

## Engineering Decision Tree
![engineering-decision-tree](./images/engineering-decision-tree.excalidraw.png)

## Real-World Architecture Examples

### Example 1: E-Commerce (Amazon-style)
Authentication: Session + Cookie
- User logs in -> session_id in HttpOnly cookie
- Session stored in Redis (shared across servers)
- Shopping cart tied to session
- 30-day persistent cookie (remember me)

Why?\
✅ Easy to invalidate (logout works instantly)\
✅ Cart persists even if not logged in\
✅ Server controls everything

### Example 2: Mobile App (Instagram-style)
Authentication: JWT + Refresh Token
- Login -> Short JWT (15 min) + Refresh token (30 days)
- Mobile app stores tokens securely
- Expired JWT? Use refresh token to get new one

Why?\
✅ Stateless (scales easily)\
✅ Works offline (JWT has all info)\
✅ Can revoke refresh tokens (security)

### Example 3: Microservices (Netflix-style)
Authentication: JWT
- API Gateway validates JWT
- Microservices trust JWT (no DB lookup)
- User info embedded in JWT payload

Why?\
✅ No shared session store needed\
✅ Each service can read user info \
✅  Scales horizontally 

### Summary 
|Method|Storage|Scalability|Security|Use Case|
|:-----|:------|:----------|:-------|:-------|
|Cookie + Session|Server|Medium (needs Redis)|✅ High|Traditional web
|JWT|Client|✅ High (stateless)|⚠️Medium|APIs, SPAs|
|OAuth|Third-party|✅High|✅High|Social login|
|Hybrid (JWT + Refresh)|Both|✅High|✅High|Production apps|


## Key Takeaways

1. **HTTP is stateless** -> Need cookies(token) to remember users
2. **Cookies** -> Browser storage (4kb limit, auto-sent)
3. **Sessions** -> Server storage (secure, revocable)
4. **JWT** -> Self-contained token (scalable, stateless)
5. **OAuth** -> Third-party login (Google, Facebook)
6. **Always secrue cookies**: HttpOnly + Secure + SameSite
7. **Production apps** -> use hybrid approach (JWT + refresh tokens)
8. **Security first**: Prevent XSS, CSRF, session hijacking