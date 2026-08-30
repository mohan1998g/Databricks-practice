# Two-Level and Three-Level Namespaces — Detailed Guide

## 1. What is a Namespace?

A **namespace** is the hierarchical naming structure used to uniquely identify data objects such as:

- Tables
- Views
- Schemas
- Databases
- Catalogs
- Functions
- Volumes/files in some platforms

The main purpose of a namespace is to avoid ambiguity.

For example, suppose two schemas both contain a table called `customers`:

```text
sales.customers
marketing.customers
```

The schema/database level distinguishes the two tables.

Modern data platforms commonly use either:

- **Two-level naming**
- **Three-level naming**

The exact terminology varies slightly between Spark, Databricks, Snowflake, BigQuery, Trino, Athena, Iceberg, etc.

---

# 2. Two-Level Namespace

A two-level namespace identifies an object using two components:

```text
<namespace>.<object>
```

The most common form is:

```text
schema.table
```

or:

```text
database.table
```

Depending on the platform, the first level may be called a **database**, **schema**, or **namespace**.

## Example

```text
sales.customers
```

Here:

```text
sales      → namespace/schema/database
customers  → table
```

Another example:

```text
finance.transactions
```

Here:

```text
finance       → namespace
transactions  → table
```

---

# 3. How a Two-Level Namespace Works

Imagine the following structure:

```text
Database
│
├── customers
├── orders
├── products
└── employees
```

If the database is called `sales`, the tables can be referenced as:

```text
sales.customers
sales.orders
sales.products
sales.employees
```

The fully qualified table name has two parts:

```text
sales.customers
│     │
│     └── Object/Table
└──────── Namespace/Database
```

---

# 4. Why Two-Level Namespaces Are Useful

Two-level namespaces provide basic isolation.

For example:

```text
sales.customers
hr.customers
marketing.customers
```

All three tables can have the same table name:

```text
customers
```

but they are different objects because their namespaces are different.

Without namespaces, you would have:

```text
customers
customers
customers
```

which would be ambiguous.

---

# 5. Two-Level Namespace Example in Spark

In environments using a traditional Hive-style catalog, you may encounter:

```sql
SELECT *
FROM sales.customers;
```

Here:

```text
sales     → database/schema
customers → table
```

PySpark:

```python
df = spark.table("sales.customers")
```

Or:

```python
df = spark.read.table("sales.customers")
```

You can also write:

```python
df.write.saveAsTable("sales.customers")
```

---

# 6. Two-Level Namespace and the Default Catalog

In Spark, the catalog can exist above the database/schema level.

For example:

```text
spark_catalog
    │
    ├── sales
    │     ├── customers
    │     └── orders
    │
    └── hr
          ├── employees
          └── salaries
```

A table may therefore conceptually have:

```text
catalog.schema.table
```

but users sometimes refer to it using:

```text
schema.table
```

because the catalog is implicit.

For example:

```sql
SELECT * FROM sales.customers;
```

can conceptually resolve to:

```text
spark_catalog.sales.customers
```

when `spark_catalog` is the current/default catalog.

This distinction becomes extremely important when working with modern Spark and Databricks.

---

# 7. Three-Level Namespace

A three-level namespace contains three components:

```text
<catalog>.<schema>.<object>
```

The most common representation is:

```text
catalog.schema.table
```

For example:

```text
main.sales.customers
```

Here:

```text
main      → catalog
sales     → schema
customers → table
```

Diagram:

```text
main.sales.customers
│    │     │
│    │     └── Table
│    └──────── Schema
└───────────── Catalog
```

---

# 8. Why Three Levels?

A three-level namespace provides an additional isolation boundary.

Compare:

## Two levels

```text
sales.customers
```

Only two components exist:

```text
schema.table
```

## Three levels

```text
production.sales.customers
```

Three components exist:

```text
catalog.schema.table
```

The catalog provides another logical boundary.

---

