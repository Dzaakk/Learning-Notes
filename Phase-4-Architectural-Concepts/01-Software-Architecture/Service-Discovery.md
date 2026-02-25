Service Discovery
===

## What is Service Discovery?
**Service Discovery** is the automatic detection of services in a network. It allows services to find and communicate with each other without hardcoding network locations (IP addresses and ports).

### The Problem Without Service Discovery
**Traditional Approach (Hardcoded)**
```go
// Bad: Hardcoded service locations
const orderServiceURL = "http://192.168.1.10:8001"
const paymentServiceURL = "http://192.168.1.15:8002"

// What if server crashes and moves to 192.168.1.20?
// What if we scale to 5 order service instances?
// Need to update code and redeploy!
```

**Problems:**
- Service can move (IP changes)
- Services can scale (multiple instances)
- Services can fail (needed failover)
- Manual updates required (error-prone)
- Doesn't work with auto-scaling

**With Service Discovery**
```go
// Good: Dynamic service discovery
orderService := discovery.GetService("order-service")
paymentService := discovery.GetService("payment-service")

// Returns: Available healthy instance automatically
// Handles: Multiple instances, failures, scaling
```

**Benefits:**
- Dynamic instances discovery
- Automatic failover
- Load balancing
- Health checking
- Works with auto-scaling

## Core Concepts

### Service Registry
Central database that stores information about available services.

**Stored information:**
- Service name (`order-service`)
- Network location (`192.168.1.10:8001`)
- Health status (`healthy`, `unhealthy`)
- Metadata (version, region, tags)

**Example registry data:**
```json
{
  "order-service": [
    {
      "id": "order-1",
      "address": "192.168.1.10",
      "port": 8001,
      "status": "healthy",
      "version": "2.1.0",
      "region": "us-east-1"
    },
    {
      "id": "order-2",
      "address": "192.168.1.11",
      "port": 8001,
      "status": "healthy",
      "version": "2.1.0",
      "region": "us-east-1"
    }
  ],
  "payment-service": [
    {
      "id": "payment-1",
      "address": "192.168.1.20",
      "port": 8002,
      "status": "healthy"
    }
  ]
}
```

### Service Registration
Process where service register themselves with the registry.

**Two patterns:**
1. **Self-Registration:** Service registers itself
2. **Third-Party Registration:** External system registers services

### Service Discovery
Process where services lookup other services in the registry.

**Two patterns:**
1. **Client-Side Discovery:** Client queries registry
2. **Server-Side Discovery:** Load balancer queries registry
