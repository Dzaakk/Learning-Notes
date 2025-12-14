Consistency Models
===

# What are Consistency Models?
**Definition:** Rules that define how and when writes become visible to readers in a distributed system\

**Spectrum:** From strongest (most restrictive) to weakest (most flexible)
![consistency models](./images/consistency-models.excalidraw.png)

# Why Consistency Models Matter
**Trade-offs:**
- **Strong consistency:** Easy to reason about, but slower and less available
- **Weak consistency:** Fast and highly available, but complex application logic

**Impact on:**
- System design decisions
- Database choise
- Application complexity
- User experience

# Strong Consistency Models

## 1. Linearizability (Strongest)
**Definition:** Operations appear to execute atomically and in real-time order

**Guarantee:** Once a write completes, all subsequent reads see that write (no going back in time)

### Characteristics:
- Operations have real-time ordering
- System behave like a single copy
- Once updated, everyone sees the update
- Acts as if there's one centralized database

### Example:
![linearizability example](./images/linearizability-example.excalidraw.png)

**Once write completes, EVERYONE sees new value immediately**

### Pros & Cons:
**Advantages:** 

✅ Easiest to reason about (behaves like single machine)\
✅ No surprises for developers\
✅ Correct by default

**Disadvantages:**

❌ Highest latency (must coordinate)\
❌ Lowest availability (blocks during failures)\
❌ Poor performance at scale

### Use Cases:
- Distributed locks
- Leader election
- Coordination services
- Critical sections
- Any system where operations must be globally ordered

### Examples:
- **etcd** (with quorum reads)
- **ZooKeeper**
- **Consul**
- Google **Spanner** (with TrueTime)

### Implementation:
- Consensus algorithm (Raft, Paxos)
- Synchronous replication with quorums
- Distributed transactions (2PC, 3PC)

## 2. Sequential Consistency

**Definition:** Operations appear to execute in some sequential order, but not necessarily real-time order

**Guarantee:** All nodes see operations in the same order, but order may differ from real-time

### Characteristics:
- Global total order of operations
- Order must be consistent across al nodes
- But order can differ from actual execution time

### Example:
![sequential example](./images/sequential-example.excalidraw.png)

### Difference from Linearizability:
![linearizability vs sequential](./images/linearizability-vs-sequential.excalidraw.png)

### Pros & Cons:
**Advantages:**\
✅ Easier to implement than linearizability\
✅ Better performance (no strict time ordering)\
✅ Still relatively simple to reason about

**Disadvantages:**\
❌ Can be confusing (operations reordered)\
❌ Still requires coordination\
❌ Not as intuitive as linearizability

### Use Cases:
- Systems where order matters but timing doesn't
- Replicated state machinse
- Some database replication

### Examples:
- Some configurations of distributed databases
- Certain memory models in multiprocessor systems