# 9. Three-Level Namespace Hierarchy

A typical hierarchy looks like:

```text
Catalog
│
├── Schema
│   ├── Table
│   ├── Table
│   └── View
│
├── Schema
│   ├── Table
│   └── Table
│
└── Schema
    └── Table
```

For example:

```text
prod
│
├── sales
│   ├── customers
│   ├── orders
│   └── products
│
├── finance
│   ├── transactions
│   └── payments
│
└── hr
    ├── employees
    └── departments
```

Fully qualified names:

```text
prod.sales.customers
prod.sales.orders

prod.finance.transactions
prod.finance.payments

prod.hr.employees
prod.hr.departments
```

---

# 10. Three-Level Namespace in Databricks Unity Catalog

One of the most important examples for a Data Engineer is **Databricks Unity Catalog**.

Unity Catalog uses a three-level namespace:

```text
catalog.schema.object
```

For example:

```text
production.sales.customers
```

The three levels are:

```text
production → Catalog
sales       → Schema
customers   → Table
```

Another example:

```text
dev.sales.customers
test.sales.customers
prod.sales.customers
```

All three can contain:

```text
sales.customers
```

because the catalog separates them.

---

# 11. Databricks Unity Catalog Hierarchy

A simplified Unity Catalog structure is:

```text
Metastore
│
├── Catalog
│   │
│   ├── Schema
│   │   ├── Table
│   │   ├── Table
│   │   └── View
│   │
│   └── Schema
│       └── Table
│
└── Catalog
    └── Schema
        └── Table
```

For example:

```text
company_metastore
│
├── dev
│   ├── sales
│   │   ├── customers
│   │   └── orders
│   │
│   └── finance
│       └── transactions
│
└── prod
    ├── sales
    │   ├── customers
    │   └── orders
    │
    └── finance
        └── transactions
```

A table can be referenced as:

```sql
SELECT *
FROM prod.sales.customers;
```

---

# 12. Why Unity Catalog Uses Three Levels

Suppose an organization has:

```text
Development
Testing
Production
```

With three catalogs:

```text
dev
test
prod
```

Each catalog can contain:

```text
sales
finance
hr
```

So you can have:

```text
dev.sales.customers
test.sales.customers
prod.sales.customers
```

This is much cleaner than trying to put everything into one namespace.

---

# 13. Three-Level Namespace in PySpark

Example:

```python
df = spark.table("prod.sales.customers")
```

SQL:

```python
df = spark.sql("""
    SELECT *
    FROM prod.sales.customers
""")
```

Writing:

```python
df.write.mode("append").saveAsTable(
    "prod.sales.customers"
)
```

Creating:

```sql
CREATE TABLE prod.sales.customers (
    customer_id BIGINT,
    customer_name STRING
);
```

---

# 14. Two-Level vs Three-Level Namespace

| Feature | Two-Level | Three-Level |
|---|---|---|
| Format | `schema.table` | `catalog.schema.table` |
| Components | 2 | 3 |
| Catalog explicit? | Usually no | Yes |
| Isolation | Basic | Greater |
| Environment separation | More difficult | Easier |
| Multi-catalog architecture | Limited | Strong |
| Common in modern lakehouses | Less common | Very common |
| Databricks Unity Catalog | Not the primary fully-qualified form | Yes |
| Example | `sales.customers` | `prod.sales.customers` |

---

# 15. Simple Real-World Example

Suppose you work for an e-commerce company.

You have:

```text
Development
Production
```

and departments:

```text
Sales
Finance
Marketing
```

and tables:

```text
customers
orders
transactions
campaigns
```

## With a two-level namespace

You might have:

```text
sales.customers
sales.orders

finance.transactions

marketing.campaigns
```

But separating development and production can become less explicit.

You might need separate catalogs/metastores or other mechanisms.

## With a three-level namespace

You can directly represent the environment:

