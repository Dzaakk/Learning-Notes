Replication
===

# What is Replication?
**Definition:** Keeping copies of the same data on multiple servers/nodes.

## Core Benefits:
- **High Availability** - System stays up if one server fails
- **Fault Tolerance** - Data survives hardware failures
- **Performance** - Read queries distributed across replicas
- **Geographic Distribution** - Data closer to users globally

# Why Replication Matters in System Design

## Scenario Without Replication
Single Database Server\
├── All reads go here\
├── All writes go here\
└── If this fails → Entire system down

### Problems:
- Single point of failure
- Limited read capacity
- High latency for distant users
- Disaster recovery is difficult

## Scenario With Replication
Primary Database (writes)\
├── Replica 1 (reads - US East)\
├── Replica 2 (reads - US West)\
├── Replica 3 (reads - Europe)\
└── Replica 4 (reads - Asia)

### Benefits:
- No single point of failure
- Distribute read load
- Low latency globally
- Easy backups and disaster recovery

# Replication Strategies

## 1. Single-Leader Replication (Master-Slave)

### Architecture:
![single leader replication](./images/single-leader-replication.excalidraw.png)

### How It Works:
1. All wrtites go to the leader
2. Leader logs changes to replication log
3. Follower pull changes and apply them
4. Reads can go to any follower

### Replication Methods:

#### a) Synchronous Replication
Client → Write to Leader → Wait for all followers to confirm → Respond to client

- **Pros:** Strong consistency, no data loss
- **Cons:** Slower writes, leader waits for slowest follower
- **Use:** Critical data (financial transactions)

#### b) Asynchronous Replication
Client → Write to Leader → Respond immediately → Followers sync later

- **Pros:** Fast writes, leader doesn't wait 
- **Cons:** Potential data loss if leader fails, eventual consistency
- **Use:** Most web applications (default MySQL, PostgreSQL)

#### c) Semi-Synchronous (Hybrid)
Wait for at least 1 follower, rest async

- **Pros:** Balance between speed and safety 
- **Cons:** More complex
- **Use:** Production systems with good balance

### Pros & Cons of Single-Leader

#### Advantages:
✅Simple to understand and implement\
✅Easy to scale reads (add more followers)\
✅Consistent writes (single source of truth)\
✅Built-in to most databases

#### Disadvantages
❌Single point of failure for writes\
❌Follower can have stale data (replication lag)\
❌Cannot scale writes (all go to one leader)\
❌Failover can be complex

### Use Cases:
- Most web applications
- Read-heavy workloads
- When strong consistency for reads is not critical
- Examples: Instagram feeds, Twitter timelines, e-commerce product listings

### Popular Implementation
- MySQL (Master-Slave replication)
- PostgreSQL (Streaming replication)
- MongoDB (Replice Sets with primary)

## 2. Multi-Leader Replication (Master-Master)

### Architecture:
![multi leader replication](./images/multi-leader-replication.excalidraw.png)

### How It Works:
1. Multiple nodes accept writes
2. Each leader replicates to others
3. Writes propagate to all leaders
4. Conflict resolution needed

### Conflict Example:
User A (US) updates price to $10\
User B (EU) updates price to $15\
→ Both leaders accept writes\
→ Conflict! Which value wins?

### Conflict Resolution Strategies:

#### a) Last Write Wins (LWW)
Compare timestamp, keep latest → Simple but can lose data

#### b) Application-Level Resolution
Application decides based on business logic → Most flexible but complex

#### c) Merge Values
Combine both changes if possible → Example: Shopping cart (merge items)

### Pro & Cons of Multi-Leader
#### Advantages:
✅ Better performance for globally distributed writes\
✅ Can write to nearest leader (low latency)\
✅ Resilient to datacenter failures\
✅ Better write scalability

#### Disadvantages
❌ Complex conflict resolution\
❌ Eventual consistency (writes take time to propagate)\
❌ Data conflicts can occur\
❌ Harder to reason about

### Use Cases:
- Multi-datacenter deployments
- Offline-first application (mobile apps)
- Collaborative editing tools
- Examples: Google Docs, CouchDB apps, Notion

