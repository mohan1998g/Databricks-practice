# DLT in Databricks — Detailed Guide

> **Important terminology:** Databricks has renamed **Delta Live Tables (DLT)** to **Lakeflow Declarative Pipelines**. You will still see `dlt` in older code, documentation, projects, and interviews, so it is important to understand both names.

---

# 1. What is DLT?

**DLT (Delta Live Tables)** was a Databricks framework for building reliable data pipelines using a **declarative approach**.

Instead of writing code that explicitly says:

> Read this table → transform it → write it → create another table → update it

you describe:

> This is my source, this is my transformation, and this is the resulting table.

Databricks then determines the appropriate execution plan.

The newer name is **Lakeflow Declarative Pipelines**, but the fundamental concepts you encounter in existing DLT implementations include:

* Streaming tables
* Materialized views
* Expectations
* Pipeline DAG
* Incremental processing
* Data quality
* Dependencies
* Checkpoints
* Event logs
* Auto Loader integration
* CDC processing
* Medallion architecture

---

# 2. Why was DLT introduced?

Before DLT, you could build a pipeline manually:

```text
Source
  ↓
Spark Job
  ↓
Bronze
  ↓
Spark Job
  ↓
Silver
  ↓
Spark Job
  ↓
Gold
```

You had to manage many things yourself:

* Table creation
* Dependencies
* Data quality
* Incremental processing
* Checkpointing
* Error handling
* Pipeline monitoring
* Schema handling
* Recovery

DLT provides a framework to manage many of these concerns declaratively.

---

# 3. Traditional Spark vs DLT

## Traditional Spark

You might write:

```python
df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .load("/data/customers")

df.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/checkpoint/customers") \
    .start("/tables/customers")
```

You explicitly manage:

* Read
* Write
* Checkpoint
* Output path
* Streaming query

---

## DLT

You describe the resulting dataset:

```python
@dlt.table
def customers():
    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("/data/customers")
    )
```

DLT manages much of the pipeline execution infrastructure.

---

# 4. Declarative vs Imperative

This is one of the **most important DLT interview concepts**.

## Imperative programming

You tell Spark **how to do something**.

```python
df = spark.read(...)
df = df.filter(...)
df.write(...)
```

You are explicitly controlling the sequence of operations.

---

## Declarative programming

You tell the system **what the desired result should be**.

```python
@dlt.table
def silver_customer():
    return dlt.read("bronze_customer").filter(...)
```

You are describing the desired dataset and its dependency.

DLT determines how the pipeline should execute.

---

# 5. DLT Pipeline Architecture

A typical architecture is:

```text
                SOURCE
                  │
                  ▼
              BRONZE
                  │
                  ▼
              SILVER
                  │
                  ▼
               GOLD
```

For example:

```text
CSV / JSON / Kafka / S3
          │
          ▼
   Bronze Streaming Table
          │
          ▼
   Silver Streaming Table
          │
          ▼
   Gold Materialized View
```

---

# 6. DLT and Medallion Architecture

DLT works extremely well with:

```text
Bronze
  ↓
Silver
  ↓
Gold
```

### Bronze

Raw data.

```text
S3
 ↓
Bronze
```

### Silver

Cleaned and transformed data.

```text
Bronze
 ↓
filter
deduplicate
join
standardize
 ↓
Silver
```

### Gold

Business-level aggregates.

```text
Silver
 ↓
aggregation
 ↓
Gold
```

---

# 7. Important DLT terminology

You should know these terms:

| Term                           | Meaning                                    |
| ------------------------------ | ------------------------------------------ |
| DLT                            | Delta Live Tables                          |
| Lakeflow Declarative Pipelines | New name for DLT                           |
| Pipeline                       | Collection of related data transformations |
| Dataset                        | Table/view produced by pipeline            |
| Streaming table                | Incrementally processed table              |
| Materialized view              | Persisted result of a query                |
| Expectation                    | Data-quality rule                          |
| DAG                            | Dependency graph                           |
| Event log                      | Pipeline execution/monitoring information  |
| CDC                            | Change Data Capture                        |
| Auto Loader                    | Incremental file ingestion                 |
| Checkpoint                     | Streaming progress/state                   |
| Update                         | Execution of a pipeline                    |

