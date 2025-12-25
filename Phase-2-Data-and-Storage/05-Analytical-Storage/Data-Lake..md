Data Lake
===

# What is a Data Lake?
**Definition:** Centralized repository that stores raw, unprocessed data in its native format (structured, semi-structured, unstructured) at any scale.

## Key Characteristics:
- **Stores raw data** (no pre-processing)
- **Schema-on-read** (define schema when querying)
- **Flexible** (any format: JSON, CSV, images, logs)
- **Scalable** (petabytes of data)
- **Cost-effective** (cheap storage like s3)

## Architecture:
![data lake architecture](./images/data-lake-architecture.png)

# Data Lake Zones (Medallion Architecture)

## Bronze Layer (Raw)
- Raw, unprocessed data
- Exact copy from source
- Append-only
- Immutable

**Example:** Raw JSON logs, CSV files as uploaded

## Silver Layer (Cleansed)
- Cleaned and validated
- Standardized formats
- Deduplicated
- Basic transformations

**Example:** Parsed logs, filtered invalid records

## Gold Layer (Curated)
- Business-level aggregates
- Optimized for analytics
- Feature-engineered for ML
- Ready for consumption

**Example:** Daily sales aggregates, customer 360 views

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

