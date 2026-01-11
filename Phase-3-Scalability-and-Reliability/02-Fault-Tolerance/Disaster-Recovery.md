Disaster Recovery
===

## Overview
Disaster Recovery (RD) is the process, policies, and procedures for restoring system operations after a catastrophic event. Unlike high availability which handles routines failures, DR focuses on recovering from large-scale disasters like data center failures, natural disasters, cyberattacks, or major infrastructure outages.

## Core Concept

### Recovery Objectives
**Recovery Time Objective (RTO):** Maximum acceptable downtime before business impact becomes unacceptable. Measured from disaster occurrence to full service restoration. Examples: 4 hours for e-commerce, 24 hours for internal tools, near-zero for critical financial systems.

**Recovery Point Objective (RPO):** Maximum acceptable data loss measured in time. How much data can you afford to lose? Examples: 0 minutes (no data loss) for financial transactions, 1 hour for customer  data, 24 hours for analytics data.

**Relationship:** Lower RTO/RPO requirements mean higher cost and complexity. RTO = 0 and RPO = 0 requires active-active systems across regions, which is extremely expensive.