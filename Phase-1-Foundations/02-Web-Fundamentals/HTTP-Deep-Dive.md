HTTP Deep Dive
======

## What is HTTP?
**HTTP (Hypertext Transfer Protocol)** is the foundation of the web — it defines how messages are formatted and transmitted between a **client** (like your browser or app) and a **server** (like a web service).

It's a **stateless, text-based protocol** built on top of **TCP**.\
That means:
- Every request is independent (the server doesn't remember past requests).
- Messages are readable (you can literally open them in plain text).

###  HTTP Request & Response Structure
Request
```http
GET /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>
```
- **Method**: what action to perform (GET, POST, PUT, DELETE) 
- **Path**: what resource (e.g., `/users`)
- **Headers**: metadata (auth, content type, etc.)
- **Body (optional)**: data sent with the request (e.g., JSON for POST)

Response
```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 1, "name": "Alice"}
```
- **Status Code**: tells if the request succeeded or failed 
- **Headers**: metadata (e.g., caching, cookies)
- **Body**: actual data from the server

### Limitations of Plain HTTP
Http itself is **not secure** — everything is sent as plain text.\
That means:
- Password, tokens, and cookies can be intercepted.
- Man-in-the-middle (MITM) attacks are possible.
- Data integrity isn't guaranteed.

## HTTPS (HTTP Secure)
**HTTPS** = **HTTP** + **SSL/TLS encryption** 

It ensures three critical security guarantees:
1. Encryption — Nobody can read your data in transit.
2. Integrity — Data can't be tampered with unnoticed.
3. Authentication — You're talking to the real server, not an imposter.

### How it works (simplified):
1. Browser connects to the server.
2. Server sends its **SSL/TLS certificate** (issued by a trusted Certificate Authority).
3. Browser verifies it's valid and from the right domain.
4. They agree on encryption keys.
5. All communication becomes encrypted.

### Key SSL/TLS Concepts
- **TLS handshake**: the initial setup of encryption.
- **Certificate Authority (CA)**: third-party that verifies the server's identity.
- **Public & private keys**: asymmetric cryptography used to encrypt/decrypt data.
- **HSTS**: enforces HTTPS only (prevents downgrade attacks).

## The Big Picture
When you type a URL:
1. Your browser finds the server (DNS + IP).
2. Opens a TCP connection (maybe through a load balancer).
3. Sends an **HTTP/HTTPS request**.
4. The server processes it and replies with an **HTTP response**.
5. Your browser renders it — securely, if using HTTPS.