Scaling Patterns
===

# Overview
Scaling patterns are proven architectural approaches to handle growth. These patterns solve specific bottlenecks as your system scales.

# 1. Load Balancing
Distribute incoming traffic across multiple servers

## How It Works
![load balancing](./images/load-balancing.png)

## Algorithms

### Round Robin
- Distributes requests sequentially
- Simple, even distribution
- Use when: have equal capacity

### Least Connections
- Routes to server with fewest active connections
- Use when: Requests have variable duration

### IP Hash
- Same client → same server (based on IP)
- Use when: Need session affinity

### Weighted Round Robin
- Distribute based on server capacity
- Use when: Servers have different specs

## Key Concepts
- **Health checks:** Remove unhealthy servers from pool
- **Layer 4 (TCP):** Fast, basic routing
- **Layer 7 (HTTP):** Smart routing based on content (path, headers)

## Example Use Case
E-commerce site with 3 web servers:
- Load balancer receives user request
- Routes to server with least connections
- If server fails health check →  route to healthy servers
- Use never notice the failure

# 2. Caching
Store frequently accessed data in fast storage to reduce load on database/backend

## Cache Layers
![cache layers](./images/cache-layers.png)

## Caching Strategies

### Cache-Aside (Lazy Loading)
1. Check cache → Hit? Return
2. Miss? Query DB
3. Store in cache
4. Return to user
   
**Best for:** Read-heavy workloads

### Write-Through
1. Write to cache
2. Write to DB (synchronously)
3. Return to user

**Best for:** Data consistency critical

### Write-Behind (Write-Back)
1. Write to cache
2. Return to user immediately
3. Write to DB asynchronously (batched)

**Best for:** Write-heavy workloads\
**Risk:** Potential data loss

### Refresh-Ahead
Automatically refresh popular items before they expire

**Best for:** Predictable access patterns

## Cache Invalidation
The hardest problem in computer science
1. **TTL (Time To Live):** Auto-expire after X seconds
2. **Event-based:** Invalidate when data changes
3. **Manual:** Clear specific keys when needed

### Example: Product Page
```js
// Check cache first
let product = await cache.get(`product:${id}`);

if (!product) {
  // Cache miss - query DB
  product = await db.query('SELECT * FROM products WHERE id = ?', id);
  
  // Store in cache for 1 hour
  await cache.set(`product:${id}`, product, 3600);
}

return product;
```
**Impact:** 1000 req/sec → only 10 DB queries/sec (if 99% cache hit rate)

# 3. Database Replication
Create copies of database to distribute read load and improve availability

## Master-Slave (Primary-Replica)
![master slave](./images/master-slave-replica.png)

### Characteristics
- Master handles all writes
- Replicas handle reads
- Data replicated asynchronously (usually)

### Benefits
- Scale reads horizontally
- High availability (failover to replica) 
- Geographic distribution

### Trade-offs
- Replication lag (eventual consistency)
- Writes don't scale (single master)

## Master-Master (Multi-Master)
![master master](./images/master-master-replica.png)

### Benefits
- Write scaling
- No sinle point of failure

### Trade-offs
- Conflict resolution needed
- Complex to manage

## Example: Social Media Feed
>Write: Post creation → Master DB\
>Read: Feed queries → Load balanced across 5 read replicas\
>Result: Handle 10K reads/sec, 100 writes/sec

# 4. Database Sharding
Split database horizontally across multiple servers. Each shard contains subset of data

## Sharding Strategies

### By Range (e.g., User ID)
>Shard 1: Users 1-1M\
>Shard 2: Users 1M-2M\
>Shard 3: Users 2M-3M

**Problem:** Uneven distribution (hotspots)

### By Hash (User ID % N)
> hash(user_id) % 3\
> → Shard 0, 1, or 2

✅ **Even distribution**\
❌ **Adding shards** = re-sharding everything


### By Geography
> US users → US Shard\
> EU users → EU Shard
> Asia users → Asia Shard

✅**Low Latency**\
✅**Data residency**

### By Entity Type
>Shard 1: Users table\
>Shard 2: Posts table
>Shard 3: Comments table

### Challenges
- **Cross-shard queries:** Slow (neeed to query shards)
- **Distributed transactoins:** Complex
- **Re-sharding:** Paintful as you grow
- **Hotspots:** Popular users/data overload one shard

### Example: Instagram
Shard by User ID:
- User 12345 → Shard 5
- All their posts, likes, followers, Shard 5
- Query "show user's posts" → single shard (fast!)
- Query "Show global trending" → all shards (slow)

**Whent to shard:** When single DB can't handle write load (~10k writes/sec)
 
