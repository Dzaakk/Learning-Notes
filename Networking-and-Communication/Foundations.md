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

## 2 Sockets (The Conversation Channel)
A **socket** is like the phone line between the client and the server.

- It represents one endpoint of a conncetion.
- It's identified by an IP address + port number (e.g. 192.168.0.1:8080).
- Sockets allow programs to send and receive data over a network

> Think of it like: 'The client *dials* the server's socket -> a connection is made -> they exchange data.

### Why it matters:
Everything from HTTP to WebSocket to FTP sits on top of sockets.
if sockets didn't exist, you'd have no way for apps to communicate through the network stack