REST API
=======

## REST (Representational State Transfer)
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
    - `/users/123` -> user with ID 123

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

