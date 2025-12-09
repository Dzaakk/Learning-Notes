TTL and Expiration
===

# What is TTL?
**TTL (Time To Live)** is a duration that specifies how long data should remain in cache before being considered expired and eligible for removal
```py
cache.set("key", "value", ttl=300) // Expires in 300 seconds (5 minutes)
```

# Why Use TTL?
- **Automatic Cleanup:** No manual invalidation needed
- **Predictable Behavior:** Data freshness is bounded
- **Resource Management:** Prevent cache from growing indefinitely
- **Balance:** Trade-off between freshness and performance

# TTL Mechanics

## Absolute Expiration
Data expires at specific point in time:
```py
cache.set("key", "value", expiresAt="2024-01-01T00:00:00Z")
```
**Use case:** Event-based data (tickets, promotions)

## Sliding Expiration
TTL resets on each access:
```py
cache.get("key")  // Resets TTL to original value
```
**Use case:** Session data, frequently accessed items

## Idle Timeout
Expires after period of inactivity:
```py
// Expires 30 min after last access
cache.set("key", "value", idleTimeout=1800)
```
**Use case:** User sessions, temporary data

# Choosing the Right TTL

## The Decision Framework
Ask yourself three questions:
1. How often does this data change?
2. What's the cost of serving stale data?
3. What's the cost of a cache miss?

## TTL Guidelines by Data Type
|Data Type|Recommended TTL|Reasoning
|-|-|-|
User profile|5-15 minutes|Changes occasionally, not critical
Product prices|1-5 minutes|Change frequently, moderately critical
Static assets(CSS/JS)|1 year + versioning|Never changes (use versioned URLs)
Session data|15-30 minutes|Security, natural expiration
API response (external)|5-60 seconds|Rate limiting, freshness varies
Analytics/metrics|5-10 minutes|Eventual consistency OK
Search results|1-5 minutes|Fresh results important
Stock prices|10-30 seconds|Real-time critical
Weather data|15-30 minutes|Slow-changing, widely shared
News articles|10-15 minutes|Balance freshness and load

## Formula-Based Approach
> TTL = (Acceptable Staleness) × (Update Frequency Factor)\
If data updates every 10 minutes and you can tolerate 50% staleness:\
TTL = 10 min × 0.5 = 5 minutes
> 

# TTL Patterns

## Pattern 1: Convervative TTL
```py
cache.set("critical_data", value, ttl=60)  // 1 minute
```
**When:** Data accuracy critical, changes frequently\
**Trade-off:** More cache misses, higher load

## Pattern 2: Aggresive TTL
```py
cache.set("analytics", value, ttl=3600)  // 1 hour
```
**When:** Eventual consistency acceptable, high traffic\
**Trade-off:** May serve stale data, fewer cache misses

## Pattern 3: Layered TTL
> L1 Cache (Local): TTL = 30 seconds\
L2 Cache (Redis): TTL = 5 minutes\
L3 Cache (CDN): TTL = 1 hour
>
**When:** Multi-tier caching, balance freshness at each layer

## Pattern 4: Dynamic TTL
```py
if (isPopular(data)) {
    ttl = 600  // 10 minutes for popular items
} else {
    ttl = 60   // 1 minute for unpopular items
}
```
**When:** Different data has different access patterns

## Pattern 5: Probabilistic Expiration
Avoid thundering herd by randomizing expiration:
> baseTTL = 300\
jitter = random(0, 30)\
actualTTL = baseTTL + jitter // 300-330 seconds
>
**When:** High traffic, want to spread out cache misses

# Advanced TTL Techniques

## 1. Grace Period (Stale-While-Revalidate)
Server stale data while fetching fresh data in background:
```py
if (isExpired(data) && !isVeryStale(data)) {
    asyncRefresh(key)
    return staleData  // Serve immediately
}
```
**Benefit:** No latency penalty for users\
**Implementation:**
```py
cache.set("key", value, ttl=300, gracePeriod=60)
// After 300s: stale but still served
// After 360s: actually removed
```

## 2. Hierarchical TTL
Different TTL for different levels of data:
```py
cache.set("user:123", userData, ttl=600)
cache.set("user:123:posts", posts, ttl=300)
cache.set("user:123:preferences", prefs, ttl=1800)
```

## 3. TTL Extension on Access
Reward hot data with longer TTL:
```py
accessCount = cache.getAccessCount(key)
if (accessCount > threshold) {
    cache.extendTTL(key, additionalTime)
}
```

## 4. Conditional TTL
Adjust TTL based on data characteristics:
```py
if (data.isPremium) {
    ttl = 3600  // Premium data cached longer
} else {
    ttl = 300
}
```

