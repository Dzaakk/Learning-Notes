Disaster Recovery
===

## Overview
Disaster Recovery (RD) is the process, policies, and procedures for restoring system operations after a catastrophic event. Unlike high availability which handles routines failures, DR focuses on recovering from large-scale disasters like data center failures, natural disasters, cyberattacks, or major infrastructure outages.

## Core Concept

### Recovery Objectives
**Recovery Time Objective (RTO):** Maximum acceptable downtime before business impact becomes unacceptable. Measured from disaster occurrence to full service restoration. Examples: 4 hours for e-commerce, 24 hours for internal tools, near-zero for critical financial systems.

**Recovery Point Objective (RPO):** Maximum acceptable data loss measured in time. How much data can you afford to lose? Examples: 0 minutes (no data loss) for financial transactions, 1 hour for customer  data, 24 hours for analytics data.

**Relationship:** Lower RTO/RPO requirements mean higher cost and complexity. RTO = 0 and RPO = 0 requires active-active systems across regions, which is extremely expensive.

### Disaster Categories
**Infrastructure Disaster:** Data center power outage, network failures, hardware failures at scale, facility damage from fire/flood.

**Natural Disaster:** Earthquakes, hurricanes, floods, wildfires affecting entire regions.

**Cyber Disaster:** Ransomware attacks, data breaches, DDoS attacks, malicious insider actions.

**Human Error:** Accidental deletion, misconfiguration, failed deployments, corrupted data.

## DR Strategies

### Backup and Restore (Cold DR)
Regularly backup data and configuration to remote location, restore from backups when disaster occurs, and rebuild infrastructure from scratch.

**Characteristics:**
- RTO: Hours to days
- RPO: Hours to days
- Cost: Lowest
- Complexity: Low

**Use Cases:** Non-critical systems, development environments, archived data, systems with high tolerance for downtime.

### Pilot Light
Maintain minimal version of system running in DR site with core data continuously replicated, scale up DR infrastructure when disaster occurs, and redirect traffic once scaled.

**Characteristics:**
- RTO: Hours 
- RPO: Minutes to hours 
- Cost: Low to medium
- Complexity: Medium

**Use Cases:** Business-critical applications with moderate downtime tolerance, cost-sensitive deployments requiring better recovery than backup/restore.

### Warm Standby
Run scaled-down version of full system in DR site with continuous data replication and all services running but at reduced capacity, scale up to handle full production load when needed.

**Characteristics:**
- RTO: Minutes to hours 
- RPO: Minutes 
- Cost: Medium to high
- Complexity: Medium to high

**Use Cases:** Important production systems, applications requiring quick recovery, systems with moderate to strict RTO/RPO requirements.

### Hot Standby (Active-Passive)
Maintain fully scaled DR environment that's identical to production with real-time data synchronization, keep DR site ready but not serving traffic, and failover quickly when disaster occurs.

**Characteristics:**
- RTO: Minutes 
- RPO: Near-zero to minutes
- Cost: High
- Complexity: High

**Use Cases:** Mission-critical systems, financial services, healthcare applications, any system with strict RTO requirements.

### Multi-Site Active-Active
Run multiple production sites simultaneously with all sites handling live traffic, data replicated in real-time across sites, and automatic traffic routing around failures.

**Characteristics:**
- RTO: Seconds to minutes 
- RPO: Near-zero
- Cost: Very high
- Complexity: Very high

**Use Cases:** Absolutely critical systems, global applications, zero-downtime requirements, systems where even minutes of downtime costs millions.