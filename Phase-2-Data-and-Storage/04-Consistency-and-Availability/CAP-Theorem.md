CAP Theorem
===

# What is CAP Theorem? 
**Definition:** In a distributed system, you can only guarantee 2 out of 3 properties simultaneously

- **C - Consistency:** All nodes see the same data at the same time
- **A - Availability:** Every request receivces a response (success or failure)
- **P - Partition Tolerance:** System continues operating despite network failures

**Key Insight:** Network partitions WILL happen in distributed systems, so you must choose between Consistency (CP) or Availability (AP)

# The Three Properties Explained

## Consistency (C)
**Concept:** All nodes return the same, most recent data

### Example:
> User updates profile in Node A\
> → Read from Node B immediately return updated data

**Strong Consistency:** Read always returns latest write\
**Weak Consistency:** Read might return stale data

## Availability (A)
**Concept:** Systems always responds to requests (even if data is stale)

### Example:
> Node A is down\
 → Node B still responds to requests\
 → No errors, no timeouts
>

**Guarantee:** Every non-failing node returns a response in reasonable time

## Partition Tolerance (P)
**Concept:** System works despite network splits

### Example:
> Network failure splits system:
> - US datacenter: Nodes A, B
> - EU datacenter: Nodes C, D
>
> Can't communicate but system still functions

**Reality:** Partitions are inevitable in distributed systems (network failures, latency, packet loss)

# Why You Can't Have All Three

## Scenario: Netwrok Partition Occurs
### Option 1: Choose Consistency (CP)
> Write to Node A (US)\
Network partition happens\
Read from Node B (EU) - can't sync with node A\
→ Return error or wait (sacrifice availability)

**Result:** System unavailable during partition but data is consistent

### Option 2: Choose Availability (AP)
> Write to Node A (US)\
Network partition happens\
Read form Node B (EU) - returns stale data\
→ Retrurn old value (sacrifice consistency)

**Result:** System available but data may be inconsitent

## Why Not All Three?
During partition, you must choose:
- Return stale data (availalbe but inconsistent)
- Return error/wait (consistent but unavailable)

You cannot have both fresh data AND availability during partition

# CAP Combinations
## CP (Consistency + Partition Tolerance)
**Trade-off:** Sacrifice availability during partitions

### Characteristics: 
- Blocks or returns error if data can't be guaranteed consistent
- Waits for partition to heal before responding
- Uses distributed locks, consensus algorithms
### Use Cases:
- Financial transactions
- Inventory management
- Banking systems
- Any system where wrong data is worse than no data

### Examples:
- **MongoDB** (with majority write concern)
- **HBase**
- **Redis** (with synchronous replication)
- **Consul**
- Traditional RDBMS with distributed transactions

### When to Choose CP:
- Data correctness is critical
- Can tolerate downtime
- Strong consistency required
- Financial/legal implications

## AP (Availability + Partition Tolerance)
**Trade-off:** Sacrifice consistency during partitions

### Characteristics:
- Always responds to requests
- May return stale data
- Eventually consistent
- Optimistic approach

### Use Cases:
- Social Media feeds
- Real-time analytics
- Shopping carts
- User sessions
- Systems where availability > correctness

### Examples:
- **Cassandra**
- **DynamoDB**
- **CouchDB**
- **Riak**
- DNS systems

### When to Choose AP:
- Availability is critical
- Can tolerate stale data
- Global distribution needed
- High throughput required

