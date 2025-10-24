Api Design Pack
==================

Topics on this file : **REST, gRPC, GraphQL**

## 1. What's an API, Really?
**API** stands for **Application Programming Interface** — a way for one system or app to talk to another in a *structured*, *predictable format*.
- A **client** (mobile app, browser, or another service) sends a **request**.
- The **API** defines what's allowed to ask for, how to ask it, and what format the answer will be in.

#### Think of an API like a restaurant menu:
- You don't enter the kitchen — you just pick form the menu (the API endpoints).
- You know what you'll get back (the data format).
- You can't order something that's not listed.

APIs keep system modular — so parts of a system can evolve independently.

## 2. REST (Representational State Transfer)
REST is the **classic** web API style — simple, widely used, built on top of HTTP.

### Core Ideas:
- Use **HTTP methods** to represent actions:
    - `GET` -> read
    - `POST` -> create
    - `PUT` -> replace
    - `PATCH` -> update part
    - `DELETE` -> remove
- Use **URLs (endpoints)** to represent resources:
    - `/users` -> all users
    - `users/123` -> user with ID 123

### You should know:
- REST is **stateless** (no session memory between requests).
- Data is usually sent as **JSON**.
- **HTTP status codes** indicate success/failure (200, 404, 500, etc.).

### Pros:
- Easy to learn and debug (uses HTTP directly).
- Caches naturally (thanks to HTTP).
- Language-agnostic — any client can use it.
### Cons:
- Can be **over-fetching** (getting more data than needed).
- Or **under-fetching** (needing multiple reequests).
- Not ideal for **real-time** or **microservice to microservice** communication.