### Popular Implementation
- MySQL (with MySQL Group Replication)
- PostgreSQL (BDR - Bi-Directional Replication)
- CouchDB (built-in multi-leader)

## 3. Leaderless Replication (Peer-to-Peer)

### Architecture:
![leaderless replication](./images/leaderless-replication.excalidraw.png)

### How It works:
1. Client writes to multiple nodes in parallel
2. Read from multiple nodes and reconcile
3. Use quorum to determine success

### Quorum Configuration:
N = Total replicas (e.g., 3)\
W = Write quorum (e.g., 2)\
R = Read quorum (e.g., 2)

Rule: W + R > N guarantees consistency

### Example:
Write to 3 nodes:\
├── Node 1: Success\
├── Node 2: Success\
├── Node 3: Failed

W = 2, so write succeeds (2 out of 3)

### Read Repair:
Read from 3 nodes:\
├── Node 1: value = "hello" (v2)\
├── Node 2: value = "hello" (v2)\
├── Node 3: value = "hi" (v1)

Return "hello" (majority), update Node 3

### Pros & Cons of Leaderless

#### Advantages:
✅ High availability (no single leader)\ 
✅ No failover needed\
✅ Handles network partitions well\
✅ Good write scalability

#### Disadvantages:
❌ Eventual consistency\
❌ Complex client logic\
❌ Conflict resolution needed\
❌ Higher operational complexity

### Use Cases:
- High available systems (tolerates node failures)
- Distributed systems with partitioning
- Shopping carts, session stores
- Examples: Amazon DynamoDB, Cassandra

### Popular Implementation:
- Cassandra
- DynamoDB
- Riak

## Replication Strategies Comparison
|Aspect|Single-Leader|Multi-Leader|Leaderless|
|-|-|-|-|
|Consistency|Strong (on leader)|Eventual|Eventual|
|Write Scalability|Limited|Good|Excellent|
|Read Scalability|Excellent|Excellent|Goood|
|Complexity|simple|Medium|High|
|Conflict Resolution|None needed|Required|Required|
|Use Case|Read-heavy|Multi-datacenter|High availability|

# Replication Lag

**Definition:** Time delay between write on leader and appearing on follower.

## Types of Inconsistencies:

### 1. Read-After-Write Inconsistency
User posts comment → Reads immediately → Comment not visible yet  

#### Solution:
- Read from leader for user's own data
- Track last update time, read from leader if recent
- Use sticky sessions (same user → same follower)

### 2. Monotonic Reads Problem

User refresh page:\
Request 1 → Follower A (has update) →  Sees comment\
Request 2 → Follower B (lagging) →  Comment dissapears!

#### Solution:
- Sticky sessions (same user → same replica)
- Read from leader for critical data
- Timestamp-based routing

### 3. Consistent Prefix Reads
Question: "What is 2+2?"\
Answer: "4"

User sees:\
Answer: "4" (from fast follower)\
Question: ?? (from slow follower, not yet replicated)

#### Solution:
- Write related data to same partition
- Use transactions
- Timestamp ordering

# Handling Replication Failures

## Leader Failure (Failover)

### Process:
1. **Detect failure** (heartbeat timeout, typically 30s)
2. **Choose new leader** (usually most up-to-date follower)
3. **Reconfigure system** (client redirect to new leader)
4. **Sync old leader** (if it comes back, make it follower)

### Challenges:

#### a) Split Brain
Network partition → Both nodes think they're leader → Data divergence, conflicts\
**Solutions:** Use consensus algorithms (Raft, Paxos)

#### b) Data Loss
Leader fails before replicating recent writes → Those writes are lost\
**Solutions:** Synchronous replication for critical data

#### c) Choosing New Leader
Which follower should be leader?\
→ Most up-to-date data\
→ Lowest latenc\
→ Manual selection

## Follower Failure

### Recovery Process:
1. Follower crashes or disconnects
2. Keeps track of last processed log position
3. Reconnects and requests missing changes
4. Catches up and resumes normal operation

**This is usually straightforward and automatic**

# Replication in Different Databases