---

# 8. DLT Dataset Types

Historically, DLT commonly used:

1. **Live Tables / Streaming Live Tables**
2. **Materialized Views**
3. Views

With newer Lakeflow terminology, you will encounter:

* Streaming tables
* Materialized views
* Views

The exact APIs and supported syntax have evolved, so don't assume every old `dlt` example is the preferred syntax for a new project.

---

# 9. Streaming Table

A streaming table processes data incrementally.

Example:

```python
import dlt

@dlt.table
def bronze_customers():

    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("/mnt/raw/customers")
    )
```

Conceptually:

```text
New files
   ↓
Read only new data
   ↓
Bronze table
```

---

# 10. Why use Streaming Tables?

Suppose S3 contains:

```text
customers/
    file1.json
    file2.json
    file3.json
```

First pipeline execution:

```text
file1
file2
file3
 ↓
Bronze
```

Later:

```text
file4.json
file5.json
```

The streaming pipeline can process the newly available data incrementally.

You don't necessarily need to reread all historical files.

---

# 11. Materialized View

A materialized view stores the result of a query and maintains it as the underlying data changes.

Example:

```python
@dlt.table
def customer_summary():

    return (
        dlt.read("silver_customers")
        .groupBy("country")
        .count()
    )
```

Conceptually:

```text
Silver
  ↓
GROUP BY country
  ↓
Materialized result
```

---

# 12. Streaming Table vs Materialized View

This distinction is extremely important.

| Feature                   | Streaming Table | Materialized View   |
| ------------------------- | --------------- | ------------------- |
| Incremental ingestion     | Excellent       | Depends on query    |
| Streaming source          | Yes             | Not necessarily     |
| Represents current result | Not necessarily | Yes                 |
| Good for ingestion        | Yes             | Usually no          |
| Aggregations              | Possible        | Excellent use case  |
| Continuous processing     | Suitable        | Depends on pipeline |
| Typical layer             | Bronze/Silver   | Silver/Gold         |

---

# 13. DLT Views

A view is generally used when you don't need to persist the result as a table.

Example conceptually:

```python
@dlt.view
def valid_customers():

    return dlt.read("bronze_customers") \
        .filter("age >= 18")
```

Think:

```text
Table
 ↓
View
 ↓
Query result
```

rather than:

```text
Table
 ↓
Physical persisted table
```

---

# 14. `dlt.read()`

Example:

```python
df = dlt.read("bronze_customers")
```

This reads another dataset defined within the DLT pipeline.

Example:

```python
@dlt.table
def silver_customers():

    return (
        dlt.read("bronze_customers")
        .filter("customer_id IS NOT NULL")
    )
```

Dependency:

```text
bronze_customers
       ↓
silver_customers
```

---

# 15. `dlt.read_stream()`

When you want streaming semantics:

```python
df = dlt.read_stream("bronze_customers")
```

Example:

```python
@dlt.table
def silver_customers():

    return (
        dlt.read_stream("bronze_customers")
        .filter("customer_id IS NOT NULL")
    )
```

Conceptually:

```text
Streaming Bronze
       ↓
Streaming Silver
```

---

# 16. DLT Automatically Builds a DAG

Suppose you define:

```python
@dlt.table
def bronze():
    return source_df
```

Then:

```python
@dlt.table
def silver():
    return dlt.read("bronze").filter(...)
```

Then:

```python
@dlt.table
def gold():
    return (
        dlt.read("silver")
        .groupBy("country")
        .count()
    )
```

DLT understands:

```text
bronze
  ↓
silver
  ↓
gold
```

This is called a **DAG — Directed Acyclic Graph**.

---

# 17. Why is the DAG important?

DLT knows:

```text
Gold depends on Silver
Silver depends on Bronze
```

Therefore:

```text
Bronze
   ↓
Silver
   ↓
Gold
```

rather than trying to execute everything randomly.

---

# 18. DLT Expectations

This is one of the biggest reasons DLT became popular.

An **expectation** is a data-quality rule.

Example:

```python
@dlt.expect(
    "valid_customer_id",
    "customer_id IS NOT NULL"
)
@dlt.table
def silver_customers():

    return dlt.read("bronze_customers")
```

