REST API (Representational State Transfer)
=======

## Definition
REST = Architectural style using HTTP for APIs
- Resource = URLs `/users/123`
- Actions = HTTP methods (GET, POST, PUT, DELETE)
- Stateless = No server memory between requests

**Analogy**: Restaurant menu (fixed options, you pick from list)

## Why It Matters
- **Universal** - Everyone knows HTTP
- **Simple** - Easy to learn, debug with curl
- **Cacheable** - HTTP caching works automatically
- **Scalable** - Sateless = add more servers easily

## Use Cases
### ✅Use When:
- Public APIs (third-party developers)
- Simple CRUD operations
- Caching important (CDN,, browser cache)
- Team knows HTTP/REST well

### ❌Avoid When:
- Real-time updates needed -> WebSocket
- Complex nested queries -> GraphQL
- High performance microservices -> gRPC
- Mobile bandwith critical -> GraphQL


## Trade-offs
|✅Pros|❌Cons|
|-|-|
|Easy to learn|Over-fetching (get all fields)|
|HTTP caching built in|Under-fetching (N+1problem)|
|Stateless (scalable)|Need versioning (/v1/, /v2/)|
|Human-readable JSON|Not real time|
|Mature tooling|Multiple roundtrips|

## Key Patterns
### HTTP Methods
- `GET` -> read
- `POST` -> create
- `PUT` -> replace
- `PATCH` -> update part
- `DELETE` -> remove
- Use **URLs (endpoints)** to represent resources:
    - `/users` -> all users
    - `/users/123` -> user with ID 123


### The N+1 Problem
`GET` /users/123              (1 request)\
`GET` /users/123/posts        (1 request)\
`GET` /posts/1/comments       (1 request)\
= 3 requests for 1 page

Solution: GraphQL or compound endpoints


## Interview Tips
**Question**: "Should we use REST?"

**Answer**:
REST is perfect for [use case] because:\
1. Simple CRUD operations
2. HTTP caching reduces load
3. External developers expect it

Trade-offs:
- Over fetching on mobile (can add field filtering)
- N+1 problem (can use compound endpoints or upgrade to GraphQL later)


## Quick Reference
### ✅Best Practices:
- Use proper HTTP methods
- Return correct status codes
- Version API (/v1/)
- Paginate large response
- Rate limit requests
- HTTPS only

### ❌Anti Patterns:
- POST for everything
- All responses 200 OK
- No versioning
- Expose stack traces

---
### Notes
- REST is **stateless** (no session memory between requests)
- **Over-fetching** (getting more data than needed)
- **Under-fetching** (needing multiple reequests)
- Not ideal for **real-time** or **microservice to microservice** communication.


