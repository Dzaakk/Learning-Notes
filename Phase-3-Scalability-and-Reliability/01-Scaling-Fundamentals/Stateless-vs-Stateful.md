Stateless vs Stateful Architecture
===

# Overview
State refers to data that an application needs to remember between requests. How you handle state determines your system's scalability and complexity.

# Stateless Architecture

## Definition
Server doesn't store any client session data. Each request contains all information needed to process it.

## How it Works
![stateless](./images/stateless.png)
All servers are **interchangeable** - no server "knows" the user

## Characteristics
- Each request is **independent**
- Server doesn't remember previous requests
- Session data stored **externally** (DB, cache, client)
- Any server can handle any request

## Pros
- **Easy to scale horizontally** (just add servers)
- **High availability** (server fails? Route to another)
- **Load balancing is simple** (any algorithm works)
- **No sticky session** needed
- **Easier deployment** (restart servers anytime)
- **Better fault tolerance**

## Cons
- Need external storage (Redis, DB)
- Network overhead (fetch state every request)
- Slightly higher latency
- More complex architecture initially

## Where State Lives
1. **Client-side:** JWT tokens, cookies
2. **External cache:** Redis, Memcached
3. **Database:** Session store
4. **URL/Headers:** Request contains everything