```text
dev.sales.customers
dev.sales.orders

prod.sales.customers
prod.sales.orders

prod.finance.transactions

prod.marketing.campaigns
```

The first component clearly identifies the catalog/environment.

---

# 16. Another Important Concept: Fully Qualified Name

A **fully qualified name (FQN)** identifies an object using all namespace levels necessary to uniquely locate it.

For three-level systems:

```text
catalog.schema.table
```

Example:

```text
prod.sales.customers
```

For two-level systems:

```text
schema.table
```

Example:

```text
sales.customers
```

The more explicit the namespace, the less dependent your SQL is on the current/default context.

---

# 17. Current Catalog and Current Schema

Modern systems often maintain a current catalog and current schema.

Suppose:

```text
Current catalog = prod
Current schema  = sales
```

Then:

```sql
SELECT * FROM customers;
```

may resolve to:

```text
prod.sales.customers
```

You could explicitly specify:

```sql
SELECT * FROM sales.customers;
```

which may resolve to:

```text
prod.sales.customers
```

And the most explicit form is:

```sql
SELECT * FROM prod.sales.customers;
```

This is why the same system can appear to support both two-level and three-level references.

---

# 18. Spark SQL Namespace Concepts

Apache Spark SQL has catalog and namespace concepts.

A common modern Spark SQL pattern is:

```text
catalog.namespace.table
```

where the middle component is commonly a schema/database.

For example:

```text
spark_catalog.default.customers
```

Here:

```text
spark_catalog → Catalog
default       → Namespace/database/schema
customers     → Table
```

You can also commonly write:

```text
default.customers
```

when the catalog is implicit.

---

# 19. Important Interview Point: Spark Is Not Simply "Two-Level" or "Three-Level"

A common interview mistake is saying:

> "Spark uses two-level namespaces."

That is too simplistic.

Modern Spark supports catalog-qualified identifiers, conceptually:

```text
catalog.namespace.object
```

But depending on the catalog implementation and context, you may commonly see:

```text
namespace.object
```

or:

```text
catalog.namespace.object
```

Therefore, the better interview answer is:

> Spark SQL supports catalog-qualified identifiers. A table can commonly be referenced using `catalog.schema.table`, while the catalog may be implicit, allowing `schema.table`.

---

# 20. Platforms Using Three-Level-Style Namespaces

Several data platforms use a three-part naming model, although the terminology and semantics differ.

## 20.1 Databricks Unity Catalog

Format:

```text
catalog.schema.table
```

Example:

```text
prod.sales.orders
```

This is one of the clearest examples of a three-level namespace in modern data engineering.

---

## 20.2 Apache Spark SQL

Spark supports catalog-qualified identifiers.

Conceptually:

```text
catalog.namespace.table
```

Example:

```text
spark_catalog.sales.orders
```

Depending on the catalog and SQL context, the catalog can be omitted:

```text
sales.orders
```

---

## 20.3 Google BigQuery

BigQuery commonly uses:

```text
project.dataset.table
```

Example:

```text
my-project.sales.customers
```

Mapping:

```text
my-project → Project
sales      → Dataset
customers  → Table
```

This looks like:

```text
project.dataset.table
```

It is structurally similar to a three-level namespace, although BigQuery's terminology is different from Databricks.

### Important difference

Do not automatically call the BigQuery `project` a Spark/Databricks "catalog".

It is better to say:

> BigQuery uses a three-part table identifier: project, dataset, table.

---

# 21. Amazon Athena

Athena commonly uses:

```text
catalog.database.table
```

Example:

```text
AwsDataCatalog.sales.orders
```

Here:

```text
AwsDataCatalog → Catalog
sales           → Database
orders          → Table
```

This is a direct three-level model.

---

# 22. Trino / Presto

Trino commonly uses:

```text
catalog.schema.table
```

Example:

```text
hive.sales.orders
```

Here:

```text
hive   → Catalog
sales  → Schema
orders → Table
```

This is very similar to Spark's catalog/schema/table concept.

