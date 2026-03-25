Rate Limiting and Throttling
==

## What is Rate Limiting?

**Rate limiting** controls how many requests a client can make to a service within a given time window. Once the limit is hit, further requests are rejected until the window resets.

Think of it like a restaurant with a capacity limit, once it's full, you wait outside until someone leaves.

**Why It Matters:**

- **Protects backend services** from being overwhelmed by traffic spikes
- **Prevents abuse**, stops a single bad actor from hogging all resources
- **Ensures fairness**, one client can't starve out everyone else
- **Reduces costs**, fewer unecessary compute cycles
  
## What is Throttling?
Throttling is often confused with rate limiting, but they slightly different:
||Rate Limiting|Throttling|
|-|-|-|
|**What it does**|Hard cap, reject requests over the limit|Slow down requests, queue or delay instead of reject|
|**User experience**|Gets a 429 error|Gets a delayed response|
|**Use case**|Public APIs, abuse prevention|Internal services, background jobs|

Think of rate limiting as a bouncer who turns you away, and throttling as a traffic light that makes you wait.

In practice, the two terms are often used interchangeably. Most systems use a combination of both.

## Rate Limiting Algorithms

### 1. Fixed Window
Divide time into fixed buckets (e.g. every 60 seconds). Each bucket has a counter. Once the counter hits the limit, reject all requests until the next window.
> Window 1 (10:00 - 11:00): allows 1000 requests\
> Window 2 (11:00 - 12:00): allows 1000 requests

```go
type FixedWindowLimiter struct {
    limit      int
    count      int
    windowSize time.Duration
    windowStart time.Time
    mu         sync.Mutex
}

func NewFixedWindowLimiter(limit int, windowSize time.Duration) *FixedWindowLimiter {
    return &FixedWindowLimiter{
        limit:       limit,
        windowSize:  windowSize,
        windowStart: time.Now(),
    }
}

func (f *FixedWindowLimiter) Allow() bool {
    f.mu.Lock()
    defer f.mu.Unlock()

    // Reset window if expired
    if time.Since(f.windowStart) >= f.windowSize {
        f.count = 0
        f.windowStart = time.Now()
    }

    if f.count >= f.limit {
        return false
    }

    f.count++
    return true
}
```

**Problem: The boundary burst:**
> 10:59 → 1000 requests (within window 1 limit)\
> 11:00 → 1000 requests (within window 2 limit)\
> = 2000 requests in under 2 minutes, limit effectively doubled

**Pros:**

- Simple to implement and understand
- Low memory usage

**Cons:**

- Vulnerable to burst attack at window boundaries
- Not smooth, traffic can spike at the start of each window 


### 2. Sliding Window
Instead of fixed time buckets, track requests over a rolling time window. At any point in time, only count requests from the last N seconds.
> At 10:30 → count requests from 09:30 to 10:30\
> At 10:31 → count requests from 09:31 to 10:31

```go
type SlidingWindowLimiter struct {
    limit      int
    windowSize time.Duration
    requests   []time.Time
    mu         sync.Mutex
}

func NewSlidingWindowLimiter(limit int, windowSize time.Duration) *SlidingWindowLimiter {
    return &SlidingWindowLimiter{
        limit:      limit,
        windowSize: windowSize,
    }
}

func (s *SlidingWindowLimiter) Allow() bool {
    s.mu.Lock()
    defer s.mu.Unlock()

    now := time.Now()
    windowStart := now.Add(-s.windowSize)

    // Remove timestamps outside the window
    valid := s.requests[:0]
    for _, t := range s.requests {
        if t.After(windowStart) {
            valid = append(valid, t)
        }
    }
    s.requests = valid

    if len(s.requests) >= s.limit {
        return false
    }

    s.requests = append(s.requests, now)
    return true
}
```
**Pros:**
- Smooth, no boundary burst problem
- More accurate than fixed window

**Cons:**
- Higher memory usage (stores sall request timestamps)
- Slightly more complex

### 3. Token Bucket (Most Common)
Imagine a bucket that holds tokens. Every request costs one token. Tokens are refilled at a constant rate. If the bucket is empty, the request is rejected.

> Bucket capacity: 10 tokens\
> Refil rate: 2 tokens/second
>
> t=0s → bucket: 10 tokens\
> t=1s → 5 requests: bucket: 5 tokens + 2 refilled = 7 tokens\
> t=2 → 8 requests: bucket: 7-8 = empty → 1 request rejected\
> t=3s → 0 requests: bucket: 0 + 2 = 2 tokens

