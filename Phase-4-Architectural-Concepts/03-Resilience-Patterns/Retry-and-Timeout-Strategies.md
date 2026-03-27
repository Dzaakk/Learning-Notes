Retry and Timeout Strategies
==

## Why These Two Always Come Together
Retries and timeouts are two sides of the same coin. Without timeouts, retries hang forever. Without retries, a single transient failure kills the request. Together, they make your system resilient against the most common failure mode in distributed systems **transient errors**.

Transient errors are temporary failures that go away on their own: 
- a brief network blip 
- a service monentarily overloaded
- a database connection hiccup

These aren't bugs, they're just the reality of running distributed systems.

## Timeouts

### What is a Timeout?
A timeout is the maximum time you're willing to wait for a response before giving up. Without one, a single slow dependency can hold your goroutine (or thread) hsotage indefinitely, stalling the entire request chain.

### Analogy
Ordering food at a restaurant. If the kitchen takes 2 hours for your order, you don't just sit there, you give up, leave, and try somewhere else. A timeout is that decision point.

### Types of Timeouts
There are several layers where timeouts should be set, and all of them matter:
```
Client → [Connection Timeout] → Server
         [Request Timeout   ]
         [Response Timeout  ]
         [Overall Deadline  ]
```

- **Connection Timeout:** Max time to establish a TCP connection. Fails fast if the server is unreachable
- **Request Timeout:** MAx time to send the full request (relevant for large payloads)
- **Response Timeout:** Max time to receive the full response after the request is sent.
- **Overall Deadline:** End to end time budget for the entire operation, including retries. This is the most important one.

### Implementation

```go
// Basic HTTP client with timeout
client := &http.Client{
    Timeout: 5 * time.Second, // overall timeout (connect + send + receive)
}

resp, err := client.Get("http://payment-service/charge")
if err != nil {
    if errors.Is(err, context.DeadlineExceeded) {
        // handle timeout specifically
    }
    return err
}
```
```go
// Granular control with context deadline
func callService(ctx context.Context) error {
    // Set an overall deadline for this operation
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    transport := &http.Transport{
        DialContext: (&net.Dialer{
            Timeout: 2 * time.Second, // connection timeout
        }).DialContext,
        ResponseHeaderTimeout: 3 * time.Second, // wait for first byte
    }

    client := &http.Client{Transport: transport}
    req, err := http.NewRequestWithContext(ctx, "GET", "http://payment-service/charge", nil)
    if err != nil {
        return err
    }

    resp, err := client.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    return nil
}
```

### Timeout Propagation
In a chain of services, each service should **pass the remaining deadline downstream**, not set a fresh one. Otherwise, you can exceed the client's deadline even if every individual service call is "within limits."

```
Client sets: 5s overall deadline
  → Service A uses 2s → passes remaining 3s to Service B
  → Service B uses 1s → passes remaining 2s to Service C
  ✅ Total: 3s, within the 5s budget

vs.

Client sets: 5s overall deadline
  → Service A sets fresh 5s → Service B sets fresh 5s → ...
  ❌ Total can exceed 5s easily
```

```go
// Pass context (with deadline) downstream — don't create a new one
func handleRequest(ctx context.Context) error {
    // ctx already has the deadline from the upstream caller
    result, err := callServiceA(ctx) // passes same ctx
    if err != nil {
        return err
    }
    return callServiceB(ctx, result) // passes same ctx
}
```

### How to Pick a Timeout Value
Set timeouts based on your **p99 latency**, the 99th percentile response time under normal load. A timeout set at p50 will reject too many legitimate requests. A timeout set at 10x p99 might as well not exist.
```
p50 latency: 50ms
p99 latency: 200ms

Good timeout: 500ms - 1s (2.5x - 5x p99)
Too tight: 100ms (rejects normal slow requests)
Too loose: 30s (hangs for too long on failures)
```

## Retries

### What is a Retry?
A retry is automatically re-sending a failed request, hoping the transient error has resolved. Simple idea, but done wrong, it makes things significantly worse.

### What's Safe to Retry?
Not all failures should be retried. The key question is: **is the operation idempotent?**

|Status|Retry?|Reason|
|-|-|-|
|Network error / timeout|✅Yes|Transient, likely safe|
|429 Too Many Requests|✅Yes (after delay)|Rate limited, not a failure|
|500 Internal Server Error|✅Yes (carefully)|Might be transient|
|503 Service Unavialable|✅Yes|Service temporarily down|
|400 Bad Request|❌No|Your request is wrong, retrying won't help|
|401 Unauthorized|❌No|Auth issue, not transient|
|404 Not Found|❌No|Resource doesn't exist|

**Never retry non-idempotent operations blindly**. A `POST /orders` that creates an order, if it timed out, you don't know if the order was created or not. Retrying could create a duplicate. Use idempotency keys to handle this.

### Retry Strategies

### 1. Immediate Retry
Retry right away. Only makes sense for very transient errors (e.g. a dropped packet). More than 1-2 immediate retires is almost always wrong.
```go
func callWithImmediateRetry(fn func() error, maxRetries int) error {
    for i := 0; i < maxRetries; i++ {
        if err := fn(); err == nil {
            return nil
        }
    }
    return fmt.Errorf("failed after %d retries", maxRetries)
}
```

