Monolith vs Microservices
===

## What is a Monolith?
A **monolith** is a single, unified application where all components (UI, business logic, data acess) are tightly coupled and deployed as one unit.

### Characteristics
- **Single codebase:** All code in one repository
- **Single deployment:** Deploy entire application together
- **Shared database:** One database for all functionality
- **Tight coupling:** Components depend on each other directly
- **Single process:** Runs as one process or application

### Example Structure
![monolith app structure](../../images/Phase-4-Architectural-Concepts/monolith-app-structure.png)

All deployed together as one application.

## What is Microservices?
**Microservices** architecture breaks application into small, independent services that communicate over network. Each service handles one specific business capability.

### Characteristics
- **Multiple codebases:** Separate repository per service
- **Independent deployment:** Deploy services individually
- **Distributed data:** Each service has its own database
- **Loose coupling:** Services communicate via APIs
- **Multiple processes:** Each service runs independently

### Example Structure
![microservices app structure](../../images/Phase-4-Architectural-Concepts/microservices-app-structure.png)

Each service deployed independently with its own database.

## Visual Comparison

### Monolith Architecture
![monolith structure](../../images/Phase-4-Architectural-Concepts/microservices-structure.png)

### Microservices Architecture
![microservices structure](../../images/Phase-4-Architectural-Concepts/microservices-structure.png)

## Detailed Comparison

### 1. Development

#### Monolith
**Advantages:**
- Simple to develop initially
- Easy to understand (everything in one place)
- Easy to debug (single process)
- Straightforward testing
- IDE support excellent (one codebase)

**Disadvantages:**
- Large codebase hard to navigate
- Long build times (rebuild everything)
- Tight coupling (changes ripple through)
- Hard to onboard new developers
- Difficult to understand all parts

#### Microservices
**Advantages:**
- Small, focused codebase
- Easy to understand individual service
- Fast build times per service
- Team autonomy (own services)
- Technology flexibility per service

**Disadvantages:**
- Distributed system complexity
- Requires DevOps expertise
- Hard to debug across services
- Network latency overhead
- Distributed transactions complex

### 2. Deployment
#### Monolith
**Process:**
> Code Change → Build → Test → Deploy Entire App → Restart

**Advantages:**
- Simple deployment process
- Single deployment artifact
- Easy rollback (one version)
- No coordination needed

**Disadvantages:**
- Downtime during deployment
- Risk affects entire application
- Long deployment times
- Can't deploy one feature independently
- Small change = full redeployment

**Example:**
```bash
# Deploy monolith
git pull
mvn clean package
systemctl restart app
# All features deployed together
```
#### Microservcies
**Process:**
> Code Change in Service A → Build Service A → Deploy Service A\
> Other services continue running

**Advantages:**
- Independent deployments
- Zero downtime possible
- Fast deployments (only changed service)
- Isolated failure impact
- Can deploy multiple times per day

**Disadvantages:**
- Complex deployment orchestration 
- API versioning challenges
- Service compatibility issues
- Requires CI/CD pipeline
- Rollback more complex

**Example:**
```bash
# Deploy only payment service
cd payment-service
docker build -t payment:v2
kubectl set image deployment/payment payment=payment:v2
# Other services unaffected
```

### 3. Scaling

#### Monolith
**Scaling Stragey:** Vertical or horizontal scaling of entire application
> Low Load:\
> [Instance 1]
> 
> High Load:\
> [Instance 1] [Instance 2] [Instance 3]\
> (All components scaled together)

**Advantages:**
- Simple to scale 
- No service discovery needed
- Load balancer handles distribution

**Disadvantages:**
- Scale everything even if only one part needs it
- Inefficient resource usage
- Expensive (scaling unused components)
- Can't optimize for specific needs

**Example:**
> Black Friday → High order volume\
> Must scale ENTIRE app (including rarely-used admin features)\
> Cost: High (scaling everything)

#### Microservices
**Scaling Strategy:** Scale individual services based on demand
> Normal Load:\
> [Auth: 2 instances]\
> [Order: 2 instances]\
> [Payment: 2 instances]
> 
> Black Friday (Order spike):\
> [Auth: 2 instances]\
> [Order: 10 instances]  ← Scale only this\
> [Payment: 3 instances]

**Advantages:**
- Scale only what needs scaling
- Cost-efficient (targeted scaling)
- Optimize resources per service
- Handle different load patterns

**Disadvantages:**
- Requires monitoring per service
- Auto-scaling configuration complex
- Service discovery needed
- Load balancing per service

**Example:**
> // Scale only order service during Black Friday\
> kubectl scale deployment order-service --replicas=10\
> // Other services unchanged

<!-- **Advantages:**
**Disadvantages:** -->