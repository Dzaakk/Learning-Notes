High Availability
===

# Overview
High Availability (HA) is a system design approach that ensures a service remains operational and accessible for the maximum possible time. It's measured by uptime percentage and aims to minimize both planned and unplanned downtime through redundancy, failover mechanisms, and fault-tolerant architectures.

# Availability Metrics

## The Nines of Availability
Availability is typically expressed as a percentage or "nines":
- **99% (Two Nines):** 3.65 days downtime per year - Acceptable for non-critical systems
- **99.9% (Three Nines):** 9.76 hours downtime per year - Standard for most web applications
- **99.99% (Four Nines):** 52.56 minutes downtime per year - Required for business-critical applications
- **99.999% (Five Nines):** 5.26 minutes downtime per year - Financial services, healthcare systems
- **99.9999% (Six Nines):** 31.5 seconds downtime per year - Extremly critical infrastructure

### Formula
> Availability = (Total Time - Downtime) / Total Time x 100%

## Composite Availability
When components are in series (dependent), multiply their availability:
- Service A: 99.9% and Service B: 99.9% → Combined: 99.8%

When components are in parallel (redundant), calculate inverse:
- Two servers at 99% each → Combined: 99.99% (1 - (0.01) * 0.01)


# Core Principles

## Eleminate Single Point of Failure (SPOF)
Every component that can fail should have a redundant backup. Common SPOFs include single database instances, single load balancers, single network links, and single datacenter dependencies.

## Fault Detection and Recovery
Systems must quickly detect failures through health checks and monitoring, then automatically recover through failover, auto-scaling, or self-healing mechanisms without requiring human intervention.

## Graceful Degradation
Instead of complete failure, system should continue operating with reduced functionality. Examples include serving cached data when database is unavailable, disabling non-critical features, or providing read-only access.

## Minimize Recovery Time
Reduce Mean Time To Recovery (MTTR) through automated failover, pre-warmed backup systems, and well-tested recover procedures.

# Architectural Patterns

## Multi-Tier Redundancy

### Load Balancer Tier:
- Multiple load balancers in active-passive or active-active configuration
- Health checking to detect and route around failures
- Session persistence mechanisms for stateful applications

### Application Tier:
- Horizontally scaled stateless application servers
- Auto-scaling based on traffic and health metrics
- Deployed across multiple availability zones

### Data Tier:
- Database replication with automatic failover
- Read replicas for scaling read operations
- Regular backups with point-in-time recovery

### Cache Tier:
- Distributed caching with cluster mode
- Cache warming strategies to prevent cold starts
- Fallback to database when cache fails

## Geographic Distribution

### Multi-AZ (Availability Zone)
Distribute resources across multiple isolated datacenters within a region, protecting against single datacenter failures while maintaining low latency.

### Multi-Region
Deploy complete system replicas across geographic regions, protecting against regional failures and reducing latency for global users, but adds complexity for data consistency.

## Active-Active vs Active-Passive

### Active-Active
All instances handle production traffic simultaneously. Provides better resource utilization and no failover delay, but requires sophisticated data synchronization and conflict resolution.

### Active-Passive
Primary handles traffic while secondary remains on standby. Simpler to implement and maintain data consistency, but wastes standby resources and has brief downtime during failover

# Implementation Strategies

## Database High Availability

### Replication Strategies:
- Synchronous replication ensures zero data loss but impacts write performance
- Asynchronous replication provides better performance but risks data loss
- Semi-synchronous replication balances both concerns

### Cluster Solutions:
- MySQL with Group Replication or Galera Cluster
- PostgreSQL with Patroni or Stolon
- MongoDB Replica sets
- Cassandra or DynamoDB for distributed databases

## Stateless Application Design
Design applications to store no session state locally. Use external stores for sessions (Redis, Memccached), shared file systems for uploads, and JWT tokens for authentication. This allows any instance to handle any request.

## Health Checks
Implement multiple levels of health checks including shallow checks (ping endpoints), deepn checks (database connectivity, dependency availability), and liveness checks (application is running) vs readiness checks (application is ready to serve traffic).

## Circuit Breakers
Automatically prevent requests to failing services, allowing them time to recover and preventing cascade failures. Typically uses states of closed (normal), open (failing, reject request), and half-open (testing recovery).