Meaning:

> Every record should have a customer ID.

---

# 19. Expectations vs WHERE filter

This distinction is important.

Suppose:

```python
.filter("customer_id IS NOT NULL")
```

This removes bad records.

But an expectation:

```python
@dlt.expect(
    "valid_customer_id",
    "customer_id IS NOT NULL"
)
```

is primarily a **data-quality rule** that can be monitored and acted upon according to its configured behavior.

---

# 20. Expectation — Warn

Example:

```python
@dlt.expect(
    "valid_customer_id",
    "customer_id IS NOT NULL"
)
```

Conceptually:

```text
Bad record
    ↓
Pipeline continues
    ↓
Expectation violation recorded
```

Use when:

> Bad records shouldn't stop the pipeline, but you want visibility.

---

# 21. Expectation — Drop

Historically:

```python
@dlt.expect_or_drop(
    "valid_customer_id",
    "customer_id IS NOT NULL"
)
```

Conceptually:

```text
              ┌── Valid ──> Continue
Input records ┤
              └── Invalid ──> Drop
```

Use when invalid records should not enter the target dataset.

---

# 22. Expectation — Fail

Historically:

```python
@dlt.expect_or_fail(
    "valid_customer_id",
    "customer_id IS NOT NULL"
)
```

Conceptually:

```text
Invalid record
      ↓
Expectation failure
      ↓
Pipeline fails
```

Use when violating the rule means:

> The dataset is unacceptable and downstream processing should not continue.

---

# 23. Three expectation behaviors

Remember:

```text
EXPECT
   ↓
Warn / observe

EXPECT_OR_DROP
   ↓
Remove bad records

EXPECT_OR_FAIL
   ↓
Stop/fail pipeline
```

This is a very common interview question.

---

# 24. Multiple expectations

Example:

```python
@dlt.expect_all({
    "valid_customer_id": "customer_id IS NOT NULL",
    "valid_age": "age >= 0",
    "valid_country": "country IS NOT NULL"
})
@dlt.table
def silver_customers():

    return dlt.read("bronze_customers")
```

You can define multiple data-quality rules.

---

# 25. `expect_all_or_drop`

Conceptually:

```python
@dlt.expect_all_or_drop({
    "valid_customer_id": "customer_id IS NOT NULL",
    "valid_age": "age >= 0"
})
```

Invalid rows are removed.

---

# 26. Data-quality example

Suppose Bronze:

| customer_id | name  | age |
| ----------: | ----- | --: |
|         101 | Ravi  |  25 |
|         102 | Mohan |  30 |
|        NULL | Kumar |  20 |
|         103 | Raj   |  -5 |

Expectation:

```python
@dlt.expect_all_or_drop({
    "valid_customer": "customer_id IS NOT NULL",
    "valid_age": "age >= 0"
})
```

Silver:

| customer_id | name  | age |
| ----------: | ----- | --: |
|         101 | Ravi  |  25 |
|         102 | Mohan |  30 |

Invalid records are removed.

---

# 27. Why DLT Expectations are powerful

Without expectations:

```text
Bad data
 ↓
Silver
 ↓
Gold
 ↓
Reports
```

With expectations:

```text
Raw
 ↓
Quality rules
 ↓
Silver
 ↓
Gold
```

Data quality becomes part of the pipeline rather than being a separate afterthought.

---

# 28. Auto Loader + DLT

This is one of the most common real-world combinations.

Architecture:

```text
S3 / ADLS / GCS
       │
       ▼
   Auto Loader
       │
       ▼
DLT Streaming Table
       │
       ▼
     Bronze
```

Example:

```python
@dlt.table
def bronze_flights():

    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("/Volumes/workspace/raw/flights")
    )
```

---

# 29. DLT + Bronze/Silver/Gold

Example:

```python
@dlt.table
def bronze_flights():

    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .option("header", "true")
        .load("/Volumes/workspace/raw/flights")
    )
```

Silver:

```python
@dlt.table
def silver_flights():

    return (
        dlt.read_stream("bronze_flights")
        .filter("flight_id IS NOT NULL")
    )
```