---

# 23. Snowflake

Snowflake commonly uses:

```text
database.schema.table
```

Example:

```text
PROD.SALES.CUSTOMERS
```

Here:

```text
PROD      → Database
SALES     → Schema
CUSTOMERS → Table
```

This is a three-part identifier, but Snowflake's hierarchy differs from Databricks Unity Catalog.

For example:

```text
Snowflake:
database.schema.table

Databricks:
catalog.schema.table
```

The number of levels is the same, but the meaning of the first level is different.

---

# 24. SQL Server

SQL Server commonly supports:

```text
database.schema.table
```

Example:

```text
SalesDB.dbo.Customers
```

Here:

```text
SalesDB  → Database
dbo      → Schema
Customers → Table
```

This is another three-part identifier.

SQL Server can also support server-qualified four-part names:

```text
server.database.schema.table
```

Therefore, SQL Server is an excellent example showing that "two-level" and "three-level" are context-dependent concepts.

---

# 25. PostgreSQL

PostgreSQL generally uses:

```text
schema.table
```

Example:

```text
sales.orders
```

The database is normally a connection-level boundary rather than part of the standard table identifier.

Therefore PostgreSQL is commonly encountered as a **two-level table namespace**:

```text
schema.table
```

---

# 26. MySQL

MySQL commonly uses:

```text
database.table
```

Example:

```text
sales.orders
```

MySQL uses the term **database** for what many other systems call a schema.

So:

```text
sales → Database
orders → Table
```

is a two-part identifier.

---

# 27. Amazon Redshift

Redshift commonly uses:

```text
schema.table
```

Example:

```text
sales.orders
```

The database is generally associated with the connection/session rather than being part of the ordinary two-part table reference.

So Redshift is commonly encountered using:

```text
schema.table
```

---

# 28. PostgreSQL vs MySQL vs Snowflake

A useful comparison:

| Platform | Typical table identifier | Levels |
|---|---|---:|
| PostgreSQL | `schema.table` | 2 |
| MySQL | `database.table` | 2 |
| Redshift | `schema.table` | 2 |
| Snowflake | `database.schema.table` | 3 |
| Databricks Unity Catalog | `catalog.schema.table` | 3 |
| BigQuery | `project.dataset.table` | 3 |
| Trino | `catalog.schema.table` | 3 |
| Athena | `catalog.database.table` | 3 |
| SQL Server | `database.schema.table` | 3 |

---

# 29. Two-Level Namespace in Data Lakes

Traditional data lake implementations often have a simpler structure.

For example:

```text
database
    │
    ├── customers
    ├── orders
    └── products
```

A Hive Metastore-style environment may commonly expose:

```text
sales.customers
sales.orders
```

The metastore itself may be responsible for maintaining metadata about the databases and tables.

---

# 30. Three-Level Namespace in a Lakehouse

Modern lakehouses often introduce an explicit catalog layer.

Example:

```text
Metastore
    │
    ├── dev
    │   └── sales
    │       └── customers
    │
    └── prod
        └── sales
            └── customers
```

The table names become:

```text
dev.sales.customers
prod.sales.customers
```

This gives a clean hierarchy:

```text
Metastore
   ↓
Catalog
   ↓
Schema
   ↓
Table
```

---

# 31. Namespace vs Storage Location

This is extremely important.

A namespace is a **logical naming system**.

A storage location is a **physical location**.

For example:

```text
Logical name:
prod.sales.customers
```

The physical storage could be:

```text
s3://company-data/prod/sales/customers/
```

These are not the same thing.

The namespace tells Spark/Databricks:

> Which table are you referring to?

The storage location tells the system:

> Where are the underlying data files?

---

# 32. Example with Delta Lake

Suppose:

```text
prod.sales.customers
```

is a Delta table.

Its physical location could be:

```text
s3://company-lake/prod/sales/customers/
```

Inside the location you might have:

```text
customers/
│
├── _delta_log/
│   ├── 00000000000000000000.json
│   ├── 00000000000000000001.json
│   └── ...
│
├── part-00000-....parquet
├── part-00001-....parquet
└── part-00002-....parquet
```

The namespace:

```text
prod.sales.customers
```

does not itself represent the S3 path.

---

# 33. Namespace and Metastore

A metastore stores metadata about objects.

For example:

```text
Catalog
   ↓
Schema
   ↓
Table
   ↓
Metadata
   ↓
Storage Location
```

For:

```text
prod.sales.customers
```

metadata may include:

```text
table type       = Delta
location         = s3://company-lake/prod/sales/customers
schema           = customer_id, name, email
partition info   = ...
properties       = ...
```

The namespace is how you identify the object.

---

# 34. Two-Level Namespace Example — Hive Metastore

A traditional Hive Metastore environment can look like:

```text
Hive Metastore
│
├── sales
│   ├── customers
│   └── orders
│
└── finance
    └── transactions
```

References:

```sql
SELECT * FROM sales.customers;
SELECT * FROM finance.transactions;
```

The catalog is usually implicit/default:

```text
spark_catalog.sales.customers
```

---

# 35. Three-Level Namespace Example — Unity Catalog

Unity Catalog:

```text
Metastore
│
├── prod
│   ├── sales
│   │   ├── customers
│   │   └── orders
│   │
│   └── finance
│       └── transactions
│
└── dev
    └── sales
        └── customers
```

References:

```sql
SELECT * FROM prod.sales.customers;
SELECT * FROM prod.finance.transactions;
SELECT * FROM dev.sales.customers;
```

---

# 36. Why the Catalog Layer Is Important

The catalog layer can provide a strong organizational and governance boundary.

For example:

```text
prod
├── sales
├── finance
└── hr

dev
├── sales
├── finance
└── hr
```

Now the same schema/table combination can exist in different catalogs:

```text
dev.sales.customers
prod.sales.customers
```

This is extremely useful for:

- Development
- Testing
- Production
- Business units
- Data domains
- Governance
- Access control
- Data sharing
- Multi-tenant architectures

---

# 37. Catalog Does Not Always Mean Environment

An important interview point:

You **can** use catalogs for environments:

```text
dev.sales.customers
test.sales.customers
prod.sales.customers
```

But a catalog does not inherently mean "environment."

You could instead organize catalogs by:

```text
company
partner
external
finance
analytics
```

The exact design depends on governance and architecture requirements.

---

# 38. Namespace Resolution

Suppose the current catalog is:

```text
prod
```

and current schema is:

```text
sales
```

Then:

```sql
SELECT * FROM customers;
```

can resolve to:

```text
prod.sales.customers
```

If you specify:

```sql
SELECT * FROM sales.customers;
```

the catalog may be inferred:

```text
prod.sales.customers
```

If you specify:

```sql
SELECT * FROM prod.sales.customers;
```

the reference is explicit.

Conceptually:

```text
customers
    ↓
sales.customers
    ↓
prod.sales.customers
```

This is namespace resolution.

---

# 39. Why Fully Qualified Names Are Useful

Consider this:

```python
df = spark.table("customers")
```

It depends on the current catalog/schema.

A more explicit version is:

```python
df = spark.table("sales.customers")
```

Even more explicit:

```python
df = spark.table("prod.sales.customers")
```

The last form is less dependent on session context.

This can be especially useful in:

- Production jobs
- Shared notebooks
- CI/CD pipelines
- Deployment pipelines
- Cross-catalog queries
- Multi-environment architectures

---

# 40. Cross-Catalog Query

With three-level namespaces, you can query objects from different catalogs when the platform permits it.

Example:

```sql
SELECT *
FROM prod.sales.customers;
```

and:

```sql
SELECT *
FROM dev.sales.customers;
```

You could potentially join them:

