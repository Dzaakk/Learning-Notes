Server-Sent Events
======

### Server-Sent Events (SSE) - The Modern way
**How it works**: Server pushes data to client over a **single, long-lived HTTP connection**.

**Key difference from polling**:
- Connection stays open indefinitely.
- Server **pushes** data when available (client doesn't ask repeatedly).
- **One-way only** (server -> client).

### Pros:
- **Efficient**: one connection, multiple updates.
- **Simple**: built into browsers (no library needed).
- **Auto-reconnect**: browser handles reconnection automatically.
- Works over HTTP (firewall-friendly).
- Lower server overhead than long polling.

### Cons:
- **One-way only** (server -> client).
- Limited browser support for IE (Internet Explorer).
- Connection limits (browser max 6 connections per domain).

### When to use:
- **Real-time updates** where client only receives (doesn't send).
- Live dashboards.
- Notifications.
- Stock tickers.
- Social media feeds.
- Logs streaming.

### Engineering Takeways:
1. **Don't use WebSocket for everything**, it's overkill if you only need server -> client.
2. **SSE is underrated**, it's perfect for dashboards, notifications, feeds.
3. **Short pollingis OK for prototypes**, but plan to upgrade.
4. **Long polling is dying**, SSE and WebSocket killed it.
5. **Always implement reconnection logic**, networks are unreliable.