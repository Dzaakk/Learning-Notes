ETL and Data Pipelines
===

# What is ETL?
**ETL = Extract, Transform, Load**

**Definition:** Process of moving data from source systems to destination systems with transformation in between

## Core Steps:
![etl core steps](./images/etl-core-steps.png)

# ETL vs ELT

## ETL (Traditional)
> Extract → Transform (outside) → Load

### Process:
- Pull data from source
- Transform in **staging server** (external compute)
- Load clean data to warehouse

### When to use:
- Warehouse compute is expensive
- Complex transformation needed
- Legacy systems
- On-premise infrastructure

### Pros:
✅ Clean data arrives in warehouse\
✅ Less warehouse compute needed\
✅ Works with limited warehouse resources

### Cons:
❌ Needed separate transformation infrastructure\
❌ Slower loading (transformation overhead)
❌ Less flexible (schema defined upfront)

## ELT (Modern Cloud)
> Extract → Load (raw) → Transform (in warehouse)

### Process:
- Pull data from source
- Load raw data directly to warehouse
- Transform using **warehouse compute power**

### When to use:
- Cloud warehouse (Redshift, BigQuery, Snowflake)
- Want leverage warehouse power
- Fast loading needed
- Schema flexibility required

### Pros:
✅ Fast loading (no transformation delay)\
✅ Leverage powerful warehouse compute\
✅ Raw data available for reprocessing\
✅ Schema flexibility required

### Cons:
❌ Higher warehouse compute costs\
❌ Raw data takes storage\
❌ Complex to manage transformations

## Comparison Table:
|Aspect|ETL|ELT|
|-|-|-|
|**Tranform Where**|Before load (staging)|After load(warehouse)
|**Loading Speed**|Slower|Faster
|**Warehouse Load**|Lower|Higher
|**Best for**|Limited resources|Cloud warehouses
|**Flexibility**|Lower|Higher|
|**Example**|On-premise → Warehouse|S3 → BigQuery

**Modern Trend:** Most cloud platforms use **ELT** (BigQuery, Snowflake, Databricks)

# Data Pipeline Patterns

## 1. Batch Processing
**Definition:** Process large volumes of data at scheduled intervals

### Characteristics:
- Run on schedule (hourly, daily, weekly)
- Process large volumes
- Higher latency (minutes to hours)
- More resource-efficient

### Example:
> Every day at 2 AM:
> 1. Extract orders from last 24 hours
> 2. Transform and aggregate
> 3. Load to warehouse
> 4. Refresh dashboards

### Use Cases:
- Daily/monthly reports
- Historical analysis
- Payroll processing
- End-of-day reconciliation

### Tools:
- Apache Airflow
- AWS Glue
- Azure Data Factory
- dbt

### Pros:
✅ Simple to implement\
✅ Cost-effective (batch resources)\
✅ Handle large volumes well\
✅ Retry logic easier

### Cons:
❌ Data not real-time (hours old)\
❌ Latency in insight\
❌ Fixed schedule

## 2. Streaming Processing (Real-Time)
**Definition:** Process data continuously as it arrives

### Characteristics:
- Continuous processing
- Low latency (seconds)
- Event-driven
- More complex

### Example:
> Continuous:\
> User clicks ad → Kafka → Transform → Real-time dashboard\
> (< 1 second latency)

### Use Cases:
- Fraud detection
- Real-time recommendations
- Stock trading
- IoT sensor monitoring
- Live dashboards

### Tools:
- Apache Kafka
- Apache Flink
- AWS Kinesis
- Spark Streaming

### Pros:
✅ Real-time insights (seconds)\
✅ Immediate actions\
✅ Always fresh data

### Cons:
❌ More complex\
❌ Higher infrastructure costs\
❌ Harder to debug\
❌ Exactly-once processing challenging

## 3. Micro-Batch (Hybrid) 
**Definition:** Small batches processed frequently (every few minutes)

### Characteristics:
- Balance between batch and streaming
- Near real-time (1-5 minutes latency)
- Simpler than pure streaming

### Example:
> Every 5 minutes:\
> Process last 5 minutes of events

### Use Cases:
- Near real-time dashboards
- Social media analytics
- E-commerce metrics

### Tools:
- Spark Streaming (DStreams)
- Apache  Flink (micro-batching mode)

### Pros:
✅ Near real-time (good enough for many cases)\ 
✅ Simpler than pure streaming\
✅ Lower cost than streaming

### Cons:
❌ Not true-real-time\
❌ Still has latency

## Pattern Comparison:
|Aspect|Batch|Micro-Batch|Streaming|
|-|-|-|-|
|Latency|Hours|Minutes|Seconds|
|Complexity|Low|Medium|high|
|Cost|Low|Medium|High|
|Use Case|Reports|Near real-time|Real-time|

### Rule of Thumb:
- Need same-day data? → Batch
- Need data withing 5 minutes? → Micro-batch
- Need data within seconds? → Streaming
