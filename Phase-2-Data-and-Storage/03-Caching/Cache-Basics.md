Cache Basics
===

# What is Caching?
Caching is a technique to store frequently accessed data in a faster storage layer (cache) to reduce latency and improve system performance. Instead of fetching data from the original source (database, API, disk) every time, we serve it from cache.

**Key Principle:** Trade-off between speed and freshness of data.

# Why Use Caching?
- **Reduce Latency:** Cache is typically in-memory, much faster than disk/network
- **Reduce Load:** Fewer requests to databases/APIs
- **Cost Savings:** Less compute and bandwidth usage
- **Improve Scalability:** Handle more requests with the same infrastructure

# Common Cache Locations

## 1. Client-Side Cache
- **Bowser Cache:** HTML, CSS, JS, images
- **Mobile App Cache:** Local storage, SQLite
- **Use Case:** Static assets, user preferences

## 2. CDN (Content Delivery Network)
- **Location:** Edge servers near users
- **Use Case:** Static content, images, videos

## 3. Application-Level Cache
- **In-Memory:** Redis, Memcached
- **Local Cache:** In-process cache (e.g., Caffeine, Guava)
- **Use Case:** Database query results, API response, session data

## 4. Database Cache
- **Query Cache:** MySQL query cache
- **Buffer Pool:** InnoDB buffer pool
- **Use Case:** Frequently accessed rows/queries

# Cache Hit vs Cache Miss
- **Cache Hit:** Data found in cache → Fast response
- **Cache Miss:** Data not in cache → Fetch from origin, then store in cache
- **Hit Ratio:** `Hits / (Hits + Misses)` → Higher is better (aim for > 80%)

# Common Caching Patterns

## 1. Cache-Aside (Lazy Loading)
![cache aside](./images/cache-aside.excalidraw.png)

**Pros:** Only cache what's needed, cache survives restarts\
**Cons:** Initial request is slow (cache miss penalty)\
**Best for:** Read heavy workloads

## 2. Read-Through Cache
![read through](./images/read-through.excalidraw.png)

**Pros:** Simpler appication code\
**Cons:** Cache becomes critical path

## 3. Write-Through Cache
![write through](./images/write-through.excalidraw.png)

**Pros:** Cache is always consistent with DB\
**Cons:** Write latency (double write)

## 4. Write-Behind (Write-Back)
![write behind](./images/write-back.excalidraw.png)

**Pros:** Fast writes\
**Cons:** Risk of data loss, complex to implement

## 5. Refresh-Ahead
![refresh ahead](./images/refresh-ahead.excalidraw.png)

**Pros:** Eleminates cache miss penalty for hot data\
**Cons:** Wasted resource if data not accessed

# Cache Eviction Policies
When cache is full, decide what to remove:

- **LRU (Least Recently Used):** Remove least recently accessed → Most Common
- **LFU (Least Frequently Used):** Remove least frequently accessed
- **FIFO (First In First Out):** Remove oldest entry
- **Random:** Remove random entry
- **TTL-based:** Remove expired entries

# What to Cache? 

## Good Candidates
- Read-heavy data (high read/ write ration)
- Expensive computations
- External API calls
- Database query results
- Session data
- Static content

## Bad Candidates
- Highly dynamic data
- Data that changes frequently
- Large objects (>1MB typically)
- Sensitive data (requires extra security)


# Cache Sizing
Rule of thumb: Cache 15-20% of your working dataset
- Too small → High miss rate
- Too large → Wasted memory, slow eviction

# Common Cache Technologies
- **Redis:** In-memory data store, supports complex data structures, persistence
- **Memcached:** Simple key-value cache, no persistence
- **Varnish:** HTTP cache, great for web content
- **CDN:** Cloudflare, Akamai, AWS CloudFront

# Key Metrics to Monitor
- **Hit Rate:** Percentage of request served from cache
- **Miss Rate:** Percentage of requests that miss cache
- **Latency:** p50, p95, p99 response times
- **Memory Usage:** Cache size vs available memory
- **Eviction Rate:** How often items are removed

# Common Pitfalls
1. **Cache Stampede:** Multiple request fetch same data on cache miss → Use locking
2. **Stale Data:** Serving outdated information → Implement proper invalidation
3. **Memory Overflow:** Not limiting cache size → Set max memory limits
4. **Over-caching:** Caching everything → Be selective

# Quick Decision Guide
|Scenario|Recommended Approach|
|-|-|
Read-heavy, rare updates|Cache-Aside + Long TTL
Frequent updates|Short TTL or Write-Through
Static Content|CDN + Long TTL
User sessions|Redis with TTL
API rate limiting|Local cache + short TTL

# Remember
- Caching is not a silver bullet
- Always measure before optimizing
- Cache adds complexity (invalidation, consistency)
- Start simple, add complexity when needed
