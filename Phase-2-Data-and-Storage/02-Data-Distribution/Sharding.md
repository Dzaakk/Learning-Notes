Sharding
==

# What is Sharding?
**Definition:** Splitting a large database into smaller, independent pieces (shards) across multiple servers

**Key Concept:** Each shard contains a subset of the total data

**Visual:**\
Before Sharding (vertical Scaling):\
[Single Database - 10 TB]\
└── All data in one server

After Sharding (Horizontal Scaling):\
[Shard 1: 2TB] [Shard 2: 2TB] [Shard 3: 2TB] [Shard 4: 2TB] [Shard 5: 2TB]\
└── Data distributed across 5 servers

# Why Sharding? 

## Problem: Database Limits
**Storage Limit:**\
Database growing to 10TB+ → Single server disk capacity maxed out

**Performance Limit:**\
Millions of queries per second → Single server CPU/memory exhausted

**Cost Limit:**\
Vertical scaling becomes exponentially expensive → Going from 32 cores to 64 cores might cost 3x more

## Solution: Sharding
**Benefits:**
- **Horizontal Scaling:** Add more servers instead of bigger servers
- **Better Performance:** Distribute load across multiple servers
- **Cost Effective:** Cheaper to add commodity servers
- **Geographic Distribution:** Data closer to users globally