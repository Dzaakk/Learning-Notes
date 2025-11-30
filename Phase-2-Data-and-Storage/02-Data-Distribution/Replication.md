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