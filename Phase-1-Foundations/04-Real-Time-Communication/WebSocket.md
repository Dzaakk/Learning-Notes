WebSocket
======

## WebSocket (Real-Time Over the Web)
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