### 2. Fixed Delay
Wait a fixed amount of time between retries. Better than immediate, but can still cause thundering herd if many clients retry at the same time.
```go
func callWithFixedDelay(fn func() error, maxRetries int, delay time.Duration) error {
    for i := 0; i < maxRetries; i++ {
        if err := fn(); err == nil {
            return nil
        }
        if i < maxRetries-1 {
            time.Sleep(delay)
        }
    }
    return fmt.Errorf("failed after %d retries", maxRetries)
}
```

### 3. Exponential Backoff (Most Common)
Double the wait time on each retry. Gives the service progressively more time to recover, and naturally spreads out retry traffic.
```
Retry 1: wait 1s
Retry 2: wait 2s
Retry 3: wait 4s
Retry 4: wait 8s
```

```go
func callWithExponentialBackoff(fn func() error, maxRetries int) error {
    baseDelay := 1 * time.Second

    for i := 0; i < maxRetries; i++ {
        if err := fn(); err == nil {
            return nil
        }

        if i < maxRetries-1 {
            delay := baseDelay * time.Duration(math.Pow(2, float64(i)))
            time.Sleep(delay)
        }
    }
    return fmt.Errorf("failed after %d retries", maxRetries)
}
```

### 4. Exponential Backoff with Jitter (Best Practice)
Pure exponential backoff has a problem: if 100 cleints all fail at the same time, they all retry at the same intervals, creating synchronized waves of traffic called a **thundering herd**.

Jitter adds randomness to break that syncrhonization.
```
Without jitter: all 100 clients retry at exactly t=1s, t=2s, t=4s
With jitter: clients retry at random times between 0 and 2^n seconds
```

```go
func callWithJitter(ctx context.Context, fn func() error, maxRetries int) error {
    baseDelay := 500 * time.Millisecond
    maxDelay := 30 * time.Second

    for i := 0; i < maxRetries; i++ {
        if err := fn(); err == nil {
            return nil
        }

        if i == maxRetries-1 {
            break
        }

        // Exponential backoff
        exp := math.Pow(2, float64(i))
        delay := time.Duration(float64(baseDelay) * exp)

        // Cap the delay
        if delay > maxDelay {
            delay = maxDelay
        }

        // Add jitter: random value between 0 and delay
        jitter := time.Duration(rand.Float64() * float64(delay))

        select {
        case <-time.After(jitter):
        case <-ctx.Done():
            return ctx.Err() // respect context cancellation
        }
    }

    return fmt.Errorf("failed after %d retries", maxRetries)
}
```

### Retry + Circuit Breaker
Retries and circuit breakers must work together. Without a circuit breaker, retries on a down service will pile up, making recovery even harder for the struggling service.
```
Service down
→ Request fails
→ Retry (fail) → Retry (fail) → Retry (fail)
→ 100 clients × 3 retries = 300 requests hammering a dead service
→ Service can't recover

With circuit breaker:
→ After N failures, circuit opens
→ Retries are rejected locally (no network call made)
→ Service gets breathing room to recover
```

```go
func callWithRetryAndCircuitBreaker(
    ctx context.Context,
    cb *gobreaker.CircuitBreaker,
    fn func() error,
    maxRetries int,
) error {
    baseDelay := 500 * time.Millisecond

    for i := 0; i < maxRetries; i++ {
        _, err := cb.Execute(func() (interface{}, error) {
            return nil, fn()
        })

        if err == nil {
            return nil
        }

        // Don't retry if circuit is open — service is clearly down
        if err == gobreaker.ErrOpenState {
            return fmt.Errorf("circuit open, aborting retries: %w", err)
        }

        if i < maxRetries-1 {
            delay := baseDelay * time.Duration(math.Pow(2, float64(i)))
            jitter := time.Duration(rand.Float64() * float64(delay))

            select {
            case <-time.After(jitter):
            case <-ctx.Done():
                return ctx.Err()
            }
        }
    }

    return fmt.Errorf("failed after %d retries", maxRetries)
}
```

## Idempotency Keys
For non-idempotent operations (like creating an order or chargin a card), use an **idempotency key**, a unique ID sent with the request so the server can detect and deduplicate retries.

```go
func createOrder(ctx context.Context, order *Order) (*Order, error) {
    // Generate a stable key for this order attempt
    idempotencyKey := uuid.New().String()

    req, _ := http.NewRequestWithContext(ctx, "POST", "http://order-service/orders", body)
    req.Header.Set("Idempotency-Key", idempotencyKey)

    return callWithJitter(ctx, func() error {
        resp, err := client.Do(req)
        // same idempotency key is sent on every retry
        // server returns the same result if already processed
        return err
    }, 3)
}
```
Server-side, store the result keyed by the idempotency key. If the same key comes in again, return the stored result immediately without reprocessing.

## Key Takeaways

### Timeouts:
- Always set timeouts, no exceptions
- Set at the connection, response, and overall deadline level
- Base timeout values on p99 latency, not guesswork
- Propagate context deadlines downstream rather than creating fresh timeouts

### Retries:
- Only retry idempotent operations, or use idempotency keys for non-idempotent ones
- Never retry 4xx errors (except 429)
- Always use exponential backoff with jitter, plain fixed delay causes thundering herd
- Always pair retries with a circuit breaker, retries without one can collapse a recovering service
- Set a max retry budget, 3 retries is usally enough

**The golden rule:** Retries buy you resillience against transient failures. Timeouts make sure retries don't run forverer. Circuit breakers make sure retries don't kill the service you're trying to reach.