Cookies, Sessions & Authentication
======

## The Stateless Problem
**HTTP is stateless**: the server doesn't remember you between requests.

### The Problem Visualized
![stateless-problem](./images/stateless-problem.excalidraw.png)

**Solution**? We need a way to "remember" the user.

### Analogy
Imagine going to a theme park:
- **No memory system**: You buy a ticket, enter. Next ride, guard asks for ticket again (you already thew it away!).
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

