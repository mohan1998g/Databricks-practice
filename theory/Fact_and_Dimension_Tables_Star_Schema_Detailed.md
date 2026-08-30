# Fact Tables and Dimension Tables — Detailed Guide

## 1. Introduction

Fact tables and dimension tables are the two fundamental building blocks of a **dimensional data warehouse model**.

They are primarily used to organize analytical data so that business questions can be answered efficiently.

A typical dimensional model contains:

```text
                    Dimension
                       │
                       │
Dimension ─────── Fact Table ─────── Dimension
                       │
                       │
                    Dimension
```

For example, in a retail company:

```text
                    dim_date
                       │
                       │
dim_customer ───── fact_sales ───── dim_product
                       │
                       │
                  dim_store
```

The fact table contains the **business events and measurements**, while dimension tables contain the **descriptive information used to understand and filter those events**.

---

# 2. What Is a Fact Table?

A **fact table** stores measurable business events or transactions.

Examples:

- A product was sold.
- A customer placed an order.
- A payment was made.
- A flight was booked.
- A bank transaction occurred.
- A shipment was delivered.
- A claim was submitted.

A fact table generally contains:

1. Foreign keys pointing to dimensions.
2. Numeric measurements called **facts/measures**.
3. Sometimes transaction identifiers or degenerate dimensions.
4. A clearly defined **grain**.

Example:

```text
fact_sales

sale_id
date_key
customer_key
product_key
store_key
quantity
sales_amount
discount_amount
cost_amount
profit_amount
```

---

# 3. What Is a Dimension Table?

A **dimension table** contains descriptive attributes that provide context for the facts.

For example:

```text
dim_customer

customer_key
customer_id
customer_name
gender
city
state
country
customer_segment
```

A fact table tells us:

> What happened?

A dimension table tells us:

> Who, what, where, when, why, or how?

For example:

```text
fact_sales
-----------
quantity = 2
sales_amount = 2000
```

By joining to dimensions, we can understand:

```text
Customer = Mohan
Product  = Laptop
Date     = 2026-08-30
Store    = Hyderabad
```

---

# 4. Simple Difference Between Fact and Dimension

The easiest way to remember:

```text
FACT       = WHAT HAPPENED + MEASUREMENTS

DIMENSION  = INFORMATION THAT DESCRIBES WHAT HAPPENED
```

Example:

A customer purchases a laptop.

The event:

```text
Customer bought laptop
```

Fact:

```text
quantity = 1
sales_amount = 75000
```

Dimensions:

```text
Customer → Mohan
Product  → Dell Laptop
Date     → 30-Aug-2026
Store    → Hyderabad
```

---

# 5. Real-World Example

Suppose Amazon-like e-commerce data contains:

```text
Customer:
Mohan

Product:
Laptop

Date:
30-Aug-2026

Store:
Hyderabad

Quantity:
2

Price:
₹50,000
```

The fact table may contain:

| sale_id | date_key | customer_key | product_key | store_key | quantity | sales_amount |
|---:|---:|---:|---:|---:|---:|---:|
| 1001 | 20260830 | 101 | 501 | 10 | 2 | 100000 |

The dimension tables provide the descriptions.

### Customer

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Mohan | Hyderabad |

### Product

| product_key | product_id | product_name | category |
|---:|---|---|---|
| 501 | P001 | Laptop | Electronics |

### Date

| date_key | date | month | quarter | year |
|---:|---|---|---|---:|
| 20260830 | 2026-08-30 | August | Q3 | 2026 |

### Store

| store_key | store_id | store_name | city |
|---:|---|---|---|
| 10 | S001 | Hyderabad Store | Hyderabad |

---

# 6. Fact Table Characteristics

A fact table usually has:

```text
Large number of rows
+
Foreign keys
+
Measures
+
Business event/transaction information
```

For example:

```text
fact_sales
```

could contain:

```text
500 million rows
```

while:

```text
dim_customer
```

might contain:

```text
10 million rows
```

and:

```text
dim_product
```

might contain:

```text
100,000 rows
```

Fact tables are usually much larger than dimension tables.

---

# 7. Dimension Table Characteristics

Dimension tables generally contain:

