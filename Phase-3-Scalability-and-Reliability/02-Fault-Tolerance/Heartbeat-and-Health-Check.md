Heartbeat and Health Check
===

# Overview
Heartbeat and health checks are monitoring mechanisms used to detect failures and assess system health. Heartbeats are periodic signals indicating a component is alive, while health checks actively probe components to verify they're functioning correctly. Both are critical for implementing automatic failover and maintaining high availability.

# Heartbeat Mechanisms

## What is a Heartbeat?
A heartbeat is a periodic signal sent by a component to indicate it's operational. If heartbeats stop, the system assumes the component has failed.

## Heartbeat Patterns

**Push-Based Heartbeat:** Components actively send "I'm alive" signals to a monitoring service at regular intervals. Simple to implement and low overhead, but can create network congestion with many components.

**Pull-Based Heartbeat:** Monitoring service actively queries components for their status. Centralizes monitoring logic and easier to adjust check frequency, but monitoring service becomes potential bottleneck.

**Hybrid Approach:** Components send heartbeats for normal operation, monitoring service actively schecks if heartbeats stop. Combines benefits of both approaches.

## Heartbeat Timing

**Interval Selection:** Typical intervals range from 1-30 seconds. Shorter intervals detect failures faster but increase network overhead. Longer intervals reduce overhead but delay failure detection.

**Timeout Threshold:** Usually 2-3 missed heartbeats before declaring failure. Single timeout might be network hiccup, multiple consecutive timeouts likely indicate real failure.\
Example: 5-second heartbeat with 3-miss threshold means 15 seconds to detect failure.

**Adaptive Intervals:** Adjust heartbeat frequency based on system load or component criticality. Increase frequency during high-traffic periods, decrease during quiet periods to save resources.

# Health Check Types

## Liveness Checks
Verify component is running and responsive, typically a simple ping or basic endpoint check. Used to detect crashed processes or completely unresponsive services.

**Example:** 
```json
GET /health/live
Response: 200 OK
```
if this fails, the component should be restarted or replaced

## Startup Checks
Special checks for components with long initialization times, allows more lenient timeouts during startup, prevents premature failure detection, and switches to regular checks once startup completes.

## Deep Health Checks
Comprehensive checks that verify end-to-end functionality including test database queries, sample business logic execution, and external API connectivity. More thorough but expensive to run, typically used less frequently or on-demand.

# Implementation Strategies

## HTTP-Based Health Checks
Most common approach using HTTP endpoints that return status codes (200 = healthy, 503 = unhealthy, 429 = temporarily unavailable). Include response body with detailed status information and component version/build info.

**Best Practices:**
- Keep checks lightweights (under 100ms)
- Don't perform expensive operations
- Cache dependency status briefly
- Return detailed JSON for debugging

## TCP/UDP Health Checks
Lower-level checks that verify service is listening on expected port, useful for non-HTTP services like databases or message queues. Faster than HTTP but provides less information.

## Database Health Checks
Execute simple query like `SELECT 1` to verify connectiviy, check replication lag for replicas, monitor connection pool availability, and verify write capability for primary databases.

