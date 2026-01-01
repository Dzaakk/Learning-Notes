Scaling Patterns
===

# Overview
Scaling patterns are proven architectural approaches to handle growth. These patterns solve specific bottlenecks as your system scales.

# 1. Load Balancing
Distribute incoming traffic across multiple servers

## How It Works
![load balancing](./images/load-balancing.png)

## Algorithms

### Round Robin
- Distributes requests sequentially
- Simple, even distribution
- Use when: have equal capacity

### Least Connections
- Routes to server with fewest active connections
- Use when: Requests have variable duration

### IP Hash
- Same client → same server (based on IP)
- Use when: Need session affinity

### Weighted Round Robin
- Distribute based on server capacity
- Use when: Servers have different specs

## Key Concepts
- **Health checks:** Remove unhealthy servers from pool
- **Layer 4 (TCP):** Fast, basic routing
- **Layer 7 (HTTP):** Smart routing based on content (path, headers)

## Example Use Case
E-commerce site with 3 web servers:
- Load balancer receives user request
- Routes to server with least connections
- If server fails health check →  route to healthy servers
- Use never notice the failure