Gold:

```python
@dlt.table
def gold_flight_count():

    return (
        dlt.read("silver_flights")
        .groupBy("airline")
        .count()
    )
```

Architecture:

```text
Files
 │
 ▼
Bronze
 │
 ▼
Silver
 │
 ▼
Gold
```

---

# 30. DLT and CDC

DLT can be used for **Change Data Capture** pipelines.

Suppose your source sends:

```text
INSERT
UPDATE
DELETE
```

Example:

```text
Customer 101 → INSERT
Customer 102 → UPDATE
Customer 103 → DELETE
```

You need those changes reflected in your Delta target.

DLT/Lakeflow Declarative Pipelines provides CDC-oriented capabilities for this type of workflow.

---

# 31. Why CDC is difficult without a framework

Suppose source sends:

```text
id | name | operation
1  | Ravi | INSERT
2  | Mohan| UPDATE
3  | Raj  | DELETE
```

You need to:

```text
INSERT → Insert
UPDATE → Update
DELETE → Delete
```

Doing this manually can require:

* MERGE
* Ordering
* Deduplication
* Handling out-of-order changes
* Delete handling
* Schema evolution

Declarative CDC features simplify this.

---

# 32. APPLY CHANGES

Older DLT implementations frequently use:

```python
dlt.apply_changes(...)
```

The conceptual idea is:

```text
CDC source
    ↓
Apply changes
    ↓
Target table
```

For example:

```python
dlt.apply_changes(
    target="customers",
    source="customer_cdc",
    keys=["customer_id"],
    sequence_by="timestamp"
)
```

Conceptually:

```text
customer_id
     ↓
Identify same entity

timestamp
     ↓
Determine latest change
```

---

# 33. Why `sequence_by` is important

Suppose:

```text
customer_id | name  | timestamp
101         | Ravi  | 10:00
101         | Mohan | 10:05
101         | Raj   | 10:03
```

The records arrived in an inconvenient order.

If timestamp is the sequence:

```text
10:00
10:03
10:05
```

The latest change is:

```text
Raj?  No
Mohan? Yes
```

Therefore the final value is:

```text
101 | Mohan
```

---

# 34. CDC architecture

```text
Source Database
      │
      ▼
   CDC Stream
      │
      ▼
Bronze CDC
      │
      ▼
Apply Changes
      │
      ▼
Silver Current State
      │
      ▼
Gold
```

---

# 35. DLT and Delta Lake

DLT is closely integrated with Delta Lake.

Typical architecture:

```text
             DLT
              │
      ┌───────┼───────┐
      ▼       ▼       ▼
   Bronze   Silver   Gold
      │       │       │
      └──── Delta Lake ────┘
```

Delta Lake provides:

* ACID transactions
* Schema enforcement
* Schema evolution capabilities
* Time travel
* Versioning
* Reliable writes

DLT provides pipeline-level capabilities around:

* Dependencies
* Data quality
* Incremental processing
* Pipeline management
* Monitoring
* Declarative transformations

---

# 36. DLT vs Delta Lake

Don't confuse them.

### Delta Lake

A **storage/table format**.

```text
Parquet + Delta transaction log
```

### DLT

A **pipeline framework**.

```text
Source
 ↓
Transformations
 ↓
Delta tables
```

Therefore:

```text
DLT
 ↓
uses
 ↓
Delta Lake
```

---

# 37. DLT vs Databricks Job

Another common interview question.

## Job

Primarily orchestration.

```text
Task 1
 ↓
Task 2
 ↓
Task 3
```

## DLT

Primarily declarative data pipeline processing.

```text
Bronze
 ↓
Silver
 ↓
Gold
```

You can combine them:

```text
Job
 ↓
Pipeline task
 ↓
DLT/Lakeflow pipeline
```

---

# 38. DLT vs normal Spark Structured Streaming

## Structured Streaming

You manage things such as:

```python
readStream
writeStream
checkpointLocation
trigger
outputMode
```

Example:

```python
df.writeStream \
    .format("delta") \
    .outputMode("append") \
    .option("checkpointLocation", checkpoint) \
    .start(path)
```

## DLT

You describe the dataset:

```python
@dlt.table
def bronze():
    return spark.readStream(...)
```

DLT handles much of the pipeline infrastructure.

---

# 39. Your earlier checkpoint question and DLT

You previously had:

```python
df.writeStream \
    .format("delta") \
    .outputMode("append") \
    .trigger(once=True) \
    .option("checkpointLocation", checkpoint_path) \
    .option("path", "...") \
    .start()
```

In normal Structured Streaming, you explicitly specify:

```text
checkpointLocation
```

In a DLT/Lakeflow pipeline, you generally **do not manually manage a checkpoint path for each table in the same way**.

The pipeline manages its execution state/checkpointing infrastructure.

This is one of the practical differences between:

```text
Normal Structured Streaming
```

and:

```text
DLT/Lakeflow Declarative Pipeline
```

---

# 40. DLT Pipeline Modes

Historically, DLT pipelines commonly had:

### Triggered

```text
Start pipeline
      ↓
Process available data
      ↓
Finish
```

### Continuous

```text
Pipeline starts
      ↓
Continuously process data
      ↓
Keep running
```

---

# 41. Triggered mode

Use when:

```text
Hourly ingestion
Daily ingestion
Batch-like incremental processing
```

Example:

```text
2 AM
 ↓
Pipeline
 ↓
Finish
```

You don't need a continuously running cluster.

---

# 42. Continuous mode

Use when:

```text
Near-real-time processing
```

is required.

Example:

```text
Kafka
 ↓
DLT
 ↓
Silver
 ↓
Gold
```

The pipeline continuously processes arriving data.

---

# 43. Don't automatically choose continuous

If your business requirement is:

> Process files every hour.

Don't necessarily choose continuous processing.

You might use:

```text
Triggered pipeline
+
Hourly schedule
```

instead.

This can reduce unnecessary resource usage.

---

# 44. DLT Pipeline Configuration

A pipeline has configuration settings controlling things such as:

* Pipeline mode
* Compute
* Catalog
* Schema/target
* Source code
* Notifications
* Development mode
* Serverless
* Pipeline storage
* Configuration parameters

Exact options vary with the current Databricks/Lakeflow pipeline type and workspace.

---

# 45. DLT Pipeline UI

Conceptually, you will see:

```text
Pipeline
│
├── Graph
│
├── Tables
│
├── Data quality
│
├── Updates
│
├── Event log
│
└── Settings
```

The **graph** is particularly useful.

Example:

```text
                 Bronze
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Silver A            Silver B
          │                   │
          └─────────┬─────────┘
                    ▼
                  Gold
```

---

# 46. Event Log

DLT provides pipeline execution/event information.

You can use it to understand:

```text
Pipeline started
 ↓
Bronze processed
 ↓
Silver processed
 ↓
Expectation violations
 ↓
Gold completed
```

This is very useful for:

* Monitoring
* Troubleshooting
* Data quality
* Operational reporting

---

# 47. Data Quality Monitoring

Suppose you have:

```python
@dlt.expect(
    "valid_email",
    "email IS NOT NULL"
)
```

The pipeline can track expectation results.

You can determine:

```text
Total records = 1,000,000
Valid records = 995,000
Invalid records = 5,000
```

This makes data quality measurable.

---

# 48. DLT Error Handling

Suppose:

```text
Bronze
 ↓
Silver
 ↓
Gold
```

and Silver fails.

The pipeline understands the dependency graph.

```text
Bronze → SUCCESS
Silver → FAILED
Gold   → NOT EXECUTED
```

You don't have to manually construct the entire dependency orchestration.

---

# 49. Pipeline Recovery

DLT/Lakeflow pipelines use managed state and incremental processing mechanisms.

If a pipeline update fails, the framework can retry/reprocess according to the pipeline's execution semantics rather than requiring you to manually reconstruct every stage.

This is another advantage over building every streaming pipeline entirely yourself.

---

# 50. Schema Evolution

Real-world source data changes.

Today:

```text
id
name
age
```

Tomorrow:

```text
id
name
age
email
```

Your ingestion pipeline needs to handle schema changes appropriately.

DLT works with Databricks' schema evolution mechanisms, but **you should not assume every schema change is automatically safe**.