## MySQL Replication
### Setup:
```sql
-- On Leader
CREATE USER 'replicator'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';

-- On Follower
CHANGE MASTER TO
  MASTER_HOST='leader-host',
  MASTER_USER='replicator',
  MASTER_PASSWORD='password';
START SLAVE;
```

### Types:
- Statement-based (replicate SQL statements)
- Row-based (replicate actual data changes)
- Mixed (automatic choice)


## PostgreSQL Replication
### Streaming Replication (default):
```bash
# On Follower
pg_basebackup -h leader -D /var/lib/postgresql/data -U replicator -P

# Configure standby
standby_mode = 'on'
primary_conninfo = 'host=leader port=5432 user=replicator'
```

### Types:
- Physical replication (binary WAL shpping)
- Logical replication (table-level, selective)

## MongoDB Replica Set
### Setup:
```javascript
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongodb1:27017" },
    { _id: 1, host: "mongodb2:27017" },
    { _id: 2, host: "mongodb3:27017" }
  ]
})
```

### Features:
- Automatic failover
- Read preferences (primary, secondary, nearest)
- Write concenr levels

# Real-World Replication Architectures

## Example 1: Instagram (Read-Heavy)
Primary PostgreSQL (writes)\
├── Replica 1 (timeline reads)\
├── Replica 2 (feed reads)\
├── Replica 3 (search)\
└── Replica 4 (analytics)

replication: Asynchronous\
Lag: < 1 second acceptable

## Example 2: Netflix (Global)
US Datacenter
├── Leader (writes)
└── Local followers (reads)

EU Datacenter
├── Leader (writes)
└── Local followers (reads)

Multi-leader replication between datacenters

## Example 3: E-commerce Checkout
Primary DB (orders, payments)\
└── Synchronous replica (backup)

Product Catalog DB\
├── Primary\
└── Multiple async replicas (for high read traffic)

Inventory DB\
├── Primary (strict consistency)\
└── Sync replica (cannot oversell)

# Replication Best Practices

## 1. Monitor Replication Lag
```sql
-- PostgreSQL
SELECT now() - pg_last_xact_replay_timestamp() AS replication_lag;

-- MySQL
SHOW SLAVE STATUS\G
```
**Alert if lag > threshold (e.g., 5 seconds)**

## 2. Use Connection Pooling
Application → Connection Pool → Primary/Replicas

Benefits:
- Reuse connections
- Route reads to replicas
- Handle failover

## 3. Implement Health Checks
```python
def check_replica_health(replica):
    lag = get_replication_lag(replica)
    if lag > MAX_LAG_SECONDS:
        remove_from_pool(replica)
        alert_ops_team()
```

## 4. Plan for Failover
- Automate detection (heartbeat monitoring)
- Test failover regularly (chaos engineering)
- Document failover procedures
- Use orchestration tools (Patroni for PostgreSQL)

## 5. Secure Replication Traffic
- Use SSL/TLS for replication
- Authentiacate replication users
- Restrict replication to private network
- Encrypt sensitive data at resta

# Commong Replica Mistakes
## ❌Mistake 1: Reading from Replicas Blindly
```python
# Bad: May read stale data
user.update_profile()
profile = read_from_random_replica()  # Might be old data
```
**Fix:** Read-after-write from primary or check lag

## ❌Mistake 2: Not Handling Failover
```python
# Bad: Hardcoded primary host
db.connect("primary-host:5432")
```
**Fix:** Use service discovery or connection pooler

## ❌Mistake 3: Ignoring Replication Lag
Write to primary → Read from replica (lagging) → Bug reports!

**Fix:** Monitor lag, route critical reads to primary

## ❌Mistake 4: Not Testing Failover
"Our failover works in theory" → First real failure: Disaster

**Fix:** Regular failover drills

# Key Takeaways
1. **Single-Leader:** Simple, works for most applications (default choice)
2. **Multi-Leader:** Use for multi-datacenter, offline apps
3. **Leaderless:** Use for high availability, eventual consistency OK
4. **Replication Lag:** Always exists in async replication, handle it
5. **Failover:** Plan and test regularly
6. **Monitoring:** Track lag, health, and performance
7. **Trafe-offs:** Consistency vs Availability vs Performance


