Yes. Below is a **detailed, option-by-option reference** for Databricks YAML, focused on **Jobs and Pipelines**, including **what each option does, when to use it, when not to use it, and realistic PySpark/Data Engineering examples**.

> **Terminology update:** Databricks Asset Bundles are now called **Declarative Automation Bundles**. The YAML structure and `databricks bundle` commands remain compatible. ([Databricks Documentation][1])

I’ll structure it as a Markdown document you can save as:

`databricks_jobs_pipelines_yaml_detailed.md`

---

# Databricks Jobs and Pipelines YAML — Detailed Guide

## 1. What are Databricks YAML files?

Databricks allows you to define resources such as:

* Jobs
* Pipelines
* Notebooks
* Python tasks
* SQL tasks
* Pipeline tasks
* Job clusters
* Serverless compute
* Schedules
* Notifications
* Permissions
* Variables
* Development/production targets

as code using YAML.

The overall project is called a **Declarative Automation Bundle**.

The basic structure is:

```text
my_databricks_project/
│
├── databricks.yml
│
├── resources/
│   ├── customer_job.job.yml
│   └── customer_pipeline.pipeline.yml
│
├── src/
│   ├── bronze.py
│   ├── silver.py
│   └── gold.py
│
└── tests/
```

Databricks officially supports separating resource definitions into additional YAML files and including them from `databricks.yml`. ([Databricks Documentation][2])

---

# 2. The three most important YAML files

A typical project has:

```text
databricks.yml
        │
        ├── Job YAML
        │
        └── Pipeline YAML
```

For example:

```text
databricks.yml
resources/
    customer_job.job.yml
    customer_pipeline.pipeline.yml
```

---

# 3. Main `databricks.yml`

A simple example:

```yaml
bundle:
  name: customer_data_project

include:
  - resources/*.yml

targets:

  dev:
    mode: development
    default: true

    workspace:
      host: https://<dev-workspace-url>

  prod:
    mode: production

    workspace:
      host: https://<prod-workspace-url>
```

---

# 4. `bundle`

```yaml
bundle:
  name: customer_data_project
```

## What does it mean?

Defines the logical name of the entire bundle/project.

Think:

```text
Bundle
  ↓
customer_data_project
```

## When needed?

**Required.**

A bundle configuration must have one top-level `bundle` mapping and a bundle name. ([Databricks Documentation][3])

## When not needed?

You don't omit it from `databricks.yml`.

---

# 5. `include`

```yaml
include:
  - resources/*.yml
```

This tells Databricks:

> Load the YAML files under `resources`.

For example:

```text
resources/
├── customer_job.job.yml
├── customer_pipeline.pipeline.yml
└── monitoring_job.job.yml
```

will all be included.

## Why use it?

Instead of putting everything into one huge file:

```text
databricks.yml
```

you separate resources.

## Recommended

```yaml
include:
  - resources/*.yml
```

## When not needed?

If everything is defined directly in `databricks.yml`.

But for a real project, separating resources is usually much cleaner.

Databricks requires additional configuration files to be referenced through `include`. ([Databricks Documentation][3])

---

# 6. `targets`

Targets represent environments.

Typical:

```text
DEV
TEST
PROD
```

Example:

```yaml
targets:

  dev:
    mode: development

  prod:
    mode: production
```

## Why?

The same YAML can be deployed to different environments.

```text
Git
 │
 ├── DEV
 │
 └── PROD
```

---

# 7. `mode`

You commonly see:

```yaml
mode: development
```

or:

```yaml
mode: production
```

### Development

```yaml
dev:
  mode: development
```

Useful when:

* Developing
* Testing
* Iterating
* Personal development
* Temporary resources

### Production

```yaml
prod:
  mode: production
```

Useful when:

* Deploying production workloads
* CI/CD
* Controlled production deployments

Databricks documents target modes and production behavior as part of bundle deployment configuration. ([Databricks Documentation][3])

---

# 8. `default`

```yaml
targets:

  dev:
    default: true
```

Means:

> Use `dev` when no target is explicitly specified.

For example:

```bash
databricks bundle deploy
```

can use the default target.

Without a default target, you can explicitly specify:

```bash
databricks bundle deploy -t dev
```

Databricks recommends targets and documents `default: true` for selecting the default bundle target. ([Databricks Documentation][3])

---

# 9. Workspace

```yaml
workspace:
  host: https://<workspace-url>
```

Specifies the Databricks workspace where the bundle is deployed.

Typical:

```text
DEV Workspace
PROD Workspace
```

Example:

```yaml
targets:

  dev:
    workspace:
      host: https://dev-workspace.cloud.databricks.com

  prod:
    workspace:
      host: https://prod-workspace.cloud.databricks.com
```

---

# 10. Variables

Variables prevent hard-coding.

Example:

```yaml
variables:

  catalog:
    default: dev

  bronze_schema:
    default: bronze
```

Then:

```yaml
catalog: ${var.catalog}
```

This is extremely useful for:

```text
DEV
TEST
PROD
```

---

# 11. Target-specific variables

Example:

```yaml
variables:

  catalog:
    default: dev

targets:

  dev:
    variables:
      catalog: dev

  prod:
    variables:
      catalog: prod
```

Then:

```yaml
catalog: ${var.catalog}
```

becomes:

```text
DEV → dev
PROD → prod
```

Target settings take precedence over top-level settings. ([Databricks Documentation][3])

---

# PART 1 — JOB YAML

# 12. What is a Databricks Job?

A Job is primarily an **orchestration/workflow** mechanism.

Example:

```text
Job
 │
 ├── Extract
 │
 ├── Bronze
 │
 ├── Silver
 │
 └── Gold
```

A Job can contain multiple tasks and dependencies.

---

# 13. Basic Job YAML

File:

```text
resources/customer_job.job.yml
```

```yaml
resources:

  jobs:

    customer_job:

      name: customer_job

      tasks:

        - task_key: bronze

          notebook_task:
            notebook_path: ../src/bronze.py
```

---

# 14. `resources`

```yaml
resources:
```

This is where Databricks resources are defined.

Examples:

```yaml
resources:
  jobs:
  pipelines:
```

There are many other supported resource types as well. ([Databricks Documentation][2])

---

# 15. `jobs`

```yaml
resources:

  jobs:
```

Means:

> The following definitions are Databricks Jobs.

---

# 16. Job resource key

```yaml
jobs:

  customer_job:
```

`customer_job` is the **bundle resource key**.

Example:

```yaml
customer_job:
```

is different from:

```yaml
name: Customer Production ETL
```

The first is the bundle key; the second is the Databricks Job's display name.

---

# 17. `name`

```yaml
name: Customer Production ETL
```

This is the Job name visible in Databricks.

Example:

```yaml
resources:
  jobs:

    customer_job:

      name: Customer Production ETL
```

---

# 18. `tasks`

```yaml
tasks:
```

Defines the tasks that the Job executes.

Example:

```yaml
tasks:

  - task_key: bronze

  - task_key: silver

  - task_key: gold
```

---

# 19. `task_key`

```yaml
task_key: bronze
```

Uniquely identifies a task within the Job.

Example:

```text
bronze
silver
gold
```

You use it when defining dependencies.

---

# 20. `notebook_task`

```yaml
notebook_task:
  notebook_path: ../src/bronze.py
```

This tells the Job to execute a notebook.

Example:

```yaml
- task_key: bronze

  notebook_task:
    notebook_path: ../src/bronze.ipynb
```

## Use when

Your logic is in a Databricks notebook.

## Don't use when

You want to execute:

* Python script
* Python wheel
* SQL file
* dbt task
* Pipeline

Use the corresponding task type instead.

---

# 21. Notebook parameters

Example:

```yaml
- task_key: bronze

  notebook_task:

    notebook_path: ../src/bronze.ipynb

    base_parameters:

      source: salesforce
      environment: dev
```

Then PySpark notebook code can retrieve them:

```python
dbutils.widgets.get("source")
```

---

# 22. `depends_on`

Example:

```yaml
- task_key: silver

  depends_on:
    - task_key: bronze

  notebook_task:
    notebook_path: ../src/silver.ipynb
```

Means:

```text
bronze
   ↓
silver
```

---

# 23. Multiple dependencies

```yaml
- task_key: gold

  depends_on:
    - task_key: customer_silver
    - task_key: order_silver
```

Graph:

```text
customer_silver ──┐
                   ├──> gold
order_silver ──────┘
```

Gold starts after both dependencies satisfy the task dependency conditions.

---

# 24. Complete Bronze → Silver → Gold Job

```yaml
resources:

  jobs:

    customer_etl:

      name: customer_etl

      tasks:

        - task_key: bronze

          notebook_task:
            notebook_path: ../src/bronze.py

        - task_key: silver

          depends_on:
            - task_key: bronze

          notebook_task:
            notebook_path: ../src/silver.py

        - task_key: gold

          depends_on:
            - task_key: silver

          notebook_task:
            notebook_path: ../src/gold.py
```

Execution:

```text
bronze
  ↓
silver
  ↓
gold
```