For example:

```text
Adding a column
```

is very different from:

```text
Changing DOUBLE → STRING
```

The latter can cause incompatibility and may require an explicit migration strategy.

---

# 51. DLT and Unity Catalog

Modern Databricks pipelines commonly use Unity Catalog.

Architecture:

```text
Unity Catalog
      │
      ▼
   Catalog
      │
      ▼
    Schema
      │
      ▼
    Tables
```

Example:

```text
prod
 │
 ├── bronze
 │     └── flights
 │
 ├── silver
 │     └── flights
 │
 └── gold
       └── flight_summary
```

This gives centralized governance.

---

# 52. DLT and Volumes

You can ingest files from Unity Catalog Volumes.

For example:

```text
/Volumes/workspace/raw/raw_flight_data/
```

can be used as a file source.

Example:

```python
spark.readStream \
    .format("cloudFiles") \
    .load("/Volumes/workspace/raw/raw_flight_data/")
```

---

# 53. DLT and Auto Loader

This combination is especially useful for cloud file ingestion.

```text
S3 / ADLS / GCS / Volume
          ↓
      Auto Loader
          ↓
       DLT Bronze
```

Auto Loader detects newly arriving files incrementally.

DLT manages the pipeline around the ingestion and transformations.

---

# 54. Complete PySpark example

Imagine:

```text
/Volumes/workspace/raw/flights/
```

contains JSON:

```json
{
  "flight_id": "F101",
  "airline": "ABC",
  "price": 500,
  "country": "India"
}
```

Bronze:

```python
import dlt

@dlt.table(
    name="bronze_flights"
)
def bronze_flights():

    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("/Volumes/workspace/raw/flights")
    )
```

---

# 55. Silver with expectations

```python
@dlt.expect_all_or_drop({
    "valid_flight_id": "flight_id IS NOT NULL",
    "valid_price": "price >= 0"
})
@dlt.table(
    name="silver_flights"
)
def silver_flights():

    return (
        dlt.read_stream("bronze_flights")
    )
```

Architecture:

```text
JSON
 ↓
Bronze
 ↓
Expectations
 ↓
Silver
```

---

# 56. Gold aggregation

```python
@dlt.table(
    name="gold_airline_summary"
)
def gold_airline_summary():

    return (
        dlt.read("silver_flights")
        .groupBy("airline")
        .count()
    )
```

Final:

```text
                bronze_flights
                       │
                       ▼
                silver_flights
                       │
                       ▼
             gold_airline_summary
```

---

# 57. SQL version

DLT historically supported SQL as well as Python.

Example:

```sql
CREATE OR REFRESH STREAMING TABLE bronze_flights
AS
SELECT *
FROM STREAM read_files(
  '/Volumes/workspace/raw/flights/',
  format => 'json'
);
```

Then:

```sql
CREATE OR REFRESH STREAMING TABLE silver_flights
AS
SELECT *
FROM STREAM(bronze_flights)
WHERE flight_id IS NOT NULL;
```

And:

```sql
CREATE OR REFRESH MATERIALIZED VIEW gold_airline_summary
AS
SELECT
    airline,
    COUNT(*) AS flight_count
FROM silver_flights
GROUP BY airline;
```

**Important:** The exact SQL syntax evolves with Databricks' transition from DLT terminology to Lakeflow Declarative Pipelines, so for new development you should follow the syntax supported by your current Databricks runtime/workspace.

---

# 58. DLT vs traditional ETL

Traditional:

```text
Job 1
 ↓
Write table

Job 2
 ↓
Read table
 ↓
Write table

Job 3
 ↓
Read table
 ↓
Write table
```

DLT:

```text
Dataset A
    ↓
Dataset B
    ↓
Dataset C
```

The pipeline framework understands the relationships.

---

# 59. DLT advantages

## 1. Declarative

You describe the desired data products.

## 2. Data quality

Expectations are built into the pipeline.

## 3. Dependency management

The DAG is derived from dataset relationships.

## 4. Incremental processing

Very useful for streaming and continuously arriving data.

## 5. Monitoring

Pipeline/event information is available.

## 6. Delta integration

Works naturally with Delta tables.

