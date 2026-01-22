Load Balancer Deep Dive
===

## What is a Load Balancer?
A **load balancer** is a system component that distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed. It acts as a traffic cop, directing requests to available servers based on various algorithms.

### Core Functions
1. **Traffic distribution:** Spread requests across multiple backend servers
2. **Health checking:** Monitor server health and remove unhealthy servers from rotation
3. **Session persistence:** Route requests from the same client to the same server when needed
4. **SSL termination:** Handle encryption/decryption to reduce load on backend servers
5. **Request routing:** Direct traffic based on URL paths, headers, or other criteria

## Types of Load Balancers

### Layer 4 (Transport Layer) Load Balancers
Operate at the TCP/UDP level, making decisions based on network information:
- Source/destination IP addresses
- Source/destination ports
- Protocol type

**Characteristics:**
- Very fast (simple packet forwarding)
- No knowledge of application content
- Cannot make routing decisions based on HTTP headers or URLs
- Lower resource consumption

**Use cases:** High-performance scenarios, non-HTTP protocols, when content inspection is not needed

**Example:** AWS Network Load Balancer (NLB)

### Layer 7 (Application Layer) Load Balancers
Operate at the HTTP/HTTPS level, making intelligent routing decisions:
- HTTP headers
- Cookies
- URL paths
- Request content

**Characteristics:**
- Content-aware routing
- Can perform SSL termination
- URL rewriting and redirects'
- More resource-intensive
- Enable advanced features (A/B testing, canary deployments)

**Use cases:** Web applications, microservices, content-based routing

**Example:** AWS Application Load Balancer (ALB), NGINX, HAProxy

### Global Load Balancers (GSLB)
Distribute traffic across multiple geographic regions:
- DNS-based routing
- Geographic proximity
- Disaster recovery
- Multi-region failover

**Example:** AWS Route 53, Cloudflare Load Balancing, Google Cloud Load Balancing

## Load Balancing Algorithms

### 1. Round Robin
Distributes requests sequentially to each server in rotation.

#### How it works:
> Reqeust 1 → Server A\
> Reqeust 2 → Server B\
> Reqeust 3 → Server C\
> Reqeust 4 → Server A (cycle repeats)

#### Pros:
- Simple and fair distribution
- Easy to implement
- Good when servers have equal capacity

#### Cons:
- Doesn't account for server load or capacity
- Doesn't consider request complexity
- Poor for sessions requiring state

**Best for:** Stateless applications with uniform server specs

### 2. Weighted Round Robin
Similiar to round robin but assigns weights based on server capacity.

#### How it works:
> Server A (weight: 3)\
> Server B (weight: 1)
>
> Request distribution:\
> A, A, A, B, A, A, A, B...

#### Pros:
- Accounts to different server capacities
- Still simple to implement

#### Cons:
- Static weights don't adapt to real-time load

**Best for:** Mixed hardware environments with known capacity differences

### 3. Least Connections
Routes requests to the server with the fewest active connections.

#### How it works:
> Server A: 10 Active connections\
> Server B: 5 active connections\
> Server C: 8 active connections
>
> Next request → Server B

#### Pros:
- Dynamic allocation based on actual load
- Better for long-lived connections
- Adapts to varying request durations

#### Cons:
- More complex tracking required
- Doesn't account for connection weight/complexity

**Best for:** Applications with variable request processing times

### 4. Weighted Least Connections
Combines least connections with server capacity weights.\
**Formula:** `Connection ration = Active Connections / Weight`\
Routes to server with lowest ratio.

**Best for:** Mixed capacity servers with variable processing times

### 5. IP Hash
Uses client IP address to determine which server receives the request.

#### How it works:
> hash (client_IP) % number_of_servers = server_index
#### Pros:
- Session persistence without cookies
- Consistent routing for same client
- No session state required on load balancer

#### Cons:
- Uneven distribution if few clients
- Difficult to rebalance when servers change
- Doesn't account for server load

**Best for:** Applications requiring session persistence, caching scenarios


### 6. Least Response Time 
Routes to server with lowest response time and fewest active connections.

#### Pros:
- Optimizes for actual performance
- Consider both load and server speed

#### Cons:
- Requires active monitoring
- More complex calculation

**Best for:** Performance-critical applications

### 7. Random
Randomly selects a server for each request.

#### Pros:
- Simple implementation
- No state tracking needed
- Eventually distributes evenly at scale

#### Cons:
- Can be uneven in short term
- No intelligence about load

**Best for:** Very simple systems, testing

## Health Checks
Load balancers continuously monitor backend server health to avoid routing traffic to failed servers.

### Types of Health Checks

#### 1. Passive Health Checks
Monitor actual traffic to detect failures:
- Track response codes (500, 502, 503, 504 errors)
- Monitor timeouts
- Count consecutive failures

**Example configuration:**
> unhealthy_threshold: 3 consecutive failures\
> timeout: 5 seconds

#### 2. Active Health Checks
Proactively send test requests to servers:
- HTTP GET request to health endpoint
- TCP connection tests
- Custom protocol checks

**Example configuration:**
> interval: 10 seconds\
> timeout: 3 seconds\
> healthy_treshold: 2 consecutive successes\
> unhealthy_treshold: 3 consecutive successes\
> path: /health\
> expected_status:200

### Health Check Best Practices
1. **Use dedicated health endpoints:** `/health` or `/ping` that check critical dependencies
2. **Keep checks lightweight:** Don't overload servers with heavy health checks
3. **Appropriate intervals:** Balance between quick failure detection and overhead
4. **Check dependencies:** Verify database, cache, and external service connectivity
5. **Graceful degradation:** Return partial health status when non-critical components fail

