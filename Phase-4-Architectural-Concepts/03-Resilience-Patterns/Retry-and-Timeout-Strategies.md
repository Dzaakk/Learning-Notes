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