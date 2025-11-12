Sockets (The Conversation Channel)
======
A **socket** is like the phone line between the client and the server.

- It represents one endpoint of a conncetion.
- It's identified by an IP address + port number (e.g. 192.168.0.1:8080).
- Sockets allow programs to send and receive data over a network.

> Think of it like: 'The client *dials* the server's socket -> a connection is made -> they exchange data.

### Analogy
Socket:  Your phone number + extension.
- **IP Address**: Your building address (192.168.1.1).
- **Port Number**: Your apartment number (8080).
- **Socket**: Complete address so messages reach the right place.

### Visual Diagram
![socket](./images/Socket.excalidraw.png)

### Real-World Examples
- **Web Server**: Listens on port 80 (HTTP) or 443 (HTTPS).
- **Database**: PostgreSQL listens on port 5432, MySQL on 3306.
- **Game Server**: Minecraft server listens on port 25565.

### Key Concepts
- **Well-known ports**: 0-1023 (HTTP=80, HTTPS=443, SSH=22).
- **Registered ports**: 1024-49151 (apps can register).
- **Dynamic ports**: 49152-65535 (temporary, assigned automatically).

### Why it matters:
Everything from HTTP to WebSocket to FTP sits on top of sockets.
if sockets didn't exist, you'd have no way for apps to communicate through the network stack.