## 7. CDC

Supports modern CDC pipeline patterns.

## 8. Medallion architecture

Very suitable for:

```text
Bronze → Silver → Gold
```

---

# 60. Limitations / when DLT may not be the best choice

Don't use DLT simply because you're using Databricks.

A normal Job may be better when:

```text
Python processing
 ↓
External API
 ↓
File creation
 ↓
Email
```

is the main requirement.

DLT is strongest when the problem is fundamentally:

```text
Data pipeline
```

rather than:

```text
General-purpose orchestration
```

---

# 61. DLT vs Jobs

| Feature                | DLT                              | Job                    |
| ---------------------- | -------------------------------- | ---------------------- |
| Main purpose           | Data pipeline                    | Orchestration          |
| DAG                    | Data dependencies                | Task dependencies      |
| Data quality           | Built-in expectations            | You implement it       |
| Scheduling             | Pipeline scheduling options      | Excellent              |
| Notebook execution     | Yes                              | Yes                    |
| Python scripts         | Yes, depending on pipeline model | Yes                    |
| CDC                    | Strong capability                | You implement/manage   |
| Streaming              | Excellent                        | Can run streaming jobs |
| General orchestration  | Limited compared with Jobs       | Excellent              |
| Email/notifications    | Supported                        | Strong                 |
| External API workflows | Not ideal                        | Excellent              |

---

# 62. DLT vs Workflows

Think:

```text
DLT
=
Data pipeline

Workflow / Job
=
Orchestration
```

Example:

```text
Job
 │
 ├── Extract Salesforce
 │
 ├── Run DLT pipeline
 │
 ├── Run data validation
 │
 └── Send notification
```

Here:

* Job = orchestration
* DLT = data processing

---

# 63. DLT vs Airflow

Airflow:

```text
Task
 ↓
Task
 ↓
Task
```

It is an orchestration platform.

DLT:

```text
Bronze
 ↓
Silver
 ↓
Gold
```

It is primarily a declarative data pipeline framework.

You can use Airflow to trigger Databricks workloads, but you don't need Airflow simply to create a Bronze/Silver/Gold DLT pipeline.

---

# 64. DLT vs dbt

This is another useful interview comparison.

### DLT

Databricks-native pipeline framework.

```text
Spark
+
Delta
+
Streaming
+
Data Quality
+
CDC
```

### dbt

Primarily SQL-based transformation framework.

```text
SQL
+
Models
+
Tests
+
Documentation
+
Lineage
```

They can coexist.

---

# 65. DLT vs Structured Streaming

### Structured Streaming

You manage:

```text
readStream
writeStream
checkpoint
trigger
outputMode
```

### DLT

You define:

```text
What dataset should exist?
What quality rules should apply?
What depends on what?
```

Then DLT manages much of the pipeline execution.

---

# 66. Interview question: Why DLT?

A strong answer:

> DLT, now called Lakeflow Declarative Pipelines, is a Databricks framework for building reliable declarative data pipelines. It simplifies incremental processing, dependency management, data-quality enforcement through expectations, pipeline monitoring, and CDC-oriented workloads. It works particularly well with Delta Lake and the Bronze-Silver-Gold architecture.

---

# 67. Interview question: What are expectations?

Answer:

> Expectations are data-quality rules applied to pipeline datasets. For example, `customer_id IS NOT NULL`. Depending on the expectation behavior, invalid records can be reported while allowing processing to continue, dropped from the target dataset, or cause the pipeline to fail.

---

# 68. Interview question: What happens if a DLT table fails?

Suppose:

```text
Bronze → SUCCESS
Silver → FAILED
Gold
```

The downstream dataset depends on Silver.

Therefore:

```text
Gold
 ↓
cannot correctly process its dependency
```

The pipeline's dependency graph helps determine which datasets can be processed and which must wait/fail based on the dependency state.

---

# 69. Interview question: DLT vs Delta Lake

Answer:

> Delta Lake is a storage/table format that provides features such as ACID transactions, transaction logs, schema enforcement, and time travel. DLT/Lakeflow Declarative Pipelines is a pipeline framework that uses technologies such as Delta Lake to build and manage data pipelines.

