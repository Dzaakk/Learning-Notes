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

## 3. gRPC (Google Remote Procedure Call)
gRPC is a **moder, high-performance** alternative to REST, designed for **service to service communication** inside distributed systems.

### How It Works:
- Instead of HTTP/JSON, gRPC uses **HTTP/2** and **binary data (Protocol Buffers / protobuf)**.
- You define your API in a `.proto` file:
```proto
service UserService{
    rpc GetUser (UserRequest) returns (UserResponse);
}
```
- gRPC automatically generates client and server code in many languages.

### Key Concepts:
- **Strong typing**: APIs are schema-defined via `.proto` files.
- **Bidirectional streaming** : both client and server can send messages continuously.
- **Compact and fast**: binary format -> less bandwith, lower latency.

### Pros:
- Extremely fast (uses HTTP/2 and protobuf).
- Great for microservices and backend to backend calls.
- Auto generates SDKs for multiple languages.
### Cons:
- Harder to debug (binary, not human readable).
- Not natively supported by browsers.
- Requires more setup and tooling.

### Use it for:
- Microservices inside your infrastructure.
- High performance systems (streaming, IoT, or internal RPC calls).

## 4. GraphQL (Flexible Data Queries)
GraphQL (by Facebook) is a **query language for APIs** that lets clients ask for exactly what they need — no more, no less.

Instead of having fixed endpoints like `/users` or `/posts`, you have **one endpoint** (usually `/graphql`), and the client defines what fields it wants.

### Example:
#### Request:
```graphql
{
  user(id: 123) {
    name
    posts {
      title
    }
  }
}
```
#### Response:
```graphql
{
  "user": {
    "name": "Alice",
    "posts": [
      {"title": "My first post"},
      {"title": "GraphQL is great"}
    ]
  }
}
```

### Key Ideas:
- Client defines the shape of the response.
- One endpoint handles all queries.
- Uses **schema and resolvers** (like a type system for your data).

### Pros:
- No over-fetching or under-fetching.
- Great for frontend flexibility.
- Strongly typed — clients know exactly what's avaliable.
### Cons:
- More complex to cache (every query is unique).
- Harder to secure and monitor.
- Needs extra server logic to resolve queries.

### Use it for:
- Public APIs or frontend-heavy apps (like mobile or dashboards).
- System with lots of relational data (users, posts, comments, etc.).