```text
Descriptive attributes
+
Business identifiers
+
Surrogate keys
```

Example:

```text
dim_product
```

could contain:

```text
product_key
product_id
product_name
brand
category
subcategory
color
size
```

These attributes are used for:

- Filtering
- Grouping
- Reporting
- Drill-down
- Drill-up
- Slicing
- Dicing

---

# 8. What Is a Measure?

A **measure** is a value stored in a fact table that can usually be aggregated.

Examples:

```text
quantity
sales_amount
cost_amount
profit_amount
tax_amount
discount_amount
```

For example:

```text
quantity = 5
sales_amount = 5000
discount_amount = 500
```

We can calculate:

```text
Total Sales
Total Quantity
Average Sales
Maximum Sales
Minimum Sales
Total Profit
```

---

# 9. Additive, Semi-Additive and Non-Additive Facts

This is an important data warehouse interview topic.

## 9.1 Additive Fact

Can be summed across all dimensions.

Example:

```text
sales_amount
quantity
cost
```

If:

```text
Monday sales = 1000
Tuesday sales = 2000
```

then:

```text
Total sales = 3000
```

---

## 9.2 Semi-Additive Fact

Can be summed across some dimensions but not all.

Classic example:

```text
account_balance
```

You can add balances across:

```text
customers
```

but adding balances across time may be meaningless.

Example:

```text
Monday balance = 1000
Tuesday balance = 1200
```

It does NOT mean:

```text
Balance = 2200
```

Instead, we might use:

```text
latest balance
average balance
```

---

## 9.3 Non-Additive Fact

Cannot meaningfully be summed.

Examples:

```text
percentage
ratio
margin percentage
```

For example:

```text
Monday margin = 20%
Tuesday margin = 30%
```

You generally should not calculate:

```text
20% + 30% = 50%
```

Instead, calculate the appropriate weighted ratio from underlying measures.

---

# 10. What Is Grain?

**Grain is one of the most important concepts in fact table design.**

Grain defines:

> What exactly does one row in the fact table represent?

For example:

```text
One row = one product on one sales order
```

or:

```text
One row = one complete order
```

or:

```text
One row = one customer account balance per day
```

You must define the grain before designing the fact table.

---

# 11. Example of Grain

Suppose order:

```text
Order 1001
```

contains:

```text
Laptop
Mouse
Keyboard
```

If the grain is:

```text
One row per order
```

then:

```text
fact_order
```

may have:

```text
1001 | 3 items | ₹75,000
```

But if the grain is:

```text
One row per order line
```

then:

```text
1001 | Laptop   | 1 | ₹60,000
1001 | Mouse    | 1 | ₹5,000
1001 | Keyboard | 1 | ₹10,000
```

The second design provides much more detailed analytical capability.

---

# 12. Grain and Fact Table Design

Before creating:

```text
fact_sales
```

you should state:

```text
Grain:
One row represents one product line purchased in one sales transaction.
```

Then the columns make sense:

```text
sale_key
order_id
date_key
customer_key
product_key
store_key
quantity
unit_price
discount_amount
sales_amount
```

---

# 13. Dimension Keys

Fact tables normally reference dimensions through keys.

Example:

```text
fact_sales
```

contains:

```text
customer_key
product_key
date_key
store_key
```

These are foreign keys logically pointing to dimension tables.

For example:

```text
fact_sales.customer_key
        ↓
dim_customer.customer_key
```

and:

```text
fact_sales.product_key
        ↓
dim_product.product_key
```

---

# 14. Surrogate Keys

Data warehouses commonly use **surrogate keys** in dimensions.

Example:

```text
dim_customer

customer_key
customer_id
customer_name
city
```

Here:

```text
customer_key = surrogate key
customer_id  = business/natural key
```

Example:

| customer_key | customer_id | customer_name |
|---:|---|---|
| 101 | C001 | Mohan |
| 102 | C002 | Ravi |

The source system might use:

```text
C001
```

as the customer ID.

The warehouse may generate:

```text
101
```

as the surrogate key.

---

# 15. Why Use Surrogate Keys?

One major reason is **historical tracking**.

Suppose customer:

```text
C001
```

moves from:

```text
Hyderabad
```

to:

```text
Bangalore
```

With a Type 2 Slowly Changing Dimension, we may maintain:

| customer_key | customer_id | city | start_date | end_date | current_flag |
|---:|---|---|---|---|---|
| 101 | C001 | Hyderabad | 2025-01-01 | 2026-06-30 | N |
| 205 | C001 | Bangalore | 2026-07-01 | NULL | Y |

The same business customer has two warehouse keys.

This allows historical facts to continue pointing to the correct version.

---

# 16. Primary Key and Foreign Key Relationship

A typical relationship:

```text
dim_customer
-------------
customer_key  ← Primary Key


             customer_key
                   ↑
                   │
fact_sales
-------------
customer_key  ← Foreign Key
```

Similarly:

```text
dim_product
-----------
product_key ← PK

       ↑
       │
product_key
       │
fact_sales
-----------
product_key ← FK
```

---

# 17. Star Schema

A **star schema** is a dimensional model where:

- A central fact table exists.
- Dimension tables surround the fact table.
- Dimensions connect directly to the fact.
- Dimensions generally do not need to be normalized into multiple related tables.

Example:

```text
                    dim_date
                       │
                       │
                       ▼
dim_customer ───── fact_sales ───── dim_product
                       ▲
                       │
                       │
                   dim_store
```

It looks like a star.

Therefore:

```text
STAR SCHEMA
```

---

# 18. Complete Star Schema Example

Suppose we build a retail warehouse.

Central fact:

```text
fact_sales
```

Dimensions:

```text
dim_customer
dim_product
dim_date
dim_store
dim_employee
```

Architecture:

```text
                         dim_date
                            │
                            │
                            ▼
dim_customer ──────── fact_sales ──────── dim_product
                            │
                            │
                            ▼
                        dim_store
                            │
                            │
                            ▼
                       dim_employee
```

The fact table is the center.

Dimensions provide descriptive context.

---

# 19. Star Schema Tables

## Fact

```text
fact_sales
----------
sales_key
date_key
customer_key
product_key
store_key
employee_key
quantity
unit_price
discount_amount
sales_amount
cost_amount
profit_amount
```

## Customer Dimension

```text
dim_customer
------------
customer_key
customer_id
customer_name
gender
city
state
country
customer_segment
```

## Product Dimension

```text
dim_product
-----------
product_key
product_id
product_name
brand
category
subcategory
color
size
```

## Date Dimension

```text
dim_date
--------
date_key
full_date
day
month
month_name
quarter
year
week
day_of_week
```

## Store Dimension

```text
dim_store
---------
store_key
store_id
store_name
city
state
region
```

---

# 20. How Tables Are Connected

The fact table contains foreign keys.

Example:

```text
fact_sales
```

| sales_key | date_key | customer_key | product_key | store_key | quantity | sales_amount |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 20260830 | 101 | 501 | 10 | 2 | 100000 |
| 2 | 20260830 | 102 | 502 | 10 | 1 | 50000 |
| 3 | 20260831 | 101 | 503 | 11 | 3 | 15000 |

The keys point to dimensions.

```text
fact_sales.customer_key
        ↓
dim_customer.customer_key

fact_sales.product_key
        ↓
dim_product.product_key

fact_sales.date_key
        ↓
dim_date.date_key

fact_sales.store_key
        ↓
dim_store.store_key
```

---

# 21. Example Join

Suppose we want:

> Total sales by customer.

SQL:

```sql
SELECT
    c.customer_name,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_customer c
    ON f.customer_key = c.customer_key
GROUP BY c.customer_name;
```

The fact table provides:

```text
sales_amount
```

The dimension provides:

```text
customer_name
```

---

# 22. Sales by Product Category

Question:

> How much did we sell for each product category?

```sql
SELECT
    p.category,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_product p
    ON f.product_key = p.product_key
GROUP BY p.category;
```

Fact:

```text
sales_amount
```

Dimension:

```text
category
```

---

# 23. Sales by Month

Question:

> What were sales by month?

```sql
SELECT
    d.year,
    d.month,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_date d
    ON f.date_key = d.date_key
GROUP BY
    d.year,
    d.month;
```