---

# 25. Parallel Tasks

```yaml
tasks:

  - task_key: bronze
    ...

  - task_key: customer_silver

    depends_on:
      - task_key: bronze

  - task_key: order_silver

    depends_on:
      - task_key: bronze
```

Graph:

```text
             bronze
                │
          ┌─────┴─────┐
          ▼           ▼
      customer       order
       silver        silver
```

## Use when

Tasks don't depend on each other.

## Benefit

They can execute independently rather than unnecessarily waiting for one another.

---

# 26. `job_clusters`

Example:

```yaml
job_clusters:

  - job_cluster_key: etl_cluster

    new_cluster:

      spark_version: 15.4.x-scala2.12
      node_type_id: <node-type>
      num_workers: 2
```

Then task:

```yaml
job_cluster_key: etl_cluster
```

---

# 27. Why use `job_clusters`?

Suppose:

```text
Bronze
Silver
Gold
```

all need the same cluster configuration.

Instead of repeating:

```yaml
new_cluster:
```

for every task, define a reusable Job cluster configuration.

---

# 28. When NOT to use a Job cluster

Don't automatically create a custom cluster if:

* Serverless is appropriate
* Workload is small
* You don't need custom cluster configuration
* Organization standardizes on serverless

Databricks currently supports serverless Jobs, and for a notebook task you can configure a serverless Job without a cluster definition. ([Databricks Documentation][4])

---

# 29. Serverless Job

Example:

```yaml
resources:

  jobs:

    serverless_job:

      name: serverless_job

      tasks:

        - task_key: bronze

          notebook_task:
            notebook_path: ../src/bronze.ipynb
```

No cluster is explicitly defined.

## Use when

* Serverless is enabled
* You don't need special cluster configuration
* You want simpler compute management

## Don't use when

You specifically require:

* Custom cluster configuration
* Specific infrastructure settings
* Features unavailable in serverless for your workload

---

# 30. `max_retries`

Example:

```yaml
- task_key: bronze

  max_retries: 2

  notebook_task:
    notebook_path: ../src/bronze.ipynb
```

Execution:

```text
Attempt 1 → Failure
Attempt 2 → Failure
Attempt 3 → Success
```

## Use when

Failures are potentially transient.

Examples:

* Temporary infrastructure failure
* Network issue
* Temporary service failure

## Don't use blindly

If the code has a deterministic bug:

```text
SyntaxError
AnalysisException
Bad SQL
Wrong column
```

retrying usually doesn't fix it.

---

# 31. `retry_on_timeout`

Example:

```yaml
retry_on_timeout: true
```

Useful when timeouts can be transient.

Don't use it blindly for expensive workloads where retries would create unnecessary compute cost.

---

# 32. `timeout_seconds`

Example:

```yaml
timeout_seconds: 3600
```

Means:

```text
Maximum task runtime = 1 hour
```

## Use when

You know the expected maximum runtime.

## Useful for

Preventing a stuck task from running indefinitely.

---

# 33. Job schedule

Example:

```yaml
schedule:

  quartz_cron_expression: "0 0 2 * * ?"

  timezone_id: Asia/Kolkata
```

Conceptually:

```text
Every day
   ↓
2 AM
   ↓
Run Job
```

## Use when

You need scheduled batch processing.

Examples:

```text
Daily ETL
Hourly ingestion
Weekly reporting
```

---

# 34. `trigger.periodic`

Example:

```yaml
trigger:

  periodic:

    interval: 1
    unit: HOURS
```

Means:

```text
Every 1 hour
```

Databricks documents periodic triggers as a Job configuration option. ([Databricks Documentation][4])

---

# 35. Schedule vs periodic trigger

### Schedule

Use when you care about an exact schedule.

```text
Every day at 2 AM
```

### Periodic

Use when you want:

```text
Every N hours/minutes
```

---

# 36. Job notifications

Example:

```yaml
email_notifications:

  on_failure:
    - data-team@example.com
```

Possible use cases include:

```text
Job fails
   ↓
Email team
```

## Use when

Production monitoring is important.

## Don't use

For every tiny development Job unless needed.

---

# 37. `queue`

Example:

```yaml
queue:
  enabled: true
```

This controls Job run queuing behavior.

Databricks' current bundle examples show:

```yaml
queue:
  enabled: true
```

as a supported Job configuration. ([Databricks Documentation][5])

## Why useful?

If multiple runs compete for available compute, queuing can prevent unnecessary concurrent execution behavior.

---

# 38. `run_as`

Production deployments often need a controlled identity.

Conceptually:

```yaml
run_as:
  service_principal_name: <service-principal>
```

This is useful because:

```text
Developer
    ↓
Deploy
    ↓
Production Job
    ↓
Runs as controlled identity
```

rather than relying on an individual's identity.

Databricks supports `run_as` at target level. ([Databricks Documentation][3])

---

# 39. Python script task

Example:

```yaml
tasks:

  - task_key: python_task

    spark_python_task:

      python_file: ../src/customer.py
```

## Use when

You have:

```text
customer.py
```

rather than a notebook.

## Don't use when

Your logic is specifically maintained as a notebook.

---

# 40. SQL task

Jobs can also execute supported SQL task types.

Conceptually:

```yaml
tasks:

  - task_key: sql_task

    sql_task:
      ...
```

Use SQL tasks when the transformation is naturally SQL-based.

Don't force a SQL task when the workload requires substantial PySpark logic.

---

# 41. Pipeline task

This is extremely important.

```yaml
tasks:

  - task_key: refresh_pipeline

    pipeline_task:

      pipeline_id: ${resources.pipelines.customer_pipeline.id}
```

This means:

```text
Job
 ↓
Refresh Pipeline
```

---

# 42. Job Parameters vs Notebook Parameters

These are often confused.

A Job can have:

```yaml
parameters:
```

while an individual notebook task can have:

```yaml
base_parameters:
```

Conceptually:

```text
Job parameters
      ↓
Task
      ↓
Notebook parameters
      ↓
PySpark
```

Use the appropriate mechanism based on whether the value is a Job-level parameter or task-specific input.

---

# PART 2 — PIPELINE YAML

# 43. What is a Pipeline?

Databricks' current terminology is **Lakeflow / Spark Declarative Pipelines**.

A pipeline describes a declarative data processing workflow.

Example:

```text
Raw
 ↓
Bronze
 ↓
Silver
 ↓
Gold
```

The pipeline focuses on the data flow itself.

---

# 44. Basic Pipeline YAML

```yaml
resources:

  pipelines:

    customer_pipeline:

      name: customer_pipeline

      catalog: workspace

      target: bronze

      libraries:

        - notebook:
            path: ../src/customer_pipeline.py
```

---

# 45. `pipelines`

```yaml
resources:

  pipelines:
```

means:

> Define pipeline resources.

---

# 46. Pipeline resource key

```yaml
pipelines:

  customer_pipeline:
```

This is the bundle resource key.

---

# 47. Pipeline `name`

```yaml
name: Customer Pipeline
```

The display name of the pipeline.

---

# 48. `catalog`

Example:

```yaml
catalog: ${var.catalog}
```

Specifies the Unity Catalog catalog used by the pipeline.

Example:

```text
dev
```

or:

```text
prod
```

---

# 49. `target`

Example:

```yaml
target: bronze
```

The target schema/database setting for the pipeline, depending on the pipeline model and workspace configuration.

A common environment-aware setup is:

```yaml
catalog: ${var.catalog}
target: ${var.schema}
```

---

# 50. `libraries`

Example:

```yaml
libraries:

  - notebook:
      path: ../src/pipeline/customer.py
```

This tells the pipeline where its source code is located.

Multiple source files can be defined:

```yaml
libraries:

  - notebook:
      path: ../src/pipeline/customer.py

  - notebook:
      path: ../src/pipeline/orders.py

  - notebook:
      path: ../src/pipeline/products.py
```

---

# 51. Pipeline with multiple transformations

```yaml
resources:

  pipelines:

    customer_pipeline:

      name: customer_pipeline

      catalog: ${var.catalog}

      target: ${var.schema}

      libraries:

        - notebook:
            path: ../src/pipeline/bronze.py

        - notebook:
            path: ../src/pipeline/silver.py

        - notebook:
            path: ../src/pipeline/gold.py
```

Conceptually:

```text
Pipeline
   │
   ├── Bronze
   │
   ├── Silver
   │
   └── Gold
```

---

# 52. `serverless: true`

Example:

```yaml
resources:

  pipelines:

    customer_pipeline:

      name: customer_pipeline

      serverless: true
```

This tells Databricks to use serverless pipeline compute.

Databricks' current examples use:

```yaml
serverless: true
```

for serverless pipelines. ([Databricks Documentation][4])

---

# 53. When to use serverless Pipeline?

Use it when:

* Your workspace supports it
* You don't need custom compute infrastructure
* You want simpler operational management

Don't use it when your workload has a requirement that is incompatible with serverless.

---

# 54. Pipeline configuration

Example:

```yaml
configuration:

  source_path: /Volumes/workspace/raw/raw_flight_data

  environment: dev
```

Configuration values can be consumed by the pipeline runtime/application according to the pipeline's configuration mechanisms.

---

# 55. Pipeline environment dependencies

For supported serverless pipeline scenarios, dependencies can be specified using an environment configuration.

Databricks' current example:

```yaml
environment:

  dependencies:
    - 'dist/*.whl'
```

This is particularly useful when your pipeline depends on custom Python packages. ([Databricks Documentation][4])

---

# 56. Pipeline development mode

A development-oriented pipeline configuration can be used during development.

However, don't simply copy development settings into production.

Prefer:

```text
dev target
   ↓
development configuration

prod target
   ↓
production configuration
```

---

# 57. Pipeline → Job relationship

A very common production architecture is:

```text
             Job
              │
              ▼
        Pipeline Refresh
              │
              ▼
          Pipeline
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
    Bronze  Silver  Gold
```

Pipeline:

```yaml
resources:

  pipelines:

    customer_pipeline:

      name: customer_pipeline

      serverless: true

      catalog: ${var.catalog}

      target: ${var.schema}

      libraries:
        - notebook:
            path: ../src/pipeline/customer.py
```

Job:

```yaml
resources:

  jobs:

    customer_pipeline_job:

      name: customer_pipeline_job

      tasks:

        - task_key: refresh_pipeline

          pipeline_task:

            pipeline_id: ${resources.pipelines.customer_pipeline.id}
```

Databricks' official bundle pipeline example uses this pattern. ([Databricks Documentation][6])

---

# PART 3 — IMPORTANT JOB OPTIONS

# 58. Job options cheat sheet

| Option                | Purpose                       | Use when                    | Avoid when                       |
| --------------------- | ----------------------------- | --------------------------- | -------------------------------- |
| `name`                | Job display name              | Always                      | Never                            |
| `tasks`               | Define workflow tasks         | Always for Jobs             | Never                            |
| `task_key`            | Task identifier               | Always                      | Never                            |
| `depends_on`          | DAG dependency                | Tasks depend on others      | Tasks are independent            |
| `notebook_task`       | Run notebook                  | Logic is notebook-based     | Python/SQL/etc. task is better   |
| `spark_python_task`   | Run Python file               | Code is `.py`               | Notebook is preferred            |
| `pipeline_task`       | Refresh pipeline              | Job orchestrates pipeline   | No pipeline                      |
| `job_clusters`        | Define reusable job compute   | Custom compute needed       | Serverless is sufficient         |
| `max_retries`         | Retry task                    | Transient failures possible | Deterministic code errors        |
| `retry_on_timeout`    | Retry timeout                 | Timeout may be transient    | Retries are expensive/unhelpful  |
| `timeout_seconds`     | Maximum runtime               | Prevent runaway jobs        | Runtime is highly unpredictable  |
| `schedule`            | Calendar schedule             | Exact schedule required     | Event/periodic trigger is better |
| `trigger.periodic`    | Periodic execution            | Every N units               | Exact calendar schedule          |
| `email_notifications` | Failure/success notifications | Production monitoring       | Temporary development            |
| `queue`               | Control queued runs           | Run concurrency matters     | Not needed                       |
| `run_as`              | Execution identity            | Production/security         | Personal dev testing             |

---

# 59. Job task types — when to choose what

```text
Need to execute...
        │
        ├── Notebook ────────> notebook_task
        │
        ├── Python file ─────> spark_python_task
        │
        ├── Python package ──> python_wheel_task
        │
        ├── Pipeline ─────────> pipeline_task
        │
        ├── SQL ──────────────> SQL task
        │
        └── dbt ──────────────> dbt task
```

The exact task configuration varies by task type and compute model.

---

# 60. When should you use Notebook Task?

Use:

```yaml
notebook_task:
```

when:

```text
bronze.ipynb
silver.ipynb
gold.ipynb
```

are your actual application units.

Good for:

* Data Engineering notebooks
* Exploration that has become productionized
* Teams already standardized on notebooks

Avoid when your organization wants a Python package/application structure instead.

---

# 61. When should you use Python Script?

Use:

```yaml
spark_python_task:
```

when:

```text
src/
    bronze.py
    silver.py
```

is your application structure.

This is often cleaner for larger software-engineering-oriented projects.

---

# 62. When should you use Python Wheel?

Use a wheel when:

```text
src/
tests/
pyproject.toml
```

and you want:

```text
Build
 ↓
.whl
 ↓
Job
```

This is especially useful for:

* Reusable code
* Unit testing
* CI/CD
* Large projects
* Shared libraries