# TTL Anti-Patterns

## ❌ Anti-Pattern 1: Magic Numbers
```py
cache.set("key", value, 12847)  // What does this mean?
```

### Better:
```py
const PRODUCT_CACHE_TTL = 300  // 5 minutes
cache.set("key", value, PRODUCT_CACHE_TTL)
```

## ❌ Anti-Pattern 2: Same TTL for Everything
```py
// Don't do this
const DEFAULT_TTL = 3600
cache.set("prices", value, DEFAULT_TTL)
cache.set("sessions", value, DEFAULT_TTL)
cache.set("static", value, DEFAULT_TTL)
```
Different data needs different TTL

## ❌ Anti-Pattern 3: Infinite TTL
```py
cache.set("key", value)  // No TTL = never expires
```
## ❌ Anti-Pattern 4: TTL Too Short for Expensive Operations
```py
// Expensive DB query
cache.set("report", generateReport(), ttl=10)  // Too short!
```
If operation is expensive, cache longer

## ❌ Anti-Pattern 5: Ignoring Cache Stampede
```py
// Many requests expire at same time
for (item in items) {
    cache.set(item, value, ttl=3600)
}
```
Add jitter to spread expiration

# Monitoring TTL Effectiveness

## Key Metrics
1. **Cache Hit Rate:** `hits / (hits + misses)`
    - Target: >80% for most use cases
    - Low hit rate → TTL too short
2. **Staleness Duration:** Time between data change and cache expiration
    - High staleness → TTL too long
3. **Cache Miss Penalty:** Latency when cache misses
    - High penalty → Increase TTL
4. **Memory Usage:** Cache size over time
    - Growing unbounded → TTL not working

## When to Adjust TTL

### Increase TTL if:
- High cache miss rate
- Origin load too high
- Cache miss penalty is high
- Data doesn't change often

### Decrease TTL if:
- Serving too much stale data
- Data changes frequently
- Freshness is critical
- Cache memory pressure

# TTL Implementation Examples

## Redis
```redis
SET user:123 "data" EX 300       # Expires in 300 seconds
SETEX session:abc 1800 "data"    # Alternative syntax
EXPIRE existing_key 600           # Set TTL on existing key
TTL user:123                      # Check remaining TTL
```

## memcached
```py
cache.set("key", "value", time=300)  # TTL in seconds
```

## HTTP Headers
```http
Cache-Control: max-age=3600, public
Expires: Wed, 21 Oct 2024 07:28:00 GMT
```

## Application-Level (Node.js example)
```js
const cache = new Map();

function set(key, value, ttl) {
    const expiresAt = Date.now() + (ttl * 1000);
    cache.set(key, { value, expiresAt });
}

function get(key) {
    const item = cache.get(key);
    if (!item) return null;
    
    if (Date.now() > item.expiresAt) {
        cache.delete(key);
        return null;  // Expired
    }
    
    return item.value;
}
```

# TTL and System Design Interviews:
Common questions:
- "How would you cache this data?" → Consider TTL based on update frequency
- "How do you prevent stale data?" → TTL + invalidation strategy
- "How do you handle high traffic?" → Layered TTL, grace period
- "What if cache expires during high load?" → Stale-while-revalidate, jittered TTL

# Best Practices Summary
1. **Always Set TTL:** Never cache without expiration
2. **Document TTL Values:** Explain why each TTL was chosen
3. **Add Jitter:** Prevent thundering herd
4. **Monitor and Adjust:** TTL is not set-it-and-forget-it
5. **Use Constants:** No magic numbers
6. **Test Edge Cases:** Expiration during high load, clock skew
7. **Consider Grace Periods:** Balance freshness and performance
8. **Layer Appropriately:** Different TTL for different cache tiers

# Quick Reference Card
> Fast-changing data (stock prices): 10-60 seconds\
User-generated content: 1-5 minutes\
API response: 1-10 minutes\
User profiles: 5-15 minutes\
Analytics/reports: 5-30 minutes\
Static content (with versioning): 1 year\
Session data: 15-30 minutes (idle)\
> Golden Rule: TTL = How long can you tolerate stale data

# Common Mistakes to Avoid
- Setting TTL without measuring data change frequently
- Using same TTL across different data types
- Not adding jitter to prevent thundering herd
- Forgetting about cache stampede during expiration
- Not monitoring TTL effectiveness
- Hardcoding TTL values without documentation

# Final Thoughts 
TTL is a simple concept but requires thoughtful implementation. Start with conservative values, monitor effectiveness, and adjust based on real traffic patterns. The best TTL balances three factors: freshness, performance, and resource usage