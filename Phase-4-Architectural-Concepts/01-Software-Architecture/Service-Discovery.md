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

## Service Discovery Patterns

### 1. Client-Side Discovery

**How It Works**\
Client is responsible for determining network location of available service instances.
![client side discovery](../../images/Phase-4-Architectural-Concepts/client-side-discovery.png)

**Implementation Example**
```go
// Client-side discovery with Consul
package main

import (
    "fmt"
    "math/rand"
    "net/http"
    "time"
    
    consulapi "github.com/hashicorp/consul/api"
)

type ServiceDiscovery struct {
    client *consulapi.client
}

func NewServiceDiscovery(consulAddr string) (*ServiceDiscovery, error) {
    config := consulapi.DefaultConfig()
    config.Address = consulAddr
    
    client, err := consulapi.NewClient(config)
    if err != nil {
        return nil, err
    }
    
    return &ServiceDiscovery{client: client}, nil
}

// Discover service instances
func (sd *ServiceDiscovery) GetService(serviceName string) (string, error) {
    // 1. Query Consul for healthy instances
    services, _, err := sd.client.Health().Service(serviceName, "", true, nil)
    if err != nil {
        return "", err
    }
    
    if len(services) == 0 {
        return "", fmt.Errorf("no healthy instances of %s", serviceName)
    }
    
    // 2. Load balance (round-robin/random)
    instance := services[rand.Intn(len(services))]
    
    // 3. Return service URL
    url := fmt.Sprintf("http://%s:%d", 
        instance.Service.Address, 
        instance.Service.Port)
    
    return url, nil
}

func main() {
    // Initialize discovery client
    discovery, err := NewServiceDiscovery("localhost:8500")
    if err != nil {
        panic(err)
    }
    
    // Get order service URL
    orderServiceURL, err := discovery.GetService("order-service")
    if err != nil {
        panic(err)
    }
    
    // Make request
    resp, err := http.Get(orderServiceURL + "/orders/123")
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    fmt.Println("Order fetched successfully")
}
```

**Advantages:**
- Simple architecture (no intermediary)
- Client controls load balancing
- Direct connection (lower latency)
- Client can implement custom logic

**Disdvantages:**
- Client complexity
- Tight coupling to registry
- Must implement in every client
- Language-specific client libraries

**When to Use:**
- Homogeneous environment (same language)
- Need custom load balancing logic
- Want direct client to service communication
- Have control over all cleints

### 2. Server-Side Discovery

**How It Works**\
Client makes request to load balancer, which queries registry and routes to instance.

![server side discovery](../../images/Phase-4-Architectural-Concepts/server-side-discovery.png)

**Implementation Example**
```go
// Server-side discovery with Kubernetes
// Client doesn't know about service discovery at all

// main.go - Simple HTTP client
package main

import (
    "fmt"
    "io"
    "net/http"
)

func main() {
    // Kubernetes Service DNS name
    // Kubernetes automatically routes to healthy pods
    resp, err := http.Get("http://order-service/orders/123")
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    fmt.Println(string(body))
}
```
```yaml
# Kubernetes Service YAML
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8001
  type: ClusterIP
```

**Advantages:**
- Simple client (just hostname)
- Language-agnostic
- Centralized load balancing
- No client library needed
- Load balancer can add features (SSL, caching)

**Disdvantages:**
- Additional network hop (load balancer)
- Load balancer is single point of failure
- Load balancer can become bottleneck
- More infrastructure to manage

**When to Use:**
- Polygot environment (multiple languages)
- Want simple clients
- Already using load balancer
- Platform provides it (Kubernetes, AWS ECS)