---

# PART 4 — PIPELINE OPTIONS

# 63. Pipeline options cheat sheet

| Option          | Purpose                  | Use when                               | Avoid when                       |
| --------------- | ------------------------ | -------------------------------------- | -------------------------------- |
| `name`          | Pipeline name            | Always                                 | Never                            |
| `catalog`       | Unity Catalog catalog    | UC pipeline                            | Not applicable                   |
| `target`        | Target schema            | Pipeline target needed                 | Depending on pipeline model      |
| `libraries`     | Source/dependencies      | Pipeline code exists                   | Never omit required source       |
| `serverless`    | Serverless compute       | Supported workload                     | Custom compute required          |
| `configuration` | Pipeline config values   | Runtime configuration                  | Hard-coded values are sufficient |
| `environment`   | Environment/dependencies | Serverless/custom package dependencies | No extra dependency              |
| `development`   | Development behavior     | Development workflow                   | Production                       |
| `continuous`    | Continuous processing    | Need continuously running pipeline     | Batch/triggered processing       |
| `notifications` | Pipeline alerts          | Production monitoring                  | Local experimentation            |

---

# 64. Continuous vs triggered pipeline

Conceptually:

### Triggered

```text
Run
 ↓
Process data
 ↓
Finish
```

Use for:

```text
Daily ETL
Hourly ETL
On-demand processing
```

### Continuous

```text
Pipeline
 ↓
Continuously process incoming data
 ↓
Keep running
```

Use when the workload requires continuous processing.

Don't use continuous mode simply because it sounds more real-time.

If hourly/daily processing is sufficient, triggered processing can be simpler and cheaper.

---

# PART 5 — VARIABLES AND ENVIRONMENTS

# 65. Complete environment-aware example

```yaml
bundle:

  name: flight_project


include:

  - resources/*.yml


variables:

  catalog:

    description: Target catalog

    default: dev

  bronze_schema:

    description: Bronze schema

    default: bronze


targets:

  dev:

    mode: development

    default: true

    variables:

      catalog: dev

    workspace:

      host: https://<dev-workspace>


  prod:

    mode: production

    variables:

      catalog: prod

    workspace:

      host: https://<prod-workspace>
```

---

# 66. Pipeline using variables

```yaml
resources:

  pipelines:

    flight_pipeline:

      name: flight_pipeline

      serverless: true

      catalog: ${var.catalog}

      target: ${var.bronze_schema}

      libraries:

        - notebook:
            path: ../src/pipeline/flight.py
```

Result:

```text
DEV:

dev.bronze

PROD:

prod.bronze
```

---

# PART 6 — COMPLETE REAL-WORLD PROJECT

# 67. Folder structure

```text
flight_data_project/
│
├── databricks.yml
│
├── resources/
│   │
│   ├── flight_pipeline.pipeline.yml
│   │
│   └── flight_job.job.yml
│
├── src/
│   │
│   ├── bronze/
│   │   └── flight_bronze.py
│   │
│   ├── silver/
│   │   └── flight_silver.py
│   │
│   └── gold/
│       └── flight_gold.py
│
└── tests/
```

---

# 68. `databricks.yml`

```yaml
bundle:

  name: flight_data_project


include:

  - resources/*.yml


variables:

  catalog:

    description: Unity Catalog catalog

    default: dev

  bronze_schema:

    description: Bronze schema

    default: bronze


targets:

  dev:

    mode: development

    default: true

    variables:

      catalog: dev

    workspace:

      host: https://<dev-workspace>


  prod:

    mode: production

    variables:

      catalog: prod

    workspace:

      host: https://<prod-workspace>
```

---

# 69. Pipeline YAML

`resources/flight_pipeline.pipeline.yml`

```yaml
resources:

  pipelines:

    flight_pipeline:

      name: flight_pipeline

      serverless: true

      catalog: ${var.catalog}

      target: ${var.bronze_schema}

      libraries:

        - notebook:
            path: ../src/bronze/flight_bronze.py

        - notebook:
            path: ../src/silver/flight_silver.py

        - notebook:
            path: ../src/gold/flight_gold.py
```

---

# 70. Job YAML

`resources/flight_job.job.yml`

```yaml
resources:

  jobs:

    flight_pipeline_job:

      name: flight_pipeline_job

      trigger:

        periodic:

          interval: 1

          unit: HOURS

      email_notifications:

        on_failure:

          - data-team@example.com

      tasks:

        - task_key: refresh_flight_pipeline

          pipeline_task:

            pipeline_id: ${resources.pipelines.flight_pipeline.id}
```

