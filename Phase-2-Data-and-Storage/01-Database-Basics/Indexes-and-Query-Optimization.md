Indexes and Query Optimization
===

## What is an Index?
**Analogy**: Like an index in a book - instead of reading every page, you jump dfirectly to relevant pages.

**Without Index (Full Table Scan)**
```sql
SELECT * FROM users WHERE email = 'john@example.com';
→ Scans ALL rows: O(n) complexity
→ Slow for large tables
```

**With Index**
```sql
→ Uses index lookup: O(log n) complexity
→ Much faster, especially for millions of rows
```

## Why Indexes Matter in System Design

### Performance Impact
- **Without Index:** 10,000 rows = 10,000 disk reads
- **With Index:** 10,000 rows = ~14 disk reads (log₂ 10,000)

### System Design Implication
- Reduce database load -> fewer servers needed
- Improves response time -> better user experience
- Enables caching strategies -> predictable query times
- Affects scalling strategy -> vertical vs horizontal

## Types of Indexes

### 1. Primary Index (Clustered Index)
**Characteristic:**
- Automatically created for PRIMARY KEY
- Physical data on disk is sorted by this index
- Only ONE per table
- Fastest for range queries

**Example:**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,  -- Automatically creates clustered index
    email VARCHAR(255),
    name VARCHAR(255)
);
```
**When to Use:**
- Already created automatically for primary keys
- Choose primary key that's frequently queried

### 2. Secondary Index (Non-Clustered Index)
**Characteristic:**
- Additional indexes on non-primary columns
- Stores pointers to actual data
- Can have MANY per table

**Example:**
```sql
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_name ON users(name);
```
**When to Use:**
- Columns frequently used in WHERE clauses
- Columns used in JOIN conditions
- Columns used in ORDER BY

### Composite Index (Multi-Column Index)
**Characteristic:**
- Index on multiple columns
- **Column order matters!**
- Can satisfy queries on leading columns

**Example:**
```sql
-- Index on (city, age)
CREATE INDEX idx_city_age ON users(city, age);
```

**Query Performance:**
```sql
-- ✅ EFFICIENT: Uses index
SELECT * FROM users WHERE city = 'Jakarta';
SELECT * FROM users WHERE city = 'Jakarta' AND age > 25;

-- ❌ INEFFICIENT: Cannot use index (age is not leading column)
SELECT * FROM users WHERE age > 25;
```

**Best Practice for Column Order:**
1. High selectivity columns first (more unique values)
2. Equality conditions before range conditions
3. Most frequently queried columns first

**Example:**
```sql
-- Good: city is more selective than age
CREATE INDEX idx_city_age ON users(city, age);

-- For query: WHERE city = 'Jakarta' AND age > 25 ORDER BY created_at
CREATE INDEX idx_city_age_created ON users(city, age, created_at);
```

### 4. Unique Index
**Characteristic:**
- Enforces uniqueness
- Also speeds up lookups
- Prevents duplicate values

**Example:**
```sql
CREATE UNIQUE INDEX idx_unique_email ON users(email);
-- Now duplicate emails are prevented
```

**When to Use:**
- Email addresses
- Usernames
- Social security numbers
- Any field requiring uniqueness

### 5. Full-Text Index
**Characteristic:**
- Specialized for text search
- Supports natural language queries
- Available in MySQL, PostgreSQL, etc.

**Example:**
```sql
CREATE FULLTEXT INDEX idx_content ON articles(content);

SELECT * FROM articles 
WHERE MATCH(content) AGAINST('system design patterns');
```

**When to Use:**
- Blog search
- Product search
- Document search
- Consider Elasticsearch for advanced needs

### 6. Partial Index (PostgreSQL)
**Characteristic:**
- Index only a subset of rows
- Saves space and improves performance

**Example:**
```sql
-- Only index active users
CREATE INDEX idx_active_users ON users(created_at) 
WHERE is_active = true;
```

**When to Use:**
- Soft deletes (only index non-deleted records)
- Status-based filtering (only active, pending, etc.)
- Time-based data (only recent records)

## Index Data Structures

### B-Tree (Defaul for Most Databases)
**Characteristic:**
- Balanced tree structure
- O(log n) complexity
- Good for equality and range queries

**When to Use:**
- Primary keys
- Foreign keys
- High cardinality columns (many unique values)
- Range queries (>, <, BETWEEN, ORDER BY)
         
### Hash Index
**Characteristic:**
- Key -> Hash Function -> Bucket
- O(1) lookup for exact matches
- **Cannot** support range queries
- Not default in most databases

**When to Use:**
- Exact match lookups only
- High-speed key lookups
- In-memory database (redis)

**Example:**
```sql
-- PostgreSQL
CREATE INDEX idx_hash_email ON users USING HASH (email);
```

### Bitmap Index
**Characteristic:**
- Bit array for low cardinality columns
- Space-efficient for limited values
- Common in data warehouses

**When to Use:**
- Boolean fields (is_active, is_premium)
- Gender (male/female/other)
- Status enums (pending/approved/rejected)
- Data warehouses with read-heavy workloads

**Not Recommended For:**
- High cardinality columns
- Frequent updates (expensive to maintain)