This is why a date dimension is extremely useful.

---

# 24. Sales by Customer and Product

Question:

> How much did each customer spend on each product category?

```sql
SELECT
    c.customer_name,
    p.category,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_customer c
    ON f.customer_key = c.customer_key
JOIN dim_product p
    ON f.product_key = p.product_key
GROUP BY
    c.customer_name,
    p.category;
```

This is a classic star-schema query.

---

# 25. PySpark Example

Load dimensions:

```python
customer_df = spark.table("prod.gold.dim_customer")
product_df = spark.table("prod.gold.dim_product")
date_df = spark.table("prod.gold.dim_date")
sales_df = spark.table("prod.gold.fact_sales")
```

Join:

```python
result = (
    sales_df
    .join(
        customer_df,
        sales_df.customer_key == customer_df.customer_key
    )
    .join(
        product_df,
        sales_df.product_key == product_df.product_key
    )
)
```

Aggregate:

```python
from pyspark.sql.functions import sum

result = (
    result
    .groupBy("customer_name", "category")
    .agg(
        sum("sales_amount").alias("total_sales")
    )
)
```

---

# 26. Fact-to-Dimension Relationship

Typically:

```text
One dimension row
        ↓
Many fact rows
```

For example:

```text
dim_customer
customer_key = 101
```

could correspond to:

```text
fact_sales
----------------
customer_key = 101
customer_key = 101
customer_key = 101
customer_key = 101
...
```

Therefore:

```text
dim_customer 1 ──────── * fact_sales
```

This is generally a:

```text
One-to-Many
```

relationship.

---

# 27. Product Relationship

One product can appear in many sales transactions.

```text
dim_product
product_key = 501
       │
       ├── fact_sales row 1
       ├── fact_sales row 2
       ├── fact_sales row 3
       └── fact_sales row 4
```

Therefore:

```text
dim_product 1 : N fact_sales
```

---

# 28. Date Relationship

One date can have many sales.

```text
dim_date
date_key = 20260830
       │
       ├── Sale 1
       ├── Sale 2
       ├── Sale 3
       └── Sale 4
```

Therefore:

```text
dim_date 1 : N fact_sales
```

---

# 29. Fact Table Types

There are several important types of fact tables.

## 29.1 Transaction Fact Table

Records individual business events.

Example:

```text
fact_sales
```

One row could represent:

```text
one product sold in one order
```

Typical columns:

```text
order_id
product_key
customer_key
date_key
quantity
sales_amount
```

---

# 30. Periodic Snapshot Fact Table

Stores the state of a business process at regular intervals.

Example:

```text
daily account balance
```

Grain:

```text
One row per customer per day.
```

Example:

| date_key | customer_key | account_balance |
|---:|---:|---:|
| 20260828 | 101 | 10000 |
| 20260829 | 101 | 12000 |
| 20260830 | 101 | 11500 |

This is useful for:

- Daily inventory
- Daily account balances
- Daily outstanding orders
- Daily website metrics

---

# 31. Accumulating Snapshot Fact Table

Tracks a process through multiple milestones.

Example:

```text
Order fulfillment
```

Possible dates:

```text
order_date
payment_date
packing_date
shipping_date
delivery_date
```

Example:

| order_key | order_date | payment_date | shipping_date | delivery_date |
|---:|---|---|---|---|
| 1001 | Aug 1 | Aug 1 | Aug 2 | Aug 5 |

This is useful when a business process has a known beginning and end.

Examples:

- Order processing
- Loan processing
- Insurance claims
- Recruitment process
- Shipment lifecycle

---

# 32. Factless Fact Table

A **factless fact table** contains no numeric measures.

It records that an event or relationship occurred.

Example:

```text
student_attendance
```

Columns:

```text
student_key
course_key
date_key
```

There may be no:

```text
quantity
sales_amount
price
```

Yet the row itself means:

> Student attended this course on this date.

Another example:

```text
customer_campaign
```

could record:

```text
customer_key
campaign_key
date_key
```

meaning:

> This customer was associated with this campaign.

---

# 33. Degenerate Dimension

Sometimes a business identifier is stored directly in the fact table without a separate dimension.

Example:

```text
fact_sales
----------
sales_key
order_number
customer_key
product_key
date_key
sales_amount
```

Here:

```text
order_number
```

may be a **degenerate dimension**.

Why?

Because the order number is useful for analysis but there may be no additional descriptive attributes requiring a separate `dim_order` table.

---

# 34. Conformed Dimensions

A **conformed dimension** is a dimension shared consistently by multiple fact tables.

Example:

```text
dim_date
```

can be used by:

```text
fact_sales
fact_returns
fact_shipments
fact_payments
```

Architecture:

```text
                 dim_date
                /    |    \
               /     |     \
              ▼      ▼      ▼
       fact_sales fact_returns fact_shipments
```

This enables consistent reporting.

For example, "August 2026 sales" and "August 2026 returns" use the same date definitions.

---

# 35. Role-Playing Dimension

The same dimension can play different roles.

The most common example is `dim_date`.

A shipment fact may contain:

```text
order_date_key
ship_date_key
delivery_date_key
```

All three can point to the same date dimension.

```text
                 dim_date
                /   |    \
               /    |     \
              ▼     ▼      ▼
        order_date ship_date delivery_date
              \      |      /
               \     |     /
                fact_order
```

The same physical dimension is playing three roles.

---

# 36. Junk Dimension

A **junk dimension** combines small, low-cardinality flags/attributes that do not logically belong to another dimension.

Example:

```text
is_gift
is_first_order
payment_type
promotion_flag
```

Instead of putting these scattered attributes in the fact table, they can sometimes be combined:

```text
dim_order_flags
---------------
flag_key
is_gift
is_first_order
payment_type
promotion_flag
```

The fact table contains:

```text
flag_key
```

---

# 37. Slowly Changing Dimensions

Dimensions can change over time.

For example:

```text
Customer city = Hyderabad
```

later becomes:

```text
Customer city = Bangalore
```

This is handled using **Slowly Changing Dimensions (SCD)**.

Common types:

```text
SCD Type 0
SCD Type 1
SCD Type 2
SCD Type 3
```

---

# 38. SCD Type 1

Old value is overwritten.

Before:

```text
customer_id = C001
city = Hyderabad
```

After:

```text
customer_id = C001
city = Bangalore
```

No history is maintained.

---

# 39. SCD Type 2

Creates a new dimension record for the changed version.

Example:

| customer_key | customer_id | city | start_date | end_date | current_flag |
|---:|---|---|---|---|---|
| 101 | C001 | Hyderabad | 2025-01-01 | 2026-06-30 | N |
| 205 | C001 | Bangalore | 2026-07-01 | NULL | Y |

This allows historical fact records to remain associated with the correct customer version.

---

# 40. Why Fact Tables Should Not Store All Descriptive Data

Suppose `fact_sales` contains:

```text
customer_name
customer_city
customer_state
product_name
product_category
product_brand
store_name
store_city
store_state
```

This creates significant duplication.

If Mohan makes:

```text
1000 purchases
```

his descriptive information may be repeated 1000 times.

Instead:

```text
fact_sales
    │
    ├── customer_key → dim_customer
    ├── product_key  → dim_product
    └── store_key    → dim_store
```

The descriptions are stored once per relevant dimension version.

---

# 41. Star Schema vs Normalized Model

## Star Schema

```text
             dim_customer
                  │
                  │
dim_date ─── fact_sales ─── dim_product
                  │
                  │
              dim_store
```

Dimensions are generally wider and more denormalized.

For example:

```text
dim_product
-----------
product
brand
category
subcategory
department
```

---

## Normalized Dimension Model

A normalized design might be:

```text
fact_sales
    │
    ▼
dim_product
    │
    ▼
dim_subcategory
    │
    ▼
dim_category
    │
    ▼
dim_department
```

This is more normalized.

---

# 42. Snowflake Schema

When dimensions are further normalized, the model is commonly called a **snowflake schema**.

Example:

```text
                         dim_category
                              │
                              ▼
                       dim_subcategory
                              │
                              ▼
dim_customer ───── fact_sales ───── dim_product
                              │
                              ▼
                         dim_store
```

Compared with star schema:

### Star

```text
fact → dim_product
```

