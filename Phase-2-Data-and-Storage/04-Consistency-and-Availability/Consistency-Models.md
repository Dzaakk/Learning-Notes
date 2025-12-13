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