```sql
SELECT
    p.customer_id,
    p.name,
    d.status
FROM prod.sales.customers p
JOIN dev.sales.customers d
    ON p.customer_id = d.customer_id;
```

Whether cross-catalog access is permitted depends on platform permissions and catalog configuration.

---

# 41. Two-Level vs Three-Level in Interview Questions

### Interview question:

> What is a two-level namespace?

Answer:

> A two-level namespace identifies an object using two components, commonly `schema.table` or `database.table`. The catalog/database context may be implicit.

Example:

```text
sales.customers
```

---

### Interview question:

> What is a three-level namespace?

Answer:

> A three-level namespace identifies an object using three components, commonly `catalog.schema.table`. The catalog provides an additional logical boundary above the schema.

Example:

```text
prod.sales.customers
```

---

### Interview question:

> Which namespace does Unity Catalog use?

Answer:

> Unity Catalog uses a three-level namespace: `catalog.schema.object`.

Example:

```text
prod.sales.customers
```

---

# 42. Very Important Terminology Difference

Do not assume these words are interchangeable across every platform:

```text
Catalog
Database
Schema
Namespace
Project
Dataset
```

For example:

### Databricks

```text
catalog.schema.table
```

### BigQuery

```text
project.dataset.table
```

### Snowflake

```text
database.schema.table
```

### MySQL

```text
database.table
```

They all look similar, but the semantics and governance models are different.

---

# 43. Comparison of the First Level

| Platform | First level | Second level | Third level |
|---|---|---|---|
| Databricks Unity Catalog | Catalog | Schema | Table |
| Spark | Catalog | Namespace/Schema | Table |
| BigQuery | Project | Dataset | Table |
| Snowflake | Database | Schema | Table |
| Athena | Catalog | Database | Table |
| Trino | Catalog | Schema | Table |
| SQL Server | Database | Schema | Table |
| MySQL | Database | — | Table |
| PostgreSQL | Schema | — | Table |

---

# 44. Common Data Engineering Architecture

A modern organization might design:

```text
Unity Catalog
│
├── dev
│   ├── bronze
│   ├── silver
│   └── gold
│
├── test
│   ├── bronze
│   ├── silver
│   └── gold
│
└── prod
    ├── bronze
    ├── silver
    └── gold
```

Tables:

```text
prod.bronze.salesforce_customer
prod.silver.customer
prod.gold.customer_360
```

Here:

```text
prod     → Catalog
bronze   → Schema
customer → Table
```

This combines:

- Three-level namespaces
- Medallion architecture
- Environment isolation

---

# 45. Three-Level Namespace + Medallion Architecture

A practical Databricks architecture can look like:

```text
Catalog
│
├── dev
│   ├── bronze
│   │   └── raw_customer
│   ├── silver
│   │   └── customer
│   └── gold
│       └── customer_360
│
└── prod
    ├── bronze
    │   └── raw_customer
    ├── silver
    │   └── customer
    └── gold
        └── customer_360
```

Fully qualified tables:

```text
dev.bronze.raw_customer
dev.silver.customer
dev.gold.customer_360

prod.bronze.raw_customer
prod.silver.customer
prod.gold.customer_360
```

This is a very common pattern to understand for Databricks Data Engineering interviews.

---

# 46. Common Mistake: Confusing Catalog and Metastore

In Unity Catalog:

```text
Metastore
    ↓
Catalog
    ↓
Schema
    ↓
Table
```

Do not write:

```text
metastore.schema.table
```

as the normal Unity Catalog table identifier.

The normal three-level table identifier is:

```text
catalog.schema.table
```

The metastore is a higher-level container that can contain catalogs.

---

# 47. Common Mistake: Confusing Schema and Database

Different platforms use different terminology.

For example:

```text
MySQL:
database.table
```

while:

```text
PostgreSQL:
schema.table
```

and:

```text
Snowflake:
database.schema.table
```

and:

```text
Databricks Unity Catalog:
catalog.schema.table
```

Therefore, always answer using the terminology of the specific platform.

---

# 48. Common Mistake: Assuming Every Three-Part Name Has the Same Meaning

These are all three-part names:

```text
prod.sales.customers
my-project.sales.customers
PROD.SALES.CUSTOMERS
hive.sales.customers
```

But their meanings differ.

### Databricks

```text
prod.sales.customers
catalog.schema.table
```

### BigQuery

```text
my-project.sales.customers
project.dataset.table
```

### Snowflake

```text
PROD.SALES.CUSTOMERS
database.schema.table
```

### Trino

```text
hive.sales.customers
catalog.schema.table
```

The structure is similar; the semantics are platform-specific.

---

# 49. Quick Visual Comparison

```text
TWO LEVEL
──────────

sales.customers
│     │
│     └── Table
└──────── Schema/Database
```

```text
THREE LEVEL
───────────

prod.sales.customers
│    │     │
│    │     └── Table
│    └──────── Schema
└───────────── Catalog
```

---

# 50. Easy Way to Remember

Think of a physical address.

## Two levels

```text
Building → Room
```

## Three levels

```text
Campus → Building → Room
```

Similarly:

### Two-level data namespace

```text
schema → table
```

### Three-level data namespace

```text
catalog → schema → table
```

The additional catalog level gives another organizational boundary.

---

# 51. Interview Cheat Sheet

| Question | Short Answer |
|---|---|
| What is a namespace? | A hierarchy used to uniquely identify data objects. |
| Two-level namespace? | Usually `schema.table` or `database.table`. |
| Three-level namespace? | Usually `catalog.schema.table`. |
| Databricks Unity Catalog? | `catalog.schema.object`. |
| BigQuery? | `project.dataset.table`. |
| Snowflake? | `database.schema.table`. |
| Trino? | `catalog.schema.table`. |
| Athena? | `catalog.database.table`. |
| PostgreSQL? | Commonly `schema.table`. |
| MySQL? | Commonly `database.table`. |
| Why use three levels? | Additional organization, isolation, governance and qualification. |
| Is a catalog the same as a database? | No. Their meaning depends on the platform. |
| Is namespace the same as storage path? | No. Namespace is logical; storage path is physical. |
| Does Spark only support two levels? | No. Spark supports catalog-qualified identifiers; the catalog can be implicit. |

---

# 52. Final Summary

## Two-Level Namespace

```text
schema.table
```

or:

```text
database.table
```

Example:

```text
sales.customers
```

Typical examples include:

- PostgreSQL
- MySQL
- Redshift
- Traditional Hive-style usage
- Spark usage when the catalog is implicit

---

## Three-Level Namespace

```text
catalog.schema.table
```

Example:

```text
prod.sales.customers
```

Important examples include:

- Databricks Unity Catalog
- Apache Spark catalog-qualified identifiers
- Trino
- Amazon Athena
- Snowflake uses a similar three-part structure but with `database.schema.table`
- BigQuery uses `project.dataset.table`
- SQL Server supports `database.schema.table`

---

# 53. Most Important Point for a PySpark/Databricks Interview

If the interviewer asks:

> "Explain two-level and three-level namespaces."

A strong answer is:

> A two-level namespace generally identifies a table as `schema.table` or `database.table`, where the catalog/database context may be implicit. A three-level namespace explicitly includes an additional catalog-level component, normally written as `catalog.schema.table`. In Databricks Unity Catalog, the standard three-level namespace is `catalog.schema.table`. For example, `prod.sales.customers` means catalog `prod`, schema `sales`, and table `customers`. The catalog provides an additional logical boundary useful for organizing environments, domains and governance. Spark also supports catalog-qualified identifiers, although the catalog can be implicit depending on the context.

That answer demonstrates both **Spark knowledge** and **Databricks/Unity Catalog knowledge** without incorrectly treating every platform's terminology as identical.