### Snowflake

```text
fact → dim_product → dim_subcategory → dim_category
```

---

# 43. Star Schema vs Snowflake Schema

| Feature | Star Schema | Snowflake Schema |
|---|---|---|
| Fact table | Central | Central |
| Dimensions | More denormalized | More normalized |
| Number of joins | Usually fewer | Usually more |
| Query simplicity | High | Lower |
| Redundancy | Higher | Lower |
| Reporting usability | Excellent | More complex |
| Typical BI performance | Often very good | Depends on engine/model |
| Maintenance | Simple | More complex |
| Design | Simple | More normalized |

---

# 44. Why Star Schema Is Popular in Analytics

A typical analytical query is:

```text
Find total sales
by year
by customer region
by product category
by store
```

With a star schema:

```text
fact_sales
   │
   ├── dim_date
   ├── dim_customer
   ├── dim_product
   └── dim_store
```

The query can join directly to each dimension.

This makes the model easy for:

- BI tools
- Analysts
- SQL developers
- Data engineers
- Reporting systems

---

# 45. Example End-to-End Data Flow

Suppose source systems provide:

```text
Salesforce
ERP
CRM
Point-of-Sale
```

Raw data enters:

```text
Bronze
```

Then transformations occur:

```text
Bronze
   ↓
Silver
   ↓
Gold
```

In Gold, we might create:

```text
dim_customer
dim_product
dim_store
dim_date
fact_sales
```

Architecture:

```text
                 Gold Layer
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
dim_customer   dim_product    dim_store
       \             │             /
        \            │            /
         └──────── fact_sales ───┘
                     ▲
                     │
                  dim_date
```

This is a typical dimensional modeling approach.

---

# 46. Complete Retail Star Schema

```text
                           dim_date
                              │
                              │ date_key
                              ▼
                       ┌─────────────┐
                       │ fact_sales  │
                       ├─────────────┤
                       │ sales_key   │
                       │ date_key    │
                       │ customer_key│
                       │ product_key │
                       │ store_key   │
                       │ employee_key│
                       │ quantity    │
                       │ sales_amt   │
                       │ discount    │
                       │ cost_amt    │
                       │ profit_amt  │
                       └─────────────┘
                        ▲     ▲    ▲
                        │     │    │
             customer_key     │    store_key
                        │     │    │
                        │ product_key
                        │     │    │
                ┌───────┘     └────┘
                │
        dim_customer       dim_product       dim_store
```

---

# 47. Example Data

## dim_customer

| customer_key | customer_id | customer_name | city | state |
|---:|---|---|---|---|
| 101 | C001 | Mohan | Hyderabad | Telangana |
| 102 | C002 | Ravi | Chennai | Tamil Nadu |
| 103 | C003 | Anil | Bangalore | Karnataka |

## dim_product

| product_key | product_id | product_name | category | brand |
|---:|---|---|---|---|
| 501 | P001 | Laptop | Electronics | Dell |
| 502 | P002 | Mouse | Electronics | Logitech |
| 503 | P003 | Chair | Furniture | IKEA |

## dim_store

| store_key | store_id | store_name | city |
|---:|---|---|---|
| 10 | S001 | Hyderabad Store | Hyderabad |
| 11 | S002 | Bangalore Store | Bangalore |

## fact_sales

| sales_key | date_key | customer_key | product_key | store_key | quantity | sales_amount |
|---:|---:|---:|---:|---:|---:|---:|
| 1 | 20260830 | 101 | 501 | 10 | 1 | 75000 |
| 2 | 20260830 | 101 | 502 | 10 | 2 | 4000 |
| 3 | 20260831 | 102 | 503 | 11 | 1 | 15000 |

---

# 48. Reading One Fact Row

Take:

```text
fact_sales
```

row:

```text
1 | 20260830 | 101 | 501 | 10 | 1 | 75000
```

Interpret it:

```text
sales_key    = 1
date_key     = 20260830
customer_key = 101
product_key  = 501
store_key    = 10
quantity     = 1
sales_amount = 75000
```

Now resolve the keys.

### Customer

```text
101 → Mohan
```

### Product

```text
501 → Dell Laptop
```

