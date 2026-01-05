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

# Deployment Considerations

## Zero-Downtime Deployments

### Rolling Updates
Gradually replace instances with new versions, maintaining overall capacity throughout deployment.

### Bule-Green Deployment
Run two identical environtments, switch traffic from blue (old) to green (new) instantly, maintain blue as instant rollback option.

### Canary Releases
Route small percentage of traffic to new version, monitor for issues, gradually increase traffic or rollback quickly.

## Database Migrations
Plan backward-compatible schema changes, use multi-phase migrations (add columns, deploy code, migrate data, remove old columns), and implement feature flags to decouple code deployment from data changes.

## Configuration Management
Store configuration externally, support hot-reload without restarts, version control all configuration, and implement gradual rollouts of configuration changes.

# Monitoring and Alerting

## Key Metrics

### Availability Metrics
- Uptime percentage over various time windows
- Error rates and types (4xx vs 5xx)
- Request success rate

### Performance Metrics
- Response time percentiles (p50, p95, p99)
- Throughput (requests per second)
- Resource utilization (CPU, memory, disk)

### System Health
- Failed health checks
- Circuit breaker state changes
- Replication lag in databases

## Alerting Best Practices
Alert on symptoms (users are affected) rather than causes, reduce alert fatigue through proper thresholds and deduplication, use severity levels appropiately, and ensure actionable alerts with clear runbooks.

# Capacity Planning

## Scaling Strategies

### Vertical Scaling (Scale Up)
Add more resources to existing machines. Simpler but has limits and creates larger blast radius.

### Horizontal Scaling (Scale Out)
Add more machines. More complex but unlimited scaling and smaller failure impact.

### Auto-Scalling
Automatically adjust capacity based on demand using metrics like CPU utilization, queue length, oor custom metrics with proper warmup time and cooldown periods.

## Capacity Buffer
Always maintain spare capacity (typically 20-40% overhead) to handle unexpected traffic spikes, failover scenarios where remaining nodes must absorb failed node's load, and traffic growth between capacity reviews.

# Cost vs Availability Trade-offs
Achieving higher availability is exponentially more expensive. Consider business requirements, SLA commitments, cost of downtime vs cost of redundancy, and acceptable risk levels.

Going from 99% to 99.9% migh cost 2x, but 99.9% to 99.99% might cost 5-10x. Each application has its optimal availability target based on business needs.

# Common Pitfalls
1. **Over-engineering:** Building for five nines when three nines suffice wastes resources
2. **Ignoring dependencies:** Third-party services often become the weakest link
3. **Untested failover:** Backup systems that haven't been tested often fail when needed
4. **Shared fate:** Multiple "redundant" components sharing common dependencies
5. **Cascading failures:** One failure triggering chain reaction across system
6. **Configuration drift:** Production and backup environments becoming inconsistent

# Test High Availability

**Regular Drills:** Pracitce failover procedures quarterly, simulate various failure scenarios, and involve entire team in excersises.

**Chaos Engineering:** Intentionally inject failures in production (with safeguards) using tools like Chaos Monkey, test system resilience continuously, and build confidence in automated recovery.

**Load Testing:** Verify system performs under expected peak load, test beyond expected capacity to find breaking points, and simulate traffic patterns similar to production.

# Real-World Example
A streaming service targeting 99.99% availability migh implement:
- Application across 3 availability zones with auto-scaling (minimum 9 instances)
- Database primary in AZ-1 with synchronous replica in AZ-2 and asynchronous replica in AZ-3
- Distributed cache cluster with 6 nodes (2 per AZ)
- Multiple load balancers with health checks every 5 seconds
- Circuit breakers with 5-second timeout and 50% error threshold
- Blue-green deployments for application updates
- Online schema migrations tools for database changes

This architecture tolerates single AZ failure, single server failures at any tier, database failover with under 30 seconds downtime, and can handle 2x normal traffic without degradation.

