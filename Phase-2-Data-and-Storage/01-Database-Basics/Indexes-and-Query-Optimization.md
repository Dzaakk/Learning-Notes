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

## Query Optimization Techniques

### 1. Analyze Query Execution Plan
**PostgreSQL**
```sql
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'john@example.com';
```

**MySQL**
```sql
EXPLAIN
SELECT * FROM users WHERE email = 'john@example.com';
```

**Look For:**
- **Seq Scan** (bad) -> needs index
- **Index Scan** (good) -> using index
- **Cost** estimates
- **Rows** examined
- **Execution time**

### 2. Select Only Needed Columns

**Bad:**
```sql
SELECT * FROM users WHERE city = 'Jakarta';
-- Fetches all columns, more I/O
```

**Good:**
```sql
SELECT id, name, email FROM users WHERE city = 'Jakarta';
-- Only needed columns, less I/O
```

**Impact:**
- Reduces network transfer
- Less memory usage
- Faster serialization
- Enables covering indexes

### 3. Avoid Functions in WHERE Clause

**Bad:**
```sql
-- Index on email cannot be used
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
```

**Good:**
```sql
-- Option 1: Store lowercase version
UPDATE users SET email_lower = LOWER(email);
CREATE INDEX idx_email_lower ON users(email_lower);

-- Option 2: Use functional index (PostgreSQL)
CREATE INDEX idx_email_lower ON users(LOWER(email));
```

**Other Common Mistakes:**
```sql
-- Bad: Function prevents index usage
WHERE YEAR(created_at) = 2025
WHERE price * quantity > 1000

-- Good: Rewrite without functions
WHERE created_at >= '2025-01-01' AND created_at < '2026-01-01'
WHERE price > 1000 / quantity
```

### 4. Use LIMIT for Pagination

**Bad:**
```sql
-- Fetches all records
SELECT * FROM orders ORDER BY created_at DESC;
```
**Good:**
```sql
-- Pagination
SELECT * FROM orders 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 0;
```
**Better (Cursor-Based Pagination):**
```sql
-- More efficient for large offsets
SELECT * FROM orders 
WHERE created_at < '2025-01-15T10:00:00'
ORDER BY created_at DESC 
LIMIT 20;
```

### 5. Avoid N+1 Query Problem

**Bad (N+1 Queries):**
```python
# 1 query to fetch users
users = db.query("SELECT * FROM users LIMIT 10")

# N queries to fetch orders for each user
for user in users:
    orders = db.query("SELECT * FROM orders WHERE user_id = ?", user.id)
```
**Good:**
```sql
SELECT users.*, orders.*
FROM users
LEFT JOIN orders ON users.id = orders.user_id
WHERE users.id IN (1, 2, 3, 4, 5);
```

**Alternative (Batch Query):**
```sql
-- 2 queries total instead of N+1
SELECT * FROM users LIMIT 10;
SELECT * FROM orders WHERE user_id IN (1, 2, 3, 4, 5);
```

### 6. Use Covering Index

**Concept:** Index contains ALL columns needed by query.

**Example:**
```sql
-- Query:
SELECT name, email FROM users WHERE city = 'Jakarta';

-- Covering index (includes all queried columns):
CREATE INDEX idx_city_name_email ON users(city, name, email);

-- Database doesn't need to access table data at all!
```

**Benefits:**
- No table lookup required
- Faster query execution
- Reduced I/O

### 7. JOIN Optimization

**Tips:**
- Index all JOIN columns
- JOIN on indexed columns (primary/foreign keys)
- **Smaller table** should be **on the left** (in most databases)
- Use INNER JOIN when possible (faster than OUTER JOIN)

**Example:**
```sql
-- Ensure indexes exist
CREATE INDEX idx_order_user_id ON orders(user_id);

-- Efficient JOIN
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id
WHERE users.city = 'Jakarta';
```

## Index Trade-offs

### Benefits✅
- Dramatically faster SELECT queries
- Faster filtering (WHERE clauses)
- Faster sorting (ORDER BY)
- Faster joins (JOIN conditions)
- Enforce uniqueness