## External Dependency Checks
Check critical external services during readiness checks, implement circuit breakers to prevent cascading failures, use cached results to avoid overwhelming dependecies, and set appropriate timeouts (don't wait indefinitely).

# Health Check Endpoints Design

## Single vs Multiple Endpoints

**Single Endpoint Approach:** One `/health` endpoint returns overall system status. Simpler for consumers but less granular control.

**Multiple Endpoint Approach:** Separate endpoints for liveness (`/health/live`), readiness (`/health/ready`), and deep checks (`/health/deep`). More flexible but requires consumers to understand differences.

## Response Format

### Minimal Response
```json
{
  "status": "healthy"
}
```

### Detailed Response
```json
{
  "status": "healthy",
  "timestamp": "2026-01-04T10:30:00Z",
  "uptime": 86400,
  "version": "1.2.3",
  "checks": {
    "database": {
      "status": "healthy",
      "responseTime": 5
    },
    "cache": {
      "status": "healthy",
      "responseTime": 2
    },
    "queue": {
      "status": "degraded",
      "responseTime": 150,
      "message": "High latency detected"
    }
  }
}
```

## Status Code
Use standard HTTP status codes consistently: 200 (OK) for fully healthy, 503 (Service Unavailable) for unhealthy, 429 (Too Many Request) for rate limiting, and 500 (Internal Server Error) for check execution failures.

# Monitoring Service Integration

## Load Balancer Health Checks
Load balancers use health checks to determine which instances receive traffic. Failed instances are automatically removed from rotation, healthy instances added back when checks pass, and traffic distribution adjusts based on instance health.

**Configuration Example:**
- Check interval: 5 seconds
- Timeout: 3 seconds
- Unhealthy threshold: 2 consecutive failures
- Healthy threshold: 3 consecutive successes

## Container Orchsetration

**Kubernetes Probes:**
- Liveness probe: Determines if container should be restarted
- Readiness probe: Determines if container should receive traffic
- Startup probe: Protects slow-starting containers

**Configuration Parameters:**
- initialDelaySeconds: Wait before first check
- periodSeconds: Check interval
- timeoutSeconds: Maximum check duration
- successThreshold: Consecutive successes needed
- failureThreshold: Consecutive failures before action

## Service Mesh Health Checks
Service meshes like Istio or Linkerd provide sophisticated health checking including automatic retry logic, circuit breaking integration, outlier detection (identifying consistently failing instances), and distributed tracing for health check calls.

# Failure Detection Patterns

## Quorum-Based Detection
Require multiple observers to agree before declaring failure, prevents false postivies from network issues, and commonly used in distributed consensus systems. Example: 3 out of 5 monitors must detect failure before taking action.

## Gossip Protocols
Nodes share health information with neighbors, information propagates through cluster organically, eventually consistent view of cluster health, and commonly used in Cassandra, Consul.

## Leader-Based Detection
Designated leader performs health checks and makes decisions, simpler coordination but leader becomes single point of failure, requires leader election protocol.

# Handling False Positives

## Graceful Degradation
Don't immediately fail hard on single check failure, allow gradual degradation with reduced capacity, and implement backoff strategies for retry logic.

## Network Partition Handling
During network partitions, both sides might think the other is down. Use quorum-based decisions, implement fencing mechanisms to prevent split-brain, and prefer consistency over availability when necessary.

## Check Correlation
Correlate multiple signals before taking action, if both heartbeat and health check fail, likely real issue, and if only one fails, might be false positive.

# Performance Considerations

**Check Overhead**\
Health Check consume resource including CPU for check execution, network bandwith for check traffic, and database connections for dependency checks. Monitor overhead and adjust frequency if needed.

**Optimization Strategies:**
- Cache dependency check results (30-60 seconds)
- Use connection pooling
- Implement check backoff during high load
- Stagger checks across instances

**Thundering Herd Problem**\
When many instances check dependency simultaneously, it can overwhelm the dependency. Randomize check timing with jitter, implement rate limiting on check endpoints, and use circuit breakers on shared dependencies.

# Best Practices
1. **Separate liveness and readiness checks** to prevent inappropriate restarts
2. **Keep checks fast** (under 100ms) to minimize overhead
3. **Test health check failures** regularly through chaos engineering
4. **Monitor health check metrics** (success rate, latency, false positive rate)
5. **Document expected behavior** for each check type and failure scenario
6. **Avoid dependency chains** in health checks to prevent cascading failures
7. **Use appropriate timeouts** that balance quick detection with stability
8. **Log health check results** for debugging and trend analysis
9. **Version health check endpoints** to enable gradual migrations
10. **Include diagnostics** in unhealthy response to aid troubleshooting

# Common Pitfalls
1. **Too aggressive timing:** Overly sensitive checks cause unnecessary failovers
2. **Checking too much:** Deep checks that verify everything slow down the system
3. **Ignoring check failures:** Not taking action on consistent failures
4. **No check at all:** Assuming components work without verification
5. **Synchronous dependency checks:** Blocking on slow external services
6. **Missing startup grace period** Killing containers before they're ready
7. **No differentiation:** Treating all failures the same way

# Monitoring Health Check System

## Key Metrics
- Health check success rate per service
- Average health check response time
- False positive rate (checks failing but service actually healthy)
- Time to detect failures (from failure occurrence to detection)
- Recovery time (from detection to service restoration)

## Alerting
Alert on sustained health check failures (not single blips), increasing false positive rates, health check systems failures, and unusual patterns in check latency.

# Real-World Example

A microservice architecture might implement health checks as follows:
- Each service exposes `/health/live` (checks if process is running) and `/health/ready` (checks database, cache, message queue connectivity). Load balancers check `/health/ready` every 5 seconds with 3-second timeout
- Removing instances after 2 consecutive failures and adding back after 3 consecutive successes

Kubernetes uses liveness probe on `/health/live` every 10 seconds to restart crashed containers, readiness probe on `/health/ready` every 5 seconds to control traffic routing, and startup probe with 30-second timeout for slow-starting services. 

A separate monitoring service subsrcibes to heartbeat messages sent every 10 seconds via message queue, raises alert if no heartbeat received for 30 seconds, and triggers automated failover if no heartbeat for 60 seconds. This multi-layered approach ensures failures are detected quickly (5-10 seconds) while avoiding false positives through multiple validation points.