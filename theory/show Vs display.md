In **Databricks**, both `df.show()` and `display(df)` are used to view a DataFrame, but they behave differently.

| Feature                  | `df.show()`                   | `display(df)`                                  |
| ------------------------ | ----------------------------- | ---------------------------------------------- |
| Origin                   | Apache Spark API              | Databricks-specific                            |
| Output                   | Text/table in notebook output | Rich interactive table                         |
| Default rows             | 20 rows                       | Typically displays a larger interactive result |
| Truncate long values     | Yes, by default               | No/less restrictive visually                   |
| Interactive              | ❌ No                         | ✅ Yes                                       |
| Visualization            | ❌ No                         | ✅ Charts/graphs available                   |
| Sorting/filtering        | ❌ No                         | ✅ Interactive options                       |
| Works outside Databricks | ✅ Yes                        | ❌ Primarily Databricks                      |
| Return value             | `None`                        | Displayed result                               |

### 1. `df.show()`

This is a **Spark DataFrame method**:

```python
df.show()
```

Example output:

```text
+---+-------+
| id|   name|
+---+-------+
|  1|  Mohan|
|  2|  Ravi |
|  3| Priya |
+---+-------+
```

You can control the number of rows:

```python
df.show(50)
```

And prevent truncation:

```python
df.show(20, truncate=False)
```

So:

```python
df.show(5, truncate=False)
```

means **show 5 rows without truncating long column values**.

---

### 2. `display(df)`

`display()` is provided by **Databricks**:

```python
display(df)
```

It renders the DataFrame as an interactive Databricks table.

You can generally:

* Sort columns
* Filter data
* Explore the result
* Create visualizations such as bar charts, line charts, etc.
* Change how the output is presented

For example:

```python
display(df)
```

Then in the Databricks notebook UI, you can switch from a table to a visualization.

### Interview answer ⭐

If asked **"What is the difference between `show()` and `display()` in Databricks?"**, say:

> **`show()` is a Spark DataFrame API method that prints a textual representation of the DataFrame, while `display()` is a Databricks-specific function that provides a richer, interactive visualization of the DataFrame, including filtering, sorting, and charting capabilities.**

One important point: **`display()` is not a Spark API method**. It is a Databricks notebook feature.