---

# 71. Complete architecture

```text
                         Git
                          │
                          ▼
                   databricks.yml
                          │
              ┌───────────┴───────────┐
              │                       │
        Job YAML                  Pipeline YAML
              │                       │
              ▼                       ▼
        Lakeflow Job           Spark Declarative
                                   Pipeline
              │                       │
              │                  ┌────┼────┐
              │                  ▼    ▼    ▼
              │               Bronze Silver Gold
              │
              └──────> Pipeline refresh
```

---

# PART 7 — DEPLOYMENT

# 72. Validate

Run:

```bash
databricks bundle validate
```

Do this before deployment.

Databricks explicitly recommends validating bundle configuration before deploying or running resources. ([Databricks Documentation][7])

---

# 73. Deploy DEV

```bash
databricks bundle deploy -t dev
```

---

# 74. Run Job

```bash
databricks bundle run -t dev flight_pipeline_job
```

---

# 75. Deploy PROD

After testing:

```bash
databricks bundle deploy -t prod
```

---

# 76. Generate YAML from an existing Databricks resource

If you already created a Job or pipeline manually in the Databricks UI, you don't necessarily have to recreate the YAML from scratch.

Databricks supports:

```bash
databricks bundle generate
```

to generate bundle configuration for existing resources. ([Databricks Documentation][2])

This is extremely useful when learning YAML.

---

# PART 8 — WHAT TO USE IN REAL PROJECTS

## 77. Small PySpark project

Use:

```text
databricks.yml
       │
       └── Job
             │
             └── Notebook
```

Don't over-engineer it with:

* Multiple clusters
* Complex artifacts
* Many variables
* Multiple environments

unless actually required.

---

# 78. Medium Data Engineering project

Use:

```text
databricks.yml

resources/
    jobs
    pipelines

src/
    bronze
    silver
    gold
```

Use:

```text
dev
prod
```

targets.

Use variables for:

```text
catalog
schema
paths
```

---

# 79. Enterprise project

Recommended architecture:

```text
Git
 │
 ▼
CI/CD
 │
 ▼
Bundle
 │
 ├── DEV
 │
 ├── TEST
 │
 └── PROD
 │
      ├── Jobs
      ├── Pipelines
      ├── Unity Catalog
      ├── Service Principal
      └── Monitoring
```

Use:

* Production target
* Controlled `run_as`
* Service principals
* Variables
* CI/CD
* Tests
* Notifications
* Appropriate retries
* Environment-specific configuration
* Reusable Python packages
* Serverless where appropriate

---

# 80. Job vs Pipeline vs Bundle

This is the most important concept.

```text
BUNDLE
│
│  Deployment / IaC
│
├───────────────┐
│               │
▼               ▼
JOB          PIPELINE
│               │
│               │
Workflow        Data processing
│               │
▼               ▼
Tasks          Tables/Data flow
```

### Bundle

**Question it answers:**

> How do I package, version and deploy my Databricks project?

### Job

**Question it answers:**

> What should run, in what order, and when?

### Pipeline

**Question it answers:**

> How should my data transformation pipeline be defined and processed?

---

# 81. Interview scenario

### Requirement

You have:

```text
Salesforce
   ↓
S3
   ↓
Bronze
   ↓
Silver
   ↓
Gold
```

You need the process to run every hour.

### Recommended architecture

```text
Databricks Bundle
       │
       ├── Job
       │     │
       │     └── Pipeline Task
       │
       └── Pipeline
             │
             ├── Bronze
             ├── Silver
             └── Gold
```

Job YAML:

```yaml
resources:

  jobs:

    sales_job:

      name: sales_job

      trigger:

        periodic:
          interval: 1
          unit: HOURS

      tasks:

        - task_key: refresh_sales_pipeline

          pipeline_task:

            pipeline_id: ${resources.pipelines.sales_pipeline.id}
```

Pipeline YAML:

```yaml
resources:

  pipelines:

    sales_pipeline:

      name: sales_pipeline

      serverless: true

      catalog: ${var.catalog}

      target: bronze

      libraries:

        - notebook:
            path: ../src/bronze.py

        - notebook:
            path: ../src/silver.py

        - notebook:
            path: ../src/gold.py
```

---

# 82. When NOT to use a Pipeline

If the requirement is simply:

```text
Run Python
 ↓
Run SQL
 ↓
Send notification
```

you may only need a Job.

Don't create a Pipeline just because the workload is a data-engineering workload.

---

# 83. When NOT to use a Job

If your requirement is fundamentally a declarative data pipeline and doesn't require complex orchestration, don't create unnecessary Job layers.