This allows short **burst** while still enforcing an average rate over time.
```go
type TokenBucket struct {
    capacity   float64
    tokens     float64
    refillRate float64 // tokens per second
    lastRefill time.Time
    mu         sync.Mutex
}

func NewTokenBucket(capacity float64, refillRate float64) *TokenBucket {
    return &TokenBucket{
        capacity:   capacity,
        tokens:     capacity, // start full
        refillRate: refillRate,
        lastRefill: time.Now(),
    }
}

func (tb *TokenBucket) Allow() bool {
    tb.mu.Lock()
    defer tb.mu.Unlock()

    now := time.Now()
    elapsed := now.Sub(tb.lastRefill).Seconds()
    tb.lastRefill = now

    // Add tokens based on elapsed time
    tb.tokens = math.Min(tb.capacity, tb.tokens+(elapsed*tb.refillRate))

    if tb.tokens < 1 {
        return false
    }

    tb.tokens--
    return true
}
```

**Pros:**
- Handles burst gracefully
- Smooth average rate enforcement
- Low memory footprint

**Cons:**
- Slightly harder to reason about exact limits
- Doesn't prevent all burst (intentional, thats the design)

### 4. Leaky Bucket
Requests go into a bucket (queue) and are processed at a constant rate, regardless of how fast they come in. If the bucket overflows, requests are dropped.

> Incoming: ▓▓▓▓▓▓▓▓▓▓  (bursty)\
> Bucket:   [▓▓▓▓▓▓▓▓▓▓] ← overflow dropped\
> Outgoing: ▓  ▓  ▓  ▓   (smooth, constant rate)

```go
type LeakyBucket struct {
    capacity  int
    queue     chan struct{}
    ticker    *time.Ticker
}

func NewLeakyBucket(capacity int, rate time.Duration) *LeakyBucket {
    lb := &LeakyBucket{
        capacity: capacity,
        queue:    make(chan struct{}, capacity),
        ticker:   time.NewTicker(rate),
    }

    // Leak requests at constant rate
    go func() {
        for range lb.ticker.C {
            select {
            case <-lb.queue:
                // process one request
            default:
                // bucket empty, nothing to do
            }
        }
    }()

    return lb
}

func (lb *LeakyBucket) Allow() bool {
    select {
    case lb.queue <- struct{}{}:
        return true
    default:
        return false // bucket full, drop request
    }
}
```

**Pros:**
- Perfectly smooth output rate
- Great for protecting downstream services from spikes

**Cons:**
- Adds latency (requests wait in queue)
- Queue can feel unresponsive during burst

## Algorithm Comparison
|Algorithm|Burst Handling|Memory|Smoothness|Best For|
|-|-|-|-|-|
|Fixed Window|Boundary burst|Low|Poor|Simple internal services|
|Sliding Window|Good|Medium|Good|General APIs|
|Token Bucket|Allows Burst|Low|Good|Public APIs, user-facing|
|Leaky Bucket|Queues burst|Medium|Excellent|Downstream protection|

## Rate Limiting in Practice

### Per-User vs Per-IP vs Per-API-Key

Different strategies suit different scenarios:

```go
// Per-IP (unauthenticated endpoints)
func getClientID(r *http.Request) string {
    if xff := r.Header.Get("X-Forwarded-For"); xff != "" {
        return strings.Split(xff, ",")[0]
    }
    return r.RemoteAddr
}

// Per-user (authenticated endpoints)
func getClientID(r *http.Request) string {
    if userID := r.Context().Value("user_id"); userID != nil {
        return fmt.Sprintf("user:%v", userID)
    }
    return r.RemoteAddr
}

// Per-API-key (developer APIs)
func getClientID(r *http.Request) string {
    if apiKey := r.Header.Get("X-API-Key"); apiKey != "" {
        return fmt.Sprintf("key:%s", apiKey)
    }
    return r.RemoteAddr
}
```

### HTTP Middleware

```go
type RateLimiterMiddleware struct {
    limiters sync.Map
    limit    int
    window   time.Duration
}

func NewRateLimiterMiddleware(limit int, window time.Duration) *RateLimiterMiddleware {
    return &RateLimiterMiddleware{
        limit:  limit,
        window: window,
    }
}

func (rl *RateLimiterMiddleware) getLimiter(clientID string) *SlidingWindowLimiter {
    limiter, _ := rl.limiters.LoadOrStore(
        clientID,
        NewSlidingWindowLimiter(rl.limit, rl.window),
    )
    return limiter.(*SlidingWindowLimiter)
}

func (rl *RateLimiterMiddleware) Handler(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        clientID := getClientID(r)
        limiter := rl.getLimiter(clientID)

        if !limiter.Allow() {
            w.Header().Set("X-RateLimit-Limit", fmt.Sprintf("%d", rl.limit))
            w.Header().Set("X-RateLimit-Remaining", "0")
            w.Header().Set("Retry-After", "60")
            http.Error(w, `{"error": "rate limit exceeded"}`, http.StatusTooManyRequests)
            return
        }

        next.ServeHTTP(w, r)
    })
}
```

### Always Return Proper Headers

Clients need to know their limit status. Always include these headers:
```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000        ← max requests allowed
X-RateLimit-Remaining: 842     ← requests left in current window
X-RateLimit-Reset: 1714000000  ← Unix timestamp when window resets

HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
Retry-After: 47                ← seconds until they can retry
```