### Store

```text
10 → Hyderabad Store
```

### Date

```text
20260830 → 30-Aug-2026
```

So the complete business meaning is:

> On 30-Aug-2026, Mohan purchased one Dell laptop from the Hyderabad Store for ₹75,000.

This is the essence of a dimensional model.

---

# 49. Why Fact Tables Usually Have Numeric Measures

Fact tables are optimized for aggregation.

Example:

```sql
SELECT SUM(sales_amount)
FROM fact_sales;
```

Or:

```sql
SELECT
    product_key,
    SUM(quantity)
FROM fact_sales
GROUP BY product_key;
```

The dimension tells us how to interpret the aggregation:

```text
product_key → product/category/brand
```

---

# 50. Dimensions Are Used for Filtering and Grouping

Examples:

```sql
WHERE c.state = 'Telangana'
```

```sql
GROUP BY p.category
```

```sql
GROUP BY d.year, d.month
```

The dimensions provide the attributes used in:

```text
WHERE
GROUP BY
JOIN
ORDER BY
```

while facts commonly provide the measures being aggregated.

---

# 51. Typical Star Schema Query

Question:

> Find total sales for Electronics products sold to customers in Telangana during 2026.

```sql
SELECT
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_customer c
    ON f.customer_key = c.customer_key
JOIN dim_product p
    ON f.product_key = p.product_key
JOIN dim_date d
    ON f.date_key = d.date_key
WHERE c.state = 'Telangana'
  AND p.category = 'Electronics'
  AND d.year = 2026;
```

Notice:

```text
Fact:
sales_amount

Dimensions:
customer → state
product  → category
date     → year
```

---

# 52. Fact Table Size vs Dimension Size

Generally:

```text
Fact table
    ↓
Very large

Dimension tables
    ↓
Usually smaller
```

Example:

```text
fact_sales       → 5 billion rows
dim_customer     → 50 million rows
dim_product      → 500,000 rows
dim_store        → 10,000 rows
dim_date         → ~10,000 rows
```

The date dimension is particularly small.

---

# 53. Fact Table Loading

A common ETL/ELT flow is:

```text
Source
  ↓
Bronze
  ↓
Silver
  ↓
Dimension processing
  ↓
Fact processing
  ↓
Gold
```

Before loading a fact table, dimension keys often need to be resolved.

For example:

```text
Source customer_id = C001
```

Find:

```text
dim_customer
```

and retrieve:

```text
customer_key = 101
```

Then fact row gets:

```text
customer_key = 101
```

---

# 54. Dimension Lookup

Source:

```text
customer_id = C001
```

Dimension:

| customer_key | customer_id | customer_name |
|---:|---|---|
| 101 | C001 | Mohan |

Fact:

```text
customer_key = 101
```

This process is often called a:

```text
Dimension lookup
```

---

# 55. Unknown Dimension Member

Sometimes a fact arrives before its dimension record.

For example:

```text
fact customer_id = C999
```

but:

```text
C999
```

does not yet exist in `dim_customer`.

A warehouse may use an **Unknown** member.

Example:

```text
customer_key = -1
customer_id  = UNKNOWN
customer_name = Unknown Customer
```

Then the fact can still be loaded:

```text
fact_sales.customer_key = -1
```

Later, the dimension can be corrected according to the data pipeline's design.

---

# 56. Fact Table Without a Direct Dimension

Not every value requires a dimension.

Example:

```text
fact_sales
----------
order_number
customer_key
product_key
date_key
quantity
sales_amount
```

`order_number` may remain directly in the fact table as a degenerate dimension.

Do not create dimensions for every single column.

---

# 57. Fact Table Design Checklist

Before designing a fact table, determine:

### 1. Business process

What process are we measuring?

```text
Sales
Orders
Payments
Shipments
```

### 2. Grain

What does one row represent?

```text
One order line
```

### 3. Dimensions

Which dimensions describe the event?

```text
Customer
Product
Date
Store
```

### 4. Measures

What measurements are required?

```text
Quantity
Revenue
Cost
Discount
Profit
```

### 5. Keys

Which dimension keys are required?

```text
customer_key
product_key
date_key
store_key
```

---

