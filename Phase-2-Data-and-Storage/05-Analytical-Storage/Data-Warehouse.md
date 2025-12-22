Data Warehouse
===

# What is a Data Warehouse?
**Definition:** Centralized repository that stores structured, processed data from multiple sources optimized for analysis and reporting

## Key Characteristics:
- **Structured Data** (schemas enforced)
- **ETL processed** (cleaned, transformed)
- **Optimized for queries** (fast aggregations)
- **Historical data** (time-series)
- **Single source of truth** for analytics

## Architecture Flow:
![data warehouse flow](./images/data-warehouse-flow.png)

# Data Warehouse Architecture Layers

## 1. Staging Layer
**Purpose:** Temporary landing zone fro raw data from source systems

### Characteristics:
- Data stored in original source format
- Exact copy of source data
- Minimal transformation
- Used for data quality checks
- Typically truncated after ETL completes

### Example:
```sql
-- Staging table
CREATE TABLE staging_orders (
    order_id INT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL,
    -- Loaded as-is from source
);
```

## 2. Integratiion/ODS Layer (Optional)
**ODS = Operational Data Store**

**Purpose:** Support operational reporting with current or near real-time data

### Characteristics:
- Time-sensitive business data for current operations
- Subject-oriented (by business area)
- Support tactical decisions
- Usually normalized (3NF schema)
- Bridge between OLTP and warehouse

**Use Case:** "What happened in the last hour?" - operational queries

### Difference from Warehouse:
- ODS: Current/recent data, data overwritten and changes frequently
- Warehouse: Historical data, static data for archiving and historical analysis

## 3. Data Warehouse Layer
**Purpose:** Historical, integrated data optimized for analysis

### Characteristics:
- Star/Snowflake schema
- Slowly Changing Dimensions (SCD)
- Time-variant data
- Non-volatile (historical records preserved)

## 4. Data Mart Layer
**Purpose:** Subset of warehouse focused on specific business function or department

### Example:
![data mart layer](./images/data-mart-layer.png)

### Benefits:
- Faster queries on smaller datasets optimized for specific needs
- Department-specific schemas
- Security/access control by department
- Cost-effective with less storage and computational power

# Data Warehouse Schema Design

## 1. Star Schema (Most Common)
**Structure:** One fact table surrounded by dimension tables

### Example: E-commerce Analytics
![star schema example](./images/star-example.png)

**Pros:**\
✅Simple queries (fewer joins)\
✅Fast query performance\
✅Easy to understand

**Cons:**\
❌Data redundancy in dimensions\
❌More storage needed

**Use Case:** Most business intelligence applications

## 2. Snowflake Schema
**Structure:** Normalized dimension tables (dimensions split into sub-dimensions)

### Example:
![snowflake example](./images/snowflake-example.png)

**Pros:**\
✅Less storage (normalized)\
✅Better data integrity

**Cons:**\
❌More complex queries (more joins)\
❌Slower query performance

**Use Case:** When storage is expensive, complex hierarchies

## 3. Galaxy Schema (Fact Constellation)
**Structure:** Multiple fact tables sharing dimension tables

### Example:
![galaxy example](./images/galaxy-example.png)

**Use Case:** Enterprise data warehouse with multiple business process

# Column-Oriented Storage
**Why Column Storage for Analytics?**

## Row-Oriented (OLTP):
>Row 1: [id=1, name="John", age=25, city="NYC", salary=50000]
>
>Row 2: [id=2, name="Jane", age=30, city="LA", salary=60000]
>
>Row 3: [id=3, name="Bob", age=35, city="NYC", salary=70000]

## Column-Oriented (OLAP):
>Column id:     [1, 2, 3]\
>Column name:   ["John", "Jane", "Bob"]\
>Column age:    [25, 30, 35]\
>Column city:   ["NYC", "LA", "NYC"]\
>Column salary: [50000, 60000, 70000]

## Benefits for Analytics:
✅**Better compression** (similar data together)\
✅**Faster aggregations (only read needed columns)\
✅**Efficient for queries** like: `SELECT AVG(salary) WHERE city='NYC'`

```sql
-- Only reads 'salary' and 'city' columns
SELECT AVG(salary) FROM employees WHERE city = 'NYC';

-- Row storage: reads entire rows (wasteful)
-- Column storage: reads only 2 columns (efficient)
```

# Popular Data Warehouse Solutions

## 1. Amazon Redshift
**Type:** Cloud-based, column-oriented

### Features:
- Massively parallel processing (MPP)
- SQL interface
- Integrates with AWS ecosystem
- Scalable (resize clusters)

**Pricing:** Pay per hour of cluster runtime

**Use Case:** AWS-heavy companies, medium to large scale

## 2. Google BigQuery
**Type:** Serverless, column-oriented

### Features:
- Serverless (no infrastructure management)
- Extremely fast (petabyte-scale)
- Pay per query (no idle costs)
- Integrates with GCP

**Pricing:** Pay per data scanned

**Use Case:** Startups to enterprise, variable workloads

## 3. Snowflake
**Type:** Cloud-agnostic, column-oriented

### Features:
- Multi-cloud (AWS, Azure, GCP)
- Separate storage and compute
- Automatic scaling
- Time trave (query historical data)

**Pricing:** Storage + compute separately

**Use Case:** Multi-cloud strategy, financial services

## 4. Apache Hive (Open Source)
**Type:** SQL on Hadoop

### Features:
- Built on Hadoop ecosystem
- HiveQL (SQL-like)
- Good for batch processing

**Use Case:** Large datasets, cost-sensitive, on-premise

# When to Use Data Warehouse

## Use Data Warehouse When:
✅Need fast, complex analytical queries\
✅Business intelligence and reporting\
✅Structured, cleaned data\
✅SQL-based analysis\
✅Consistent schema\
✅Regulatory compliance (audit trails)

## Examples:
- Sales dashboards
- Financial reporting
- Customer segmentation
- Trend analysis
- KPI tracking