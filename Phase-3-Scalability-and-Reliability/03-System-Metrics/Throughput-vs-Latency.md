Throughput vs Latency
===

## Core Definitions

### Throughput
**Throughput** is the number of operations or amount of data processed in a given time period. It measures capacity and volume.

**Common measurements:**
- Request per second (RPS)
- Queries per second (QPS)
- Transactions per second (TPS)
- Megabytes per second (MB/s)

**Example:** A system that can handle 10,000 user requests every second has a throughput of 10,000 RPS.

### Latency
**Latency** is the time it takes to complete a single operation or request. It measures the delay between initiating an action and receiving a response.

**Common measurements:**
- Response time (e.g., 100ms, 2 seconds)
- Round-trip time (RTT)
- Time to first byte (TTFB)

**Example:** When you click a button on a website, latency is how long it takes before you see the result appear on your screen.

## The Trade-off
Latency and throughput often have an inverse relationship, creating important design trade-offs:

### High Throughput, Higher Latency
- **Batching requests:** Processing multiple requests together increases throughput but individual requests wait longer
- **Resource sharing:** Handling many concurrent requests can slow down each individual request
- **Queue building:** High load creates queues, increasing wait time for each request

### Low Latency, Lower Throughput
- **Immediate processing:** Handling requests one at a time or in small groups reduces wait time
- **Dedicated resources:** Allocating exclusive resources to requests speeds them up but limits total capacity
- **Smaller batch sizes:** Processing fewer items at once means faster individual response times

## Real-World Examples

### Example 1: Database Queries
#### Scenario A - Optimized for Latency:
- Execute queries immediately as they arrive
- Use indexed lookups for fast retrieval
- Result: 5ms average latency, 200 QPS throughput

#### Scenario B - Optimized for Throughput:
- Batch queries together every 100ms
- Execute batched queries in parallel
- Result: 150ms average latency, 2000 QPS throughput

### Example 2: Video Streaming
#### Low Latency Mode (Live Sports):
- Minimal buffering (2-3 seconds)
- Higher bandwidth requirements
- Lower compression
- Better for interactive, real-time content

#### High Throughput Mode (On-Demand):
- Larger buffers (30+ seconds)
- Efficient compression
- Can serve more concurrent users
- Better for non-interactive content

## Key Metrics and Percentiles
When measuring latency, percentiles matter more than averages:

### Why Percentile Matter
- **p50 (median):** 50% of requests are faster than this
- **p95:** 95% of requests are faster than this - captures most user experience
- **p99:** 99% of requests are faster than this - identifies outliers
- **p99.9:** Critical for understanding worst-case scenarios

**Example:**
- Average latency: 100ms
- p50: 80ms
- p95: 200ms
- p99: 500ms

This shows that while most request are fast, 1% of users experience significant delays.

## Design Considerations

### When to Optimize for Latency
- User-facing features (UI interactions, search)
- Real-time systems (gaming, video calls, trading)
- Health checks and monitoring
- Mobile applications with poor connectivity
- APIs where user is actively waiting

### When to optimize for Throughput
- Background porcessing (ETL jobs, analytics)
- Batch operations (email sending, report generation)
- Data pipelines
- Non-interactive systems
- Resource-constrained environments

### Balanced Approach
Most systems need both:
1. **Set SLA targets:** Define acceptable latency (e.g., p99 < 500ms) and minimum throughput (e.g., 5000 RPS)
2. **Use caching:** Reduce latency for frequent requests without sacrificing throughput
3. **Horizontal scaling:** Add more servers to increase throughput while maintaining latency
4. **Connection pooling:** Reuse connections to balance both metrics
5. **Circuit breakers:** Protect latency during high throughput periods

## Measuring and Monitoring

### Tools for Latency
- Application Performance Monitoring (APM): New Relic, Datadog, Dynatrace
- Distributed tracing: Jaeger, Zipkin
- Synthetic monitoring: Pingdom, UptimeRobot

### Tools for Throughput
- Load testing: Apache JMeter, Gatling, K6
- Metrics platforms: Prometheus, Grafana
- Log aggregation: ELK Stack, Splunk

### Key Formulas
> Throughput = Number of Requests / Time Period\
> Latency = Response Time (usually measured in percentiles)\
> Capacity = Throughput x Latency (Litte's Law)

### Little's Law: `L = λW`
- L = number of requests in system
- λ = arrive rate (throughput)
- W = average time in system (latency)

This helps predic system behavior under different loads

## Common Pitfalls
1. **Optimizing average instead of percentiles:** Averages hide outliers that affect user experience
2. **Ignoring the trade-off:** Trying to maximize both simultaneously without understanding constraints
3. **Not testing at scale:** Performance characteristics change dramatically udner real load
4. **Over-optimization:** Premature optimization before identifying actual bottlenecks
5. **Forgetting about tail latency:** The slowest requests often affect overal system health

## Summary
- **Latency** measures speed of individual operations (how fast)
- **Throughput** measures volume of operations (how many)
- They often trade off against each other
- Choose optimization strategy based on use case
- Monitoring both metrics with appropriate percentiles
- Design systems with SLAs that balance both requirements