# 58. Dimension Design Checklist

For each dimension determine:

### Business key

Example:

```text
customer_id
```

### Surrogate key

Example:

```text
customer_key
```

### Descriptive attributes

Example:

```text
customer_name
city
state
country
segment
```

### Historical behavior

Determine whether attributes should:

```text
Never change
Overwrite
Maintain history
```

This determines the SCD strategy.

---

# 59. Fact vs Dimension — Final Comparison

| Property | Fact Table | Dimension Table |
|---|---|---|
| Purpose | Stores business events | Describes business entities |
| Data type | Mostly measures + keys | Mostly descriptive attributes |
| Size | Usually very large | Usually smaller |
| Growth | Very fast | Usually slower |
| Keys | Foreign keys | Primary/surrogate key |
| Measures | Yes | Usually no |
| Used for | Aggregation | Filtering/grouping |
| Example | `fact_sales` | `dim_customer` |
| Grain | Business event | Entity/version |
| Historical tracking | Usually through dimension keys | Often uses SCD |
| Typical relationship | Many | One side of 1:N |

---

# 60. Star Schema — Final Mental Model

Remember this picture:

```text
                         DIMENSION
                         dim_date
                            │
                            │
                            ▼
DIMENSION              FACT TABLE              DIMENSION
dim_customer ───────► fact_sales ◄────────── dim_product
                            ▲
                            │
                            │
                         dim_store
                         DIMENSION
```

The fact table is the center.

Dimensions surround it.

That is why it is called a:

# ⭐ Star Schema

---

# 61. The One-Sentence Rule

If you remember only one thing:

> **Fact tables store measurable business events at a defined grain, while dimension tables store descriptive context used to analyze those events.**

And the connection is generally:

```text
Dimension primary/surrogate key
              ↓
Fact foreign key
```

For example:

```text
dim_customer.customer_key
              ↑
              │
fact_sales.customer_key
```

---

# 62. Interview Answer

If an interviewer asks:

> Explain fact and dimension tables and star schema.

A strong answer is:

> A fact table stores measurable business events at a clearly defined grain. It generally contains foreign keys to dimensions and measures such as quantity, revenue, cost and discount. Dimension tables contain descriptive attributes such as customer, product, store and date information. The fact table connects to dimensions using foreign keys referencing dimension surrogate keys, generally creating one-to-many relationships from a dimension to the fact. When a central fact table is surrounded by directly connected, generally denormalized dimension tables, the model is called a star schema. For example, `fact_sales` can connect to `dim_customer`, `dim_product`, `dim_date` and `dim_store`. Analysts can then join these dimensions to the fact table to calculate metrics such as total sales by customer, product category, month or region.

---

# 63. Quick Revision Diagram

```text
                    ┌──────────────┐
                    │  dim_date    │
                    └──────┬───────┘
                           │
                           │
┌──────────────┐     ┌─────▼────────┐     ┌──────────────┐
│dim_customer  │────►│  fact_sales  │◄────│ dim_product  │
└──────────────┘     └─────┬────────┘     └──────────────┘
                           │
                           │
                    ┌──────▼───────┐
                    │  dim_store   │
                    └──────────────┘

FACT:
- Business events
- Measures
- Foreign keys
- Large number of rows

DIMENSIONS:
- Descriptive information
- Surrogate keys
- Attributes
- Used for filtering/grouping

STAR SCHEMA:
- Fact in center
- Dimensions around fact
- Direct fact-to-dimension relationships
```

---

# 64. Key Terms to Know for Interviews

Make sure you understand these terms together:

```text
Fact Table
Dimension Table
Grain
Measure
Additive Fact
Semi-Additive Fact
Non-Additive Fact
Surrogate Key
Natural/Business Key
Foreign Key
Star Schema
Snowflake Schema
Conformed Dimension
Role-Playing Dimension
Junk Dimension
Degenerate Dimension
Factless Fact Table
Transaction Fact
Periodic Snapshot
Accumulating Snapshot
Slowly Changing Dimension
SCD Type 1
SCD Type 2
Unknown Dimension Member
Dimension Lookup
```

These concepts form the foundation of **dimensional modeling and data warehouse design**.