### Costs❌
- Slower INSERT/UPDATE/DELETE (index must be updated)
- Extra storage space (indexes can be 10-50% of table size)
- Memory overhead (indexes cached in RAM)
- Maintenance overhead (rebuilding, analyzing)

## Indexing Best Practices

### DO's✅
1. **Index Foreign Keys**
    ```sql
    CREATE INDEX idx_order_user_id ON orders(user_id);

2. **Index WHERE Clause Columns**
    ```sql
    -- Frequently queried
   CREATE INDEX idx_status ON orders(status);
    ```

3. **Index JOIN Columns**
    ```sql
    CREATE INDEX idx_product_id ON order_items(product_id);
    ```

4. **Use Composite Indexes Wisely**
    ```sql
    -- For: WHERE status = 'pending' ORDER BY created_at
    CREATE INDEX idx_status_created ON orders(status, created_at);
    ```

5. **Monitor Index Usage**
    ```sql
    -- PostgreSQL: Find unused indexes
    SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
    ```

### Dont's❌

1. **Don't Index Low Cardinality Columns**
    ```sql
    -- Bad: Only 2-3 unique values
    CREATE INDEX idx_gender ON users(gender);
    ```

2. **Don't Over-Index**
    - Max 5-7 indexes per table (guideline)
    - Each index slows down writes

3. **Don't Index Small Tables**
    - Tables with < 1000 rows often don't benefit
    - Full table scan is fast enough

4. **Don't Forget to Drop Unused Indexes**
    ```sql
    DROP INDEX idx_rarely_used;
    ```

## System Design Considerations

### Read-Heavy System
- More indexes are acceptable
- Consider read replicas
- Cache frequent queries
- Use covering indexes

### Write-Heavy systems
- Minimize indexes
- Batch writes when possible
- Consider eventual consistency
- Use write-optimized databases (Cassandra)

### Large Tables (Millions of Rows)
- Partitioning + local indexes
- Archive old data
- Consider separate OLAP database
- Use summary tables for analytics

## Common Indexing Pattern

### Pattern 1: Time-Series Data
```sql
-- Partition by month + index on timestamp
CREATE INDEX idx_created_at ON events(created_at);

-- Query recent data
SELECT * FROM events 
WHERE created_at > NOW() - INTERVAL '7 days';
```

### Pattern 2: Multi-Tenant Applications
```sql
-- Always filter by tenant_id
CREATE INDEX idx_tenant_user ON users(tenant_id, id);

-- All queries include tenant
SELECT * FROM users WHERE tenant_id = 123 AND status = 'active';
```

### Pattern 3: Soft Deletes
```sql
-- Partial index for non-deleted records
CREATE INDEX idx_active_users ON users(id) WHERE deleted_at IS NULL;
```

## Real-World Example

### E-commerce Order Table

**Schema:**
```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id INT,
    status VARCHAR(50),
    total DECIMAL(10,2),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
```

**Indexes:**
```sql
-- User's orders
CREATE INDEX idx_user_created ON orders(user_id, created_at DESC);

-- Admin dashboard: filter by status
CREATE INDEX idx_status_created ON orders(status, created_at DESC);

-- Reports: date range queries
CREATE INDEX idx_created ON orders(created_at);

-- API: lookup by ID (already covered by PRIMARY KEY)
```

**Query Performance:**
```sql
-- Fast: Uses idx_user_created
SELECT * FROM orders 
WHERE user_id = 12345 
ORDER BY created_at DESC 
LIMIT 10;

-- Fast: Uses idx_status_created
SELECT * FROM orders 
WHERE status = 'pending' 
ORDER BY created_at DESC;

-- Fast: Uses idx_created
SELECT COUNT(*), SUM(total) 
FROM orders 
WHERE created_at >= '2025-01-01';
```

## Key Takeaways
1. **Index strategically:** Not every column needs an index
2. **Composite indexes:** Column order matters
3. **Monitor performance:** Use EXPLAIN to analyze queries
4. **Balance read/write:** More indexes = slower writes
5. **Covering indexes:** Include all queried columns when possible
6. **Avoid anti-patterns:** No functions in WHERE, no N+1 queries
7. **System design:** Consider workload (read vs write heavy)