---

# 70. Interview question: DLT vs Spark Structured Streaming

Answer:

> Structured Streaming is Spark's streaming execution engine, where developers explicitly configure streaming reads, writes, checkpoints, triggers, and output modes. DLT provides a higher-level declarative pipeline framework around data transformations, dependencies, data quality, monitoring, and incremental processing.

---

# 71. The complete mental model

Remember this:

```text
                       DATABRICKS
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
       Delta Lake        Jobs           DLT/Lakeflow
          │                │                │
      Storage           Workflow       Data Pipeline
          │                │                │
          │          ┌─────┴─────┐          │
          │          ▼           ▼          │
          │       Notebook      SQL         │
          │                              ┌───┴───┐
          │                              ▼       ▼
          │                           Bronze   Silver
          │                                      │
          │                                      ▼
          │                                     Gold
          │
          └──────────── Tables ─────────────────┘
```

---

# 72. Most important concepts to remember

If you're preparing for a **Databricks/PySpark Data Engineering interview**, prioritize these:

### Level 1 — Must know

```text
DLT
 ↓
Declarative pipeline
 ↓
Bronze → Silver → Gold
 ↓
Streaming tables
 ↓
Materialized views
 ↓
Expectations
```

### Level 2 — Very important

```text
DAG
Auto Loader
Delta Lake
Checkpointing
Incremental processing
Pipeline monitoring
Unity Catalog
```

### Level 3 — Advanced

```text
CDC
Apply Changes
Sequence by
Deduplication
SCD Type 1
SCD Type 2
Schema evolution
Event logs
Pipeline recovery
```

---

# 73. One complete real-world example

Imagine your project:

```text
Salesforce
     │
     ▼
     S3
     │
     ▼
 Auto Loader
     │
     ▼
┌──────────────┐
│    BRONZE    │
│ Raw records  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    SILVER    │
│ Clean        │
│ Deduplicate  │
│ Validate     │
│ CDC          │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│     GOLD     │
│ Aggregations │
│ Business     │
│ Metrics      │
└──────────────┘
```

DLT/Lakeflow provides the framework around this pipeline.

Delta Lake provides the underlying reliable table storage.

Unity Catalog provides governance.

A Job can orchestrate when the pipeline runs.

So the complete architecture becomes:

```text
                   Unity Catalog
                         │
                         │ Governance
                         ▼
Salesforce → S3 → Auto Loader → DLT/Lakeflow
                                  │
                         ┌────────┼────────┐
                         ▼        ▼        ▼
                      Bronze   Silver     Gold
                         │        │        │
                         └────────┼────────┘
                                  ▼
                            Delta Lake
                                  │
                                  ▼
                             BI / Reports
```

---

# 74. Final comparison

| Technology                               | Main responsibility                 |
| ---------------------------------------- | ----------------------------------- |
| **Spark**                                | Distributed processing engine       |
| **Structured Streaming**                 | Streaming execution engine          |
| **Delta Lake**                           | Reliable table/storage layer        |
| **DLT / Lakeflow Declarative Pipelines** | Declarative data pipeline framework |
| **Databricks Jobs**                      | Workflow/orchestration              |
| **Auto Loader**                          | Incremental cloud-file ingestion    |
| **Unity Catalog**                        | Governance, security, metadata      |
| **dbt**                                  | SQL transformation/modeling         |
| **Airflow**                              | General workflow orchestration      |

The easiest way to remember the relationships is:

```text
                         Databricks
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
       Spark             Delta Lake          Jobs
          │                  │                  │
   Processing engine    Table/storage      Orchestration
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                             ▼
                   DLT / Lakeflow
                             │
                Declarative Data Pipeline
                             │
                 ┌───────────┼───────────┐
                 ▼           ▼           ▼
              Bronze      Silver       Gold
                 │           │           │
                 └───────────┼───────────┘
                             ▼
                      Business Data
```

**One terminology point to remember for interviews in 2026:** if someone says **“DLT”**, understand it as the older **Delta Live Tables** name; Databricks now positions this technology as **Lakeflow Declarative Pipelines**. So a good interview answer is: **“DLT, now called Lakeflow Declarative Pipelines…”**