### Distributed Rate Limiting
Per-instance in-memory limiters break down once you scale horizontally, each instance has its own counter, so a client can just hit different instances to bypass the limit.

The fix: use a **shared store** like Redis.

```go
import "github.com/go-redis/redis/v8"

type RedisRateLimiter struct {
    client *redis.Client
    limit  int
    window time.Duration
}

func NewRedisRateLimiter(client *redis.Client, limit int, window time.Duration) *RedisRateLimiter {
    return &RedisRateLimiter{
        client: client,
        limit:  limit,
        window: window,
    }
}

func (r *RedisRateLimiter) Allow(ctx context.Context, clientID string) (bool, error) {
    key := fmt.Sprintf("rate_limit:%s", clientID)

    // Use a Lua script to ensure atomicity
    script := redis.NewScript(`
        local current = redis.call("INCR", KEYS[1])
        if current == 1 then
            redis.call("EXPIRE", KEYS[1], ARGV[1])
        end
        return current
    `)

    windowSeconds := int(r.window.Seconds())
    result, err := script.Run(ctx, r.client, []string{key}, windowSeconds).Int()
    if err != nil {
        return false, err
    }

    return result <= r.limit, nil
}
```

**Why Lua script?** INCR and EXPIRE need to be atomic. Without the script, a race condition between the two calls could cause the key to never expire.

## Throttling Patterns

### Client-Side Throttling

Instead of getting rejected, the client slows itself down proactively.

```go
type Throttler struct {
    ticker  *time.Ticker
    tokens  chan struct{}
}

// Allow 10 requests per second
func NewThrottler(ratePerSec int) *Throttler {
    t := &Throttler{
        ticker: time.NewTicker(time.Second / time.Duration(ratePerSec)),
        tokens: make(chan struct{}, ratePerSec),
    }

    go func() {
        for range t.ticker.C {
            select {
            case t.tokens <- struct{}{}:
            default:
            }
        }
    }()

    return t
}

func (t *Throttler) Wait(ctx context.Context) error {
    select {
    case <-t.tokens:
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}

// Usage: throttle outgoing API calls
throttler := NewThrottler(10) // max 10 req/sec

for _, item := range items {
    if err := throttler.Wait(ctx); err != nil {
        return err
    }
    go processItem(item)
}
```

### Adaptive Throttling

Dynamically adjust the rate based on server feedback, back off when the server signals overload, speed up when it's healthy.

```go
type AdaptiveThrottler struct {
    currentRate float64
    minRate     float64
    maxRate     float64
    mu          sync.Mutex
}

func (at *AdaptiveThrottler) OnSuccess() {
    at.mu.Lock()
    defer at.mu.Unlock()
    // Gradually increase rate on success
    at.currentRate = math.Min(at.maxRate, at.currentRate*1.1)
}

func (at *AdaptiveThrottler) OnRateLimited() {
    at.mu.Lock()
    defer at.mu.Unlock()
    // Aggressively back off on 429
    at.currentRate = math.Max(at.minRate, at.currentRate*0.5)
}

func (at *AdaptiveThrottler) OnServerError() {
    at.mu.Lock()
    defer at.mu.Unlock()
    // Moderate back off on 5xx
    at.currentRate = math.Max(at.minRate, at.currentRate*0.8)
}
```

## Real-World Rate Limits
| API | Free Tier | Paid Tier | Window |
|---|---|---|---|
| Twitter/X | 15 requests | 450 requests | 15 min |
| GitHub | 60 requests | 5,000 requests | 1 hour |
| Stripe | 100 requests | No limit | 1 second |
| Google Maps | 40,000 requests | Custom | 1 month |
| OpenAI | Varies by model | Varies by tier | 1 minute |


## Best Practices

### Do:
- Use **429 Too Many Requests** not 403 or 500
- Always include `Retry-After` and `X-RateLimit-*` headers
- Set **different limits per endpoint**, a read endpoint can afford higher limits than a write endpoint
- Use **Redis** for rate limiting once you scale past a single instance
- Apply limits **per user/API key**, not just per **IP** (shared IPs like office networks can trip IP-based limits unfairly)

### Don't:
- Return a vague error with no `Retry-After`, clients won't know when to retry
- Apply the same limit to all endpoints blindly
- Forget to handle 429 on the **client side**, always respect `Retry-After`
- Use in-memory rate limiting in a multi-instance setup without a shared store

## Key Takeaways
**Rate Limiting** = hard cap. Too many requests → reject with 429.

**Throttling** = slow down. Too many requests → queue or delay.

**Algorithm to pick:**
- Just need something simple → Fixed Window
- Need accuracy without burst issues → Sliding Window
- Need to allow short bursts → Token Bucket (most common choice)
- Need perfectly smooth output → Leaky Bucket

**Distributed systems:** Always use a shared store (Redis) for rate limiting counters, in-memory per-instance limiters are useless once you sclae horizontally.

**Always tell clients what happened:** Include `X-RateLimit-*` and `Retry-After` headers so they can back off gracefully insteand of hammering your service harder.