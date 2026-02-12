Distributed Systems Basics
===

## What is a Distributed System?
A distributed system is a collection of independent computers that work together by communicating over a network to achieve a common goal. Each computer (node) has its own memory and processor, and they coordinate only by passing messages.

**Example:** A web application with separate database servers, application servers, and cache servers working together.

## Why Use Distributed Systems?

### Scalability
Single machines have limits. Distributed systems let you scale horizontally by adding more machines, which is often cheaper than buying one massive machine.

**Example:** Handle 1 million requests/second by using 100 servers instead of one impossibly expensive super-computer.

### Reliability and Fault Tolerance
If one machine fails, others continue working. Redundancy across multiple nodes prevents total system failure.

**Example:** Store data on 3 servers so if 1 fails, you still have 2 copies.

### Geographic Distribution
Place servers closer to user worldwide to reduce latency.

**Example:** Netflix has servers in multiple continents to serve local users quickly.

### Performance Through Parallelism
Process tasks simultaneously across many machines.

**Example:** Analyze 1 billion images by distribting work across 1000 machines.

## Fundamental Challenges

### 1. Network Unreliability
Networks lose messages, delay them, duplicate them, or deliver them out of order. When you don't get a response, you can't tell if the request was lost, the server crashed, or the response was lost.

**Implication:** Must handle retries, timeouts, and idempotency.

### 2. Partial Failures
Some nodes fail while others keep working. Unlike single machines where everything fails together, distributed systems are simultaneously healthy and broken.

**Implication:** Cannot distinguish between slow nodes and crashed nodes just by observing network behavior.

### 3. Clock Synchronization
Each node has its own clock that drifts at different rates. Even with NTP, clocks are only accurate to within milliseconds at best.

**Implication:** Cannot realiably order events across different machines using timestamps alone.

### 4. Consistency
When data is replicated across nodes, keeping all copies in sync is hard. Updates take time to propagate, creating temporary inconsistencies.

**Implication:** Must choose between strong consistency (slower) and eventual consistency (faster but temporarily inconsistent).

## The CAP Theorem
**States:** In a distributed system, you can have at most 2 of 3 guarantees:

- **Consistency (C):** All nodes see the same data at the same time
- **Availability (A):** Every request gets a response (success or failure)
- **Partition Tolerance (P):** System works despoite network partitions

Since network partitions WILL happen, you must choose between C or A during partitions.

### CP Systems (Consistency + Partition Tolerance)
Refuse requests during partitions to maintain consistency. Sacrifice availability.

**Examples:** HBase, MongoDB (default), ZooKeeper, banking systems\
**When to use:** Correctness matters more than availability

### AP Systems (Availability + Partition Tolerance)
Keep serving requests during partitions but may return stale data. Sacrifice consistency.

**Examples:** Cassandra, DynamoDB, Riak, social media feeds\
**When to use:** Availability matters more than always having latest data

### CA Systems 
Not possible in distributed systems because partitions are inevitable. Only works on single machine (not distributed).

## Communication Patterns

### Synchronous (Request-Response)
Sender waits for response before continuing.

**Pros:** Simple, immediate feedback\
**Cons:** Sender blocked, both services must be up simultaneously\
**Example:** HTTP REST API calls

### Asynchronous (Message Queue)
Sender doesn't wait, continues immediately.
**Pros:** Decouples services, resilient to temporary failures\
**Cons:** More complex, delayed feedback\
**Example:** RabbitMQ, Kafka, SQS

### Publish-Subscribe
Publishers send to topics, subscribers receive from topics they're interested in.
**Pros:** Loose coupling, dynamic scaling\
**Cons:** Harder to debug\
**Example:** Kafka topics, Redis Pub/Sub

## Data Replication Strategies

### Single-Leader Replication
One leader accepts all writes, replicates to followers.

**Pros:** Simple, clear ordering\
**Cons:** Leader is bottleneck and single point of failure for writes\
**Example:** MySQL primary-replica setup 

### Multi-Leader Replication
Multiple leaders accept writes, sync between each other.

**Pros:** Better write availability, good for multi-region\
**Cons:** Must handle write conflicts\
**Example:** Multi-region database deployments

### Leaderless Replication
Write to any node, use quorum for reads/writes.

**Pros:** High availability, no single leader to fail\
**Cons:** Comple conflict resolution\
**Example:**  Cassandra, DynamoDB

