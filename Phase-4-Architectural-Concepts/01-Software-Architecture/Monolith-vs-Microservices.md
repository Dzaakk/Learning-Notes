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

### 4. Technology Stack

#### Monolith
**Constraint:** Single technology for entire application
> Application Stack:
> - Language: Java
> - Framework: Spring Boot
> - Database: PostgreSQL
> - Cache: Ehcache (in-process)

**Advantages:**
- Consistent technology
- Team expertise focused
- Simpler hiring
- Shared library easy

**Disadvantages:**
- Stuck with initial choices
- Hard to adopt new technologies
- Not optimized for specific tasks
- Legacy technology burden

#### Microservices
**Flexibility:** Different technology per service
> Auth Service:     Node.js + MongoDB\
> Order Service:    Java + PostgreSQL\
> Payment Service:  Go + MySQL\
> Search Service:   Python + Elasticsearch

**Advantages:** 
- Right tool for right job
- Easy to adopt new technologies
- Optimize per service needs
- Experiment safely

**Disadvantages:**
- Multiple technologies to maintain
- Requires diverse expertise
- Harder to hire (need polyglots)
- More operational overhead

### 5. Data Management
#### Monolith
**Pattern:** Single shared database
![mono data management](../../images/Phase-4-Architectural-Concepts/mono-data-management.png)

**Advantages:** 
- ACID transactions across tables
- Joins are easy and fast
- Data consistency guaranteed
- Simple backup/recovery
- Foreign keys enforce integrity

**Disadvantages:**
- Database becomes bottleneck
- Tight coupling via database
- Hard to scale writes
- Schema changes affect everything
- Can't use different database types

**Example:**
```sql
-- Easy transaction across tables
BEGIN TRANSACTION;
  INSERT INTO orders (...);
  UPDATE inventory SET stock = stock - 1;
  INSERT INTO payments (...);
COMMIT;
```

#### Microservices
**Pattern:** Database per service

![micro data management](../../images/Phase-4-Architectural-Concepts/microservice-data-management.png)
**Advantages:** 
- Service independence
- Different database types possible
- Easier to scale per service
- Schema changes isolated
- No single database bottleneck

**Disadvantages:**
- No ACID across services
- Distributed transactions needed
- Data duplication required
- Eventual consistency
- Complex queries across services

**Example:**
```js
// Distributed transaction (Saga pattern)
try {
  // 1. Create order
  await orderService.createOrder(orderData);
  
  // 2. Process payment
  await paymentService.charge(paymentData);
  
  // 3. Update inventory
  await inventoryService.decreaseStock(items);
} catch (error) {
  // Compensating transactions
  await orderService.cancelOrder(orderId);
  await paymentService.refund(paymentId);
}
```

### 6. Team Structure

#### Monolith
**Organization:** Typically by technical layer
> Frontend Team  →  UI Layer\
> Backend Team   →  Business Logic\
> Database Team  →  Data Layer

**Advantages:** 
- Clear technical expertise
- Easy to standardize
- Simple reporting structure

**Disadvantages:**
- Cross-team coordination needed for features
- Slower feature delivery
- Handoffs between teams
- Unclear ownership

#### Microservices
**Organization:** By business capability (cross-functional teams)
> Payments Team  →  Payment Service (Full Stack)\
> Orders Team    →  Order Service (Full Stack)\
> Auth Team      →  Auth Service (Full Stack)

**Advantages:** 
- Team autonomy
- Faster feature delivery
- Clear ownership
- End-to-end responsibility

**Disadvantages:**
- Requries full-stack developers
- Duplication across teams
- Coordination for cross-service features
- Potential inconsistency

### 7. Failure Handling
#### Monolith
**Failure Impact:** All or nothing
> Any components fails → Entire application down

**Advantages:** 
- Simpler failure modes
- Easier to debug
- Clear state (up or down)

**Disadvantages:**
- Single point of failure
- One bug can crash everything
- No partial degradation
- Higher blast radius

**Example:**
> Payment module bug → Entire app crashses\
> Users can't access ANY feauter 

#### Microservices
**Failure Impact:** Isolated failures
> Payment service fails → Other services continue

**Advantages:** 
- Fault isolation 
- Graceful degradation possible
- Partial availability
- Smaller blast radius

**Disadvantages:**
- Complex failure scenarios
- Cascade failures possible
- Harder to debug
- Requires circuit breakers

**Example**
> Payment service down → Disable checkout\
> But users can still:
> - Browse products
> - Add to cart
> - View account