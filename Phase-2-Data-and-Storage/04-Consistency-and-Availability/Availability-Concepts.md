Availability Concepts
===

# What is Availability?
**Definition:** The proportion of time a system is operational and accessible

**Formula:**\
![availability formula](./images/availability-formula.png)

**Example:**
- Uptime: 8,759 hours/year
- Downtime: 1 hour/year
- Availability: 8,759 / (8,579 + 1) = 99.98%

# The Nines of Availability
|Availability|Downtime/Year|Downtime/Month|Downtime/Week|Use Case|
|-|-|-|-|-|
|90% (one nine)|36.5 days|3 days|16.8 hours|Internal tools
|99% (two nines)|3.65 days|7.2 hours|1.68 hours|Basic websites
|99.9% (three nines)|8.76 hours|43.8 minutes|10.1 minutes|Standard Saas
|99.99% (four nines)|52.56 minutes|4.38 minutes|1.01 minutes|Business-critical 
|99.999% (five nines)|5.26 minutes|26.3 seconds|6 seconds|Financial systems
|99.9999% (six nines)|31.5 seconds|2.63 seconds|0.6 seconds|Mission-critical

**Note:** Each additional nine costs exponentially more

# High Availability (HA) vs Fault Tolerance

## High Availability
**Goal:** Minimize downtime through redundancy and quick recovery

**Characteristics:**
- Brief service interruptions acceptable
- Automatic failover (seconds to minutes)
- Cost-effective
- Some data loss acceptable

**Example:**
- Primary database fails
- Failover to replica in 30 seconds
- 30 seconds of downtime

## Fault Tolerance
**Goal:** Zero downtime through complete redundancy

**Characteristics:**
- No service interruption
- Instant failover (miliseconds)
- Expensive
- No data loss

**Example:**
- Active-active setup
- Both systems running simultaneously
- Failure is transparent

## Comparison
|Aspect|High Availability|Fault Tolerance|
|-|-|-|
|Downtime|Seconds/minutes|None (instant)
|Cost|Moderate|Very high
|Complexity|Medium|High
|Data loss|Possible|None
|Use case|Most applications|Critical systems

# Components of Availability

## 1. Redundancy

### Active-Passive (Hot Standby)
![active passive](./images/active-passive.png)

**Characteristics:**
- Secondary is ready but not serving traffic
- Failover time: seconds to minutes
- Cost-effective
- Most commong approach

**Example:** Database primary with read replica promoted on failure

### Active-Active (Load Balanced)
![active active](./images/active-active.png)

**Characteristics:**
- All nodes serve traffic
- No failover needed (instant)
- Better resource utilization
- More complex

**Example:** Multiple API servers behind load balancer

### N + 1 Redundancy
> Required capacity: 100 units\
> Deploy: 101 units (1 extra)

**Characteristics:**
- One extra component beyond mininmum needed
- Survives single failure
- Balance between cost and reliability

### N + 2 Redundancy
> Required capacity: 100 units\
> Deploy: 102 units (2 extra)

**Characteristics:**
- Survives two simultaneous failures
- Higher availability'
- Higher cost


