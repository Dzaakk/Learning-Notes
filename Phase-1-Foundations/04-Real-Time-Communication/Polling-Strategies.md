Polling 
======

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
- Less wasteful than short polling.
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

