Failover and Redundancy
===

# Overview
Failover and redundancy are fundamental fault tolerance mechanisms that ensure system availability when components fail. Redundancy involves having backups components ready, while failover is the process of switching to those backups when primary components fail.

# Core Concepts

## Redundancy
Redundancy means having multiple copies of critical system components. When one fails, others can take over without service interruption.

### Types of Redundancy
- **Active-Active:** All redundant components actively handle requests simultaneously, sharing the load
- **Active-Passive:** Primary component handles requests while backup remains on standby, only activating when primary fails
- **N+1 Redundancy:** N components needed for operation, plus 1 spare (e.g., 3+1 servers means 3 handle normal load, 1 is backup)
- **N+M Redundancy:** N components needed, M spares available (e.g., 10+2 means 10 active, 2 backup)

## Failover
Failover is the automatic or manual process of switching from a failer component to a redundant one

### Failover Types
- **Automatic Failover:** System detects failure and switches automatically without human intervention
- **Manual Failover:** Operations team manually triggers the switch to backup systems
- **Hot Failover:** Backup is already running and synchronized, switch is instantaneous
- **Warm Failover:** Backup is running but not fully synchronized, requires brief sync period
- **Cold Failover:** Backup must be started from scratch, longer recovery time

# Key Components

## Load Balancers
Distribute traffic across multiple servers and detect failures to route around them. Essential for implementing redundancy at the application layer.

## Database Replication
- **Master-Slave:** Single master handles writes, multiple slaves handle reads. If master fails, promote a slave
- **Master-Master:** Multiple masters can handle writes simultaneously, more complex but provides better availability
- **Quorum-based:** Requires majority of nodes to agree on operations, prevents split-brain scenario

## Geographic Redundancy
Replicate systems across multiple data centers or regions to protect against regional failures like natural disasters or network partition.

# Implement Considerations

## State Management
Stateless service are easier to make redundant since any instance can handle any request. Form stateful services, you need to replicate state across instance, typically using distributed caches, databases, or session stores.

## Data Consistency
With redundant data stores, you must decide between consistency models. Strong consistency ensures all replicas have identical data but impacts performance. Eventual consistency allows temporary inconsistencies for better availability and preformance.

## Failover Detection Time
Faster detection means quicker recovery but risks false positives. Most systems use timeouts between 3-30 seconds. You need to balance quick recovery against stability.

## Split-Brain Problem
When network partition occurs, multiple components may think they're primary, leading to data conflicts. Solutions include quorum systems, fencing mechanisms, or designated tie breakers.

# Common Patterns

## DNS Failover
Update DNS records to point to backup servers when primary fails. Simple but has TTL delay, typically taking minutes to propagate.

## Virtual IP Failover
Move a virtual IP address from failed server to backup. Much faster than DNS (seconds) and transparent to clients.

## Database Failover with Proxy
Use a database proxy that maintains connections to multiple database replicas. When primary fails, proxy redirects queries to promoted replica while applications maintain their connections.

## Circuit Breaker
Automatically stop sending requests to failing services, giving them time to recover. Prevents cascade failures and allows graceful degradation.

# Metrics to Monitor
- **Recovery Time Objective (RTO):** Maximum acceptable downtime
- **Recovery Point Objective (RPO):** Maximum acceptable data loss (measured in time)
- **Failover Time:** Actual time taken to detect failure and switch to backup
- **Mean Time Between Failures (MTBF):** Average time between system failures
- **Mean Time To Recovery (MTTR):** Average time to restore service after failure

# Trade-offs

## Active-Active vs Active-Passive
- Active-Active provides better resource utilization but is more complex to implement
- Active-Passive is simpler but wastes backup resources during normal operation

## Automatic vs Manual Failover
- Automatic is faster but risk unnecessary failovers from false postiives
- Manual is more controlled but requires human interventions and is slower

## Synchronouse vs Asynchronous Replication
- Synchronous ensures no data loss but impacts write performance
- Asynchronouse is faster but risks data loss during failover

# Best Practices
1. **Test failover regularly** through chaos engineering and disaster recovery drills
2. **Monitor everything** including backup systems, not just primary ones
3. **Automate failover process** to reduce human error and recovery time
4. **Document runbooks** for manual intervention scenarios
5. **Plan for cascading failrues** where one failure triggers others
6. **Consider cost vs benefit** as full redundancy is expensive
7. **Implement gradual rollback** mechanisms to recover from bad deployments

# Real-World Example
A typical web application migh implement redundancy as follows:
- Multiple application servers behind a load balancer (active-active)
- Database with master-slave replication (active-passive with automatic failover)
- Distributed cache cluster (active-active with consistent hashing)
- Infrastructure replicated across multiple availability zones in the same region for zone-level fault tolerance

When the master database fails:
- Monitoring system detects the loss of heartbeat within 10 seconds
- Automatically promotes the most up-to-date slave to master 
- Updates the connection proxy

Applications reconnect within 30 seconds total, achieving an RTO of under 1 minute with zero data loss (RPO of 0) 