Foundations Pack
==================

Topics on this file : **Clien-Server Model, Socket, TCP, UDP, WebSocket**


## 1. The Client-Server Model 
Internet runs on a simple pattern: **a client asks, a server answers**

- **Client** = the requester (e.g. your phone, browser, or app).
- **Server** = the provider (the machine hosting the data or service).

### Flow
1. Client sends a request.
2. Server processes that request.
3. Server sends a response back.
---

## 2. Sockets (The Conversation Channel)
A **socket** is like the phone line between the client and the server.
- It represents one endpoint of a conncetion.
- It's identified by an IP address + port number (e.g. 192.168.0.1:8080).
- Sockets allow programs to send and receive data over a network

> Think of it like: 'The client *dials* the server's socket -> a connection is made -> they exchange data.

### Why it matters:
Everything from HTTP to WebSocket to FTP sits on top of sockets.
if sockets didn't exist, you'd have no way for apps to communicate through the network stack

## 3. TCP (Transmission Control Protocol)
TCP is the reliable, polite delivery guy of the internet.

When data travels across the internet, it's chopped into **packets** (small chunks). TCP ensures:
- Every packet arrives in **order**.
- Missing packets are **re-sent**.
- The sender doesn't overwhelm the receiver.

### Analogy
Imagine you send a 5-page letter.
TCP makes sure the receiver gets **all 5 pages, in the rigth order**, even if some got lost along the way.

### Used for:
- Web browsing (HTTP/HTTPS) 
- Email
- File Transfer

### Downside:
Because it ensures reliability, it's slower (more handshake, error checking).

## 4. UDP (User Datagram Protocol)
UDP is the fast but *carefree* delivery guy.
- It sends packets **without checking** if they arrive or not.
- No guarantee of order, no retries — just ***fire and forget***.

### Analogy
You shout a message into a crowd — some people might hear it, some might not. But you don't wait for confirmation.

### Used for:
- Real-time stuff: video calls, gaming, streaming — where speed > reliability.
- DNS lookups.

## Trade-off:
- Faster, less overhead.
- If some data is lost, too bad — it won't be resent.

## 5. WebSocket (Real-Time Over the Web)
**WebSocket** sits on top of *TCP*, and it allows **two-way** (**full-duplex**) communication between client and server

In normal HTTP, communcation is **one-way** — client asks, server responds. With WebSocket, **both can talk and listen instantly**.

### Analogy
HTTP is like sending letters — one at a time.

WebSocket is like a live phone call — both can talk and listen instantly.

### Used for:
- Chat apps
- Real-time dahboards
- Multiplayer games
- Notifications

### Technical note:  
It starts as an HTTP connection, then **upgrades** to a persistent WebSocket connection.

## 6. Polling & Server-Sent Events

### The Problem:
Sometimes clients need **real-time updates** from server, but WebSocket is too heavy or you don't need **two-way** communication.

### Use cases:
- Notifications
- Live scores
- Stock prices
- System monitoring dashboards
---

### Solution 1: Short Polling (The Naive Way)
**How it works**: Client requests to server **repeatedly** every few seconds.
#### Example:
```javascript
// Client-side (browser)
setInterval(() => {
    fetch('/api/notifications')
    .then(res => res.json())
    .then(data => updateUI(data));
}, 5000); // Every 5 seconds
```

### Pros:
- Simple to implement.
- Works everywhere (just HTTP).
### Cons:
- **Wasteful** - most requests return "no new data".
- High server load (many empty requests).
- Not truly real-time (max 5s delay if polling every 5s).
- Battery drain on mobile.
### When to use:
- Quick prototype.
- Updates every 30s+ is acceptable.
- Low trafficv systems.

### Solution 2: Long Polling (Smarter Polling)

**How it works**: Client requests, server **holds the connection open** until there's new data (or timeout).

#### Flow:
1. Client: "Any new data?"
2. Server: waits... waits... waits...
3. Server: "Yes! Here's new data" (close connection)
4. Client: immediately re-request "Any new data?"

### Pros:
- Less wasteful than shor polling.
- Near real-time (data comes immediately when available).
- Works over HTTP.

### Cons:
- Server has many **hanging connections** (resource intensive).
- Complex to implement properly.
- Timeout management tricky.
- Not truly bidirectional.

### When to use:
- Need near real-time but WebSocket overkill.
- Legacy systems that can't do WebSocket.
- Chat applications (WhatsApp Web used this before WebSocket).