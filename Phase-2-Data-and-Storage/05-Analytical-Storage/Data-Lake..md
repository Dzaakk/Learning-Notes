Data Lake
===

# What is a Data Lake?
**Definition:** Centralized repository that stores raw, unprocessed data in its native format (structured, semi-structured, unstructured) at any scale.

## Key Characteristics:
- **Stores raw data** (no pre-processing)
- **Schema-on-read** (define schema when querying)
- **Flexible** (any format: JSON, CSV, Parquet, images, logs)
- **Scalable** (petabytes of data)
- **Cost-effective** (cheap storage like s3)
- **ELT approach** (Extract → Load → Transform)

## Architecture:
![data lake architecture](./images/data-lake-architecture.png)

# Data Lake Zones (Medallion Architecture)
**Pattern:** Progressively improve data quality through layers

## Bronze Layer (Raw)
**Purpose:** Landing zone for raw data

### Characteristics:
- Exact copy from source (immutable)
- Append-only
- Any format
- Minimal validation

**Example:** Raw JSON logs, CSV files as uploaded

**Storage:** JSON, CSV, Parquet

## Silver Layer (Cleansed)
**Purpose:** Cleaned and validate data

### Transformations:
- Remove duplicates
- Handle nulls
- Standardized formats (dates, phones)
- Data type fixes
- Basic quality checks

**Example:** Parsed logs, filtered invalid records

**Storage:** Parquet (compressed, columnar)

## Gold Layer (Curated)
**Purpose:** Business-ready data

### Transformations:
- Aggregations (daily, monthly summaries)
- Business metrics (KPIs, revenue)
- Feature engineering for ML
- Optimized for specific use cases

**Example:** Daily sales by region, customer 360 views

**Storage:** Parquet, Delta Lake, or Data Warehouse tables

## Data Flow Example:
```sql
-- Bronze: Raw data
{"user_id": "123", "event": "click", "timestamp": "2024-12-26T10:00:00Z"}

-- Silver: Cleaned
user_id | event | timestamp           | date
123     | click | 2024-12-26 10:00:00 | 2024-12-26

-- Gold: Aggregated
date       | total_clicks | unique_users
2024-12-26 | 1500000     | 50000
```

# Schema-on-Read vs Schema-on-Write

## Schema-on-Write (Data Warehouse)
![schema on write](./images/schema-on-write.png)
- Define schema before loading
- Slower ingestion (validation)
- Fast queries (optimized structure)

## Schema-on-Read (Data Lake)
![schema on read](./images/schema-on-read.png)
- Load data as-is
- Fast ingestion (validation)
- Flexible querying
- Slower queries (no optimization)
  
**Trade-off:** Flexibility vs Performance

# Popular Data Lake Solutions

## 1. Amazon S3 + AWS Glue
**Storage:** S3 (object storage)\
**Catalog:** AWS Glue (metadata)\
**Query:** Athena (SQL), EMR (Spark)

### Benefits:
- Cost-effective
- Scalable
- Integrates with AWS ecosystem

## 2. Azure Data Lake Storage (ADLS)
**Storage:** ADLS Gen2\
**Processing:** Azure Databricks, synapse\
**Catalog:** Azure Pureveiw

### Benefits:
- Hierarchical namespace
- Enterprise security
- Integrates with Azure services

## 3. Google Cloud Storage + BigQUery
**Storage:** GCS\
**Processing:** Dataproc (Spark), BigQuery\
**Catalog:** Data Catalog

## 4. Databricks Lakehouse
**Hybrid:** Combines Data Lake + Warehouse features\
**Format:** Delat Lake (ACID on data lakes)

### Features:
- ACID tarnsaction on s3/ADLS
- Time travel
- Schema evolution
- Unified batch and streaming

# Data Lake Challenges

## 1. Data Swamp Problem
**Issue:** Unorganized data becomes unusable

### Solutions:
- Implement metadata catalog
- Enforce naming conventions
- Zone-based organization (bronze/silver/gold)
- Data quality checks

## 2. Performance Issues
**Issue:** Querying raw data is slow

### Solutions:
- Partition data (by date, region, etc)
- Use columnar formats (Parquet, ORC)
- Create indexes/statistics
- Cache frequently accessed data

## 3. Security and Governance
**Issue:** Sensitive data exposed

### Solutions:
- Fine-grained access control
- Encryption at rest and in transit
- Data masking
- Audit logs

# When to Use a Data Lake
- Need to store diverse data types
- Don't know future use cases
- Machine learning/ data science
- Real-time/streaming data
- Need cheap storage
- Exploratory analysis

## Examples:
- IoT sensor data
- Application logs
- Social media feeds
- Image/video storage
- ML training data