For example:

```text
Source
 ↓
Bronze
 ↓
Silver
 ↓
Gold
```

may be naturally represented as a pipeline.

---

# 84. When to use BOTH

Use both when:

```text
Scheduling
   +
Orchestration
   +
Declarative data processing
```

are all required.

Example:

```text
Every hour
    ↓
Job
    ↓
Pipeline refresh
    ↓
Bronze
    ↓
Silver
    ↓
Gold
```

This is one of the most useful patterns to understand for Databricks Data Engineering interviews.

---

# 85. Final cheat sheet

```yaml
# Main bundle

bundle:
  name: project

include:
  - resources/*.yml

variables:
  catalog:
    default: dev

targets:

  dev:
    mode: development
    default: true

  prod:
    mode: production
```

### Job

```yaml
resources:

  jobs:

    my_job:

      name: my_job

      tasks:

        - task_key: task1

          notebook_task:
            notebook_path: ../src/task.ipynb
```

### Dependency

```yaml
depends_on:
  - task_key: task1
```

### Job cluster

```yaml
job_clusters:

  - job_cluster_key: cluster

    new_cluster:
      spark_version: ...
      node_type_id: ...
      num_workers: 2
```

### Serverless Job

```yaml
tasks:

  - task_key: task1

    notebook_task:
      notebook_path: ../src/task.ipynb
```

### Schedule

```yaml
schedule:

  quartz_cron_expression: "..."

  timezone_id: Asia/Kolkata
```

### Periodic trigger

```yaml
trigger:

  periodic:

    interval: 1
    unit: HOURS
```

### Retry

```yaml
max_retries: 2
```

### Timeout

```yaml
timeout_seconds: 3600
```

### Pipeline

```yaml
resources:

  pipelines:

    my_pipeline:

      name: my_pipeline

      catalog: ${var.catalog}

      target: bronze

      libraries:

        - notebook:
            path: ../src/pipeline.py
```

### Serverless Pipeline

```yaml
serverless: true
```

### Job → Pipeline

```yaml
pipeline_task:

  pipeline_id: ${resources.pipelines.my_pipeline.id}
```

### Validate

```bash
databricks bundle validate
```

### Deploy

```bash
databricks bundle deploy -t dev
```

### Run

```bash
databricks bundle run -t dev my_job
```

---

## Key takeaway

For your **PySpark/Data Engineering interview preparation**, remember this hierarchy:

```text
                 databricks.yml
                       │
              Declarative Automation
                    Bundle
                       │
          ┌────────────┴────────────┐
          │                         │
         JOB                    PIPELINE
          │                         │
     Orchestration             Data Flow
          │                         │
      ┌───┼────┐              ┌─────┼─────┐
      ▼   ▼    ▼              ▼     ▼     ▼
   Python SQL Notebook      Bronze Silver Gold
          │
          ▼
   Pipeline Task
```

**Bundle = deployment/IaC**

**Job = orchestration**

**Pipeline = declarative data processing**

And the most important production pattern is:

```text
Bundle
  ↓
Job
  ↓
Pipeline Task
  ↓
Pipeline
  ↓
Bronze → Silver → Gold
```

Databricks' official documentation currently provides this Job → Pipeline pattern in its Bundle examples. ([Databricks Documentation][6])

[1]: https://docs.databricks.com/aws/en/dev-tools/bundles/?utm_source=chatgpt.com "What are Declarative Automation Bundles? | Databricks on AWS"
[2]: https://docs.databricks.com/gcp/en/dev-tools/bundles/resources?utm_source=chatgpt.com "Declarative Automation Bundles resources | Databricks on Google Cloud"
[3]: https://docs.databricks.com/aws/en/dev-tools/bundles/reference?utm_source=chatgpt.com "Configuration reference | Databricks on AWS"
[4]: https://docs.databricks.com/aws/en/dev-tools/bundles/examples?utm_source=chatgpt.com "Bundle configuration examples | Databricks on AWS"
[5]: https://docs.databricks.com/aws/en/dev-tools/bundles/workspace-tutorial?utm_source=chatgpt.com "Tutorial: Create and deploy a bundle in the workspace | Databricks on AWS"
[6]: https://docs.databricks.com/aws/en/dev-tools/bundles/pipelines-tutorial?utm_source=chatgpt.com "Develop pipelines with Declarative Automation Bundles | Databricks on AWS"
[7]: https://docs.databricks.com/aws/en/dev-tools/bundles/work-tasks?utm_source=chatgpt.com "Develop Declarative Automation Bundles | Databricks on AWS"