## CA (Consistency + Availability)
**Reality:** Not possible in distributed systems with partitions
### Only Possible:
- Single-node systems
- Systems without network issues (doesn't exist in practice)
### Examples:
- Traditional single-server RDBMS (PostgreSQL, MySQL on one server)
- But these aren't truly distributed

**Note:** Most systems claiming CA are actually CP that haven't experienced a partition yet

# CAP in Practice
## Real-World Trade-offs
### Banking System (CP)
> Scenario: Money transfer between accounts\
> Problem: Network partition between datacenters
>
> CP Choice:
> - Block transaction until partition resolves
> - Prevent double-spending
> - User sees error: "Service temporarily unavailable"

**Reasoning:** Better to reject transaction than lose money

### Instagram Feed (AP)
> Scenario: User posts photo\
> Problem: Network partition between datacenters
>
> AP Choice:
> - Accept post in local datacenter 
> - Show post immediately
> - Sync to other datacenters eventually

**Reasoning:** User seeing post 5 seconds later is acceptable

### E-commerce Hybrid
> Shopping Cart (AP) :
> - Add/remove items works always 
> - Slight inconsitency acceptable
>  
> Checkout/Payment (CP) : 
> - Must be consistent
> - Can show error during partition

# Common Misconceptions
## ❌ Myth 1: "NoSQL = AP, SQL - CP"
**Reality:**
- MongoDB can be CP with proper configuration
- PostgreSQL with async replication is effectively AP
- It's about configuration, not technology

## ❌ Myth 2: "Pick two and forget the third"
**Reality:**
- You MUST have partition Tolerance in distributed systems
- Real choice is: CP or AP during partitions
- Rest of the time, you get all three

## ❌ Myth 3: "It's a binary choice"
**Reality:**
- Many systems use hybrid approaches
- Different operations can have different guarantees
- Can be CP for writes, AP for reads

## ❌ Myth 4: "CAP means you can't scale"
**Reality:**
- Both CP and AP systems can scale
- CAP is about trade-offs during partitions
- Not about maximum scale

# PACELC Theorem (Extended CAP)
**Problem:** CAP only describes behavior during partitions. What about normal operation?

**PACELC:**
- **P**artition: Choose between **A**vailability and **C**onsistency (CAP)
- **E**lse (no partition): Choose between **L**atency and **C**onsistency

## PACELC Examples

### PA/EL (Cassandra, DyanmoDB)
- During **P**artition: Choose **A**vailability
- **E**lse: Choose **L**atency (low latency over consistency)
- Result: Always fast, eventually consistent

### PC/EC (HBase, BigTable)
- During **P**artition: Choose **C**onsistency
- **E**lse: Choose **C**onsistency (strong consistency even during normal operation)
- Result: Always consistent, may be slower

### PA/EC (MongoDB with default settings)
- During **P**artition: Choose **A**vailability
- **E**lse: Choose **C**onsistency 
- Result: Fast reads, consistent writes

# Decision Framework
## Question 1: What happens if user sees stale data?
- **Critical Problem** → CP
- **Acceptable** → AP

## Question 2: What happens if system is unavailable?
- **Unacaptable (user leave)** → AP
- **Tolerable (rare event)** → CP

## Question 3: Is this data financial or legal?
- **Yes** → CP
- **No** → Consider AP

## Question 4: What's the read/write pattern?
- **Read-heavy, writes rare** → AP (eventual consistency fine)
- **Write-heavy with dependencies** → CP (consistency critical)

## Question 5: Global distribution needed?
- **Yes, worldwide users** → AP (latency matters)
- **No, single region** → CP (easier to maintain consistency)

# Practical Examples
## Example 1: Twitter 
|Feature|Type|Reasoning|
|-|-|-|
|Tweets|AP|Slight delay acceptable
|Follower count|AP|Eventual consistency OK
|Direct messages|CP|Must be reliable
|Account balance|CP|Financial, must be accurate

## Example 2: Uber
|Feature|Type|Reasoning|
|-|-|-|
|Driver locations|AP|Real-time, availability critical
|Ride matching|AP|Speed matters
|Payment processing|CP|Financial transactions
|Trip history|CP|Legal/audit requirements

## Example 3: E-commerce
|Feature|Type|Reasoning|
|-|-|-|
|Product browsing|AP|Availability critical
|Shopping cart|AP|User experience matters
|Inventory check|CP|Prevent overselling
|Payment|CP|Financial accuracy
|Order confirmation|CP|Must be reliable


# Handling Partitions in Practice
## CP System Strategies
1. **Quorum reads/writes:** Require majority agreement
2. **Distributed locks:** Coordinate across nodes
3. **Consensus protocols:** Raft, Paxos for agreement
4. **Synchronous replication:** Wait for acknowledgment
5. **Reject operations:** Return errors during partition

## AP System Strategies
1. **Conflict resolution:** Last-write-wins, vector clocks
2. **Asynchronous replication:** Don't wait for acknowledgment
3. **Read repair:** Fix inconsistencies on read
4. **Anti-entropy:** Background sync process
5. **Accept all writes:** Merge conflict later

# Monitoring CAP Trade-offs
## Key Metrics
### For CP Systems:
- Availability percentage during partitions
- Time to recover from partition
- Number of rejected requests
- Latency during consensus

### For AP Systems:
- Replication lag
- Conflict rate
- Time to consistency (convergence time)
- Stale read percentage

# Key Takeaways
1. **CAP is about partitions:** You only choose during network failures
2. **Partition tolerance is mandatory:** For distributed systems
3. **Real choice: CP vs AP:** Consistency or Availability during partitions
4. **No perfect solution:** Every choice has trade-offs
5. **Hybrid is common:** Different parts of system can make different choices
6. **PACELC extends CAP:** Consider latency vs consistency even without partitions
7. **Business requirements decide:** Not a technical decision alone

# Quick Decision Guide
- Financial data → CP
- User experience critical → AP
- Both important → Hybrid (CP for critical, AP for non-critical)

# Remember
- CAP theorem is a constraint, not a goal
- Understanding CAP helps design better systems
- Most large systems use multiple databases with different CAP choices
- Test your assumptions: simulate partitions in staging

