SQL vs NoSQL
===

## SQL Databases (Relational)

### Core Characteristic
- **Structured data** in tables with fixed schema
- **Relationships** via foreign keys
- **ACID transactions** support
- **SQL language** for querying

### When to Use SQL
✅Data has celar, consistent structure\
✅Need complex transactions (e.g., money transfers)\
✅Frequent JOINs between entities\
✅Consistency is critical\
✅Query patterns are well-defined

### Common Use Cases
- Banking systems (transactional-critical)
- E-commerce orders & inventory
- User authentication & authorization
- Financial reporting
- CRM systems

### Popular SQL Databases
- **PostgreSQL** - Most versatile, advanced features
- **MySQL** - Simple, reliable, widely adopted
- **Oracle** - Enterprise-grade
- **SQL Server** - Microsoft ecosystem

### Pros & Cons
#### Advantages:
- Strong ACID guarantees
- Mature ecosystem & tooling
- Powerful query language
- Built-in data integrity (constraints, foreign key)
- Standardized SQL

#### Disadvantages:
- Vertical scaling (limited)
- Schema changes can be risky in production
- Difficult to scale horizontally
- Performance degrades with massive data

## NoSQL Databases (Non-Relational)

### Core Characteristic
- **Flexible or no schema**
- **Optimized for specific use cases**
- **Horizontal scalling** support
- **Trade-off**: consistency for availability/performance

### Types off NoSQL Databases

#### 1. Document Store

**Concept**: Store data as documents (JSON-like format)

**Best For:**
- Semi-structured or frequently changing schema
- Complex nested objects
- Varied query patterns within a single document

**Examples:** MongoDB, Couchbase, Firebase

**Use Cases:**
- Content management systems
- Product catalogs (e-commerce)
- User profiles with custom fields
- Mobile app backends

**Data Structure Example:**
```json
{
  "_id": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "addresses": [
    {
      "type": "home",
      "city": "Jakarta",
      "country": "Indonesia"
    }
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

#### 2. Key-Value Store

**Concept:** Simple mapping: key -> value

**Best For:**
- Ultra-low latency requirements
- Simple access patterns (get by key)
- Caching layer
- Session storage

**Examples:** Redis, DynamoDB, Memcached

**Use Cases:**
- Session management
- Real-time leaderboards
- Rate limiting
- Shopping cart
- Feature flags

**Data Structure Example:**
```json
Key: "session:abc123"
Value: {"userId": "user456", "loginTime": "2025-01-15T10:30:00Z"}
```

#### 3. Column-Family Store

**Concept:** Store data by column rather than by row

**Best For:**
- Write-heavy workloads
- Time-series data
- Need to scan specific columns from many rows
- Analytics queries

**Examples:** Cassandra, HBase, ScyllaDB

**Use Cases:**
- IoT sensor data
- Application logs
- Time-series metrics
- Event sourcing
- Messaging platforms

**Data Structure Example:**
```json
Row Key: "sensor_001"
Column Family: "temperature"
  - 2025-01-15T10:00:00: 25.5
  - 2025-01-15T10:01:00: 25.7
  - 2025-01-15T10:02:00: 25.6
```

#### 4. Graph Database

**Concept:** Store data as nodes and relationships

**Best For:**
- Data with many interconnections
- Need to traverse relationships
- Recommendations based on connections
- Network analysis

**Examples:** Neo4j, Amazon Neptune

**Use Cases:**
- Social networks (friend connections)
- Recommendation engines
- Fraud detection
- Knowledge graphs
- Supply chain tracking

**Data Structure Example:**
```
(User:John)-[:FRIENDS_WITH]->(User:Jane)
(User:John)-[:LIKES]->(Product:iPhone)
(User:Jane)-[:LIKES]->(Product:iPhone)
```

## SQL vs NoSQL Comparison
|Aspect|SQL|NoSQL|
|-|-|-|
|Schema|Rigid, predefined|Flexible, dynamic|
|Scaling|Vertical (scale up)|Horizontal (scale out)|
|Consistency|Strong (ACID)|Eventual (BASE)|
|Transactions|Multi-row ACID|Limited or none|
|Query Language|Standardized SQL|Database-specific|
|Data Integrity|Enforced by DB|Handled by application|
|Joins|Efficient, built-in|Difficult or impossible|
|Best For|Complex queries|High throughput, flexibility|

## Polyglot Persistence (Hybrid Approach)

Modern systems often use multiple database for different needs.

**Example Architecture**\
PostgreSQL (SQL) -> User accounts, orders, payments\
Redis (key-value) -> Session cache, rate limiting\
Elasticsearch -> Full-text search\
Cassandra (Column) -> Event logs, time-series data\
Neo4j (Graph) -> Friend recommendations

**Principle:** Use the right database for each specific use case.

## Decision Framework

### Choose SQL If:
- Data is structured with important relationships
- Need strong consistency
- Complex queries with JOINs
- ACID transactions are critical
- Data volume < 1TB (rough guideline)
- Team familiar with SQL

### Choose NoSQL If:
- Schema changes frequently
- Need horizontal scaling
- High write throughput
- Eventual consistency is acceptable
- Specific use case (cache, timeseries, graph, etc.)
- Data volume > 1TB and growing fast


## Real-World Examples

### Instagram
- **SQL (PostgreSQL):** User accounts, private messages
- **NoSQL (Cassandra):** Feed posts, likes, comments
- **NoSQL (Redis):** Feed cache, session data

### Netflix
- **SQL (MySQL):** User profiles, billing, subscriptions
- **NoSQL (Cassandra):** Viewing history, recommendations
- **NoSQL (DynamoDB):** Streaming metadata

### Uber
- **SQL (PostgreSQL):** Payments, trip records
- **NoSQL (Cassandra):** Location tracking
- **NoSQL (Redis):** Real-time driver locations

## Key Takeways
1. **No Silver Bullet:** Neither SQL nor NoSQL is universally better
2. **Start with SQL:** Unless you have a specific reason not to
3. **NoSQL for Scale:** When horizontal scaling is mandatory
4. **Polyglot is Common:** Most large systems use multiple database
5. **Consider Trade-offs:** Consistency vs Availability vs Partition Tolerance