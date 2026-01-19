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
