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