## Session Persistence (Sticky Sessions)

### What is Session Persistence?
Ensures requests from the same client always route to the same backend server.

### Implementation Methods

#### 1. Cookie-Based Persistence
Load balancer inserts a cookie to track which server handled the first request.

**Example:**
> First request → Server B\
> Response includes: Set-Cookie:\
> LB_SERVER=B\
> Subsequent requests with cookie → Server B

**Pros:**
- Works across different client IPs (mobile networks)
- Survives client network changes

**Cons:**
- Requires cookie support
- Adds overhead

#### 2. IP-Based Persistence
Uses source IP address (IP Hash algorithm).

**Pros:**
- No cookies needed
- Transprent to application

**Cons:**
- Breaks with NAT/proxies
- Problem with mobile clients

#### 3. Application-Controlled
Application sets custom header or parameter.

**Pros:**
- Full aplication control
- Can migrate sessions

**Cons:**
- Requires application changes

### When to Use Session Persistence

**Use when:**
- Application stores session state locally on servers
- WebSocket connections
- Shopping carts, user session without centralized storage

**Avoid when:**
- Sessions are stored in shared cache (Redis)
- Database-backed sessions
- Stateless applications

**Better alternatives:**
- Use centralized session storage (Redis, Memcached)
- Use database-backed sessions
- Design stateless applications

## SSL/TLS Termination

### What is SSL Termination?
Load balancer handles SSL/TLS encryption/decryption instead of backend servers.

### SSL Termination at Load Balancer
#### Advantages:
- Reduces CPU load on backend servers
- Centralized certificate management
- Can inspect/modify encrypted traffic
- Simplifies backend server configuration

#### Disadvantages:
- Traffic between LB and backend is unencrypted (security concern)
- Load balancer becomes bottleneck for encryption

### SSL Pass-Through
Load balancer forwards encrypted traffic directly to backend servers.

#### Advantages:
- End-to-end encryption
- No decryption overhead on load balancer

#### Disadvantages:
- Cannot perform content-based routing
- Each backend needs certificates
- Higher CPU usage on backends

### SSL Re-Encryption (SSL Bridging)
Load balancer decrypts, inspects, then re-encrypts traffic to backends.

#### Best of both worlds but:
- Highest overhead
- Most secure
- Most flexible

## Advanced Load Balancer Features

### 1. Connection Draining
Gracefully removes servers from rotation:
- Stop sending new connections
- Allow existing connections to complete
- Set timeout for stragglers

**Use cases:** Deployments, maintenance, scaling down

### 2. Rate Limiting
Limit request per client/IP:\
> max_request: 100 per minute\
> burst: 20

**Protect against:**
- DDoS attacks
- Abusive clients
- Accidental traffic spikes

### 3. Request Routing
Route based on content\
> /api/* → API servers\
> /static/* → CDN/static servers\
> /admin/* → Admin servers

### 4. Auto Scaling Integration
Automatically add/remove servers based on metrics:
- CPU usage
- Request count
- Response time
- Custom metrics

### 5. Observability
- Access logs
- Metrics (request rate, error rate, latency)
- Distributed tracing headers
- Health check status

## Popular Load Balancer Technologies

### Hardware Load Balancers
- **F5 BIG-IP:** Enterprise-grade, expensive, high performance
- **Citrix ADC:** Application delivery controller with advanced features

**Pros:** High performance, dedicated hardware\
**Cons:** Expensive, less flexible, vendor lock-in

### Software Load Balancers
- **NGINX:** High-performance, widely used, both L4 and L7
- **HAProxy:** Extremely fast, TCP and HTTP, highly configurable
- **Envoy:** Modern, cloud-native, service mesh support
- **Traefik:** Kubernetes-native, automatic service discovery

**Pros:** Flexible, cost-effective, cloud-friendly\
**Cons:** Requires more management, runs on shared hardware

### Cloud Load Balancers
- **AWS ELB/ALB/NLB:** Managed service, highly available
- **Google CLoud Load Balancing:** Global, anycast, integrated
- **Azure Load Balancer:** Layer 4 and 7 options

**Pros:** Fully managed, auto-scaling, high availability\
**Cons:** Vendor lock-in, can be expensive at scale

## Common Architectures

### Single Load Balancer
> Internet → Load Balancer → [Server 1, Server 2, Server 3]

**Issues:** Single point of failure

### High Availability Load Balancers
> Internet → [LB 1 (Active), LB 2 (Standby)] → Backend Servers

**Uses:** Keepalived, VRRP protocol for failover

### Multi-Tier Load Balancing
> Internet → Externeal LB → [Web Tier LBs] → [App Tier LBs] → [DB Read Replicas]

### Global Load Balancing
> User → DNS (Route 53) → Regional LBs → Backend Servers

## Best Practices
1. **Always use health checks:** Detec and remove unhealthy servers automatically
2. **Enable connection draining:** Graceful shutdowns during deployments
3. **Monitor load balancer metrics:** It can become a bottleneck
4. **Use multiple availability zones:** Distribute across failure domains
5. **Implement retry logic:** Handle transient failures
6. **Set appropriate timeouts:** Balance between patience and responsiveness
7. **Use centralized session storage:** Avoid sticky session requirements
8. **Regular capacity planning:** Ensure load balancer can handle peak traffic
9. **Enable logging and monitoring:** Track traffic patterns and errors
10. **Consider Layer 7 for flexibility:** Unless performance demands Layer 4

## Summary
Load balancers are critical for:
- High availability and fault tolerance
- Horizontal scaling
- Traffic management
- Performance optimization

Choose the right type (Layer 4 vs Layer 7), algorithm, and features based on your specific requirements for availability, performance, and cost.
