<p align="center">
  <img src="https://raw.githubusercontent.com/dbt-labs/dbt-core/fa1ea14ddfb1d5ae319d5141844910dd53ab2834/etc/dbt-core.svg" alt="dbt logo" width="750"/>
</p>

![version 1.10](https://img.shields.io/badge/📌%20version-1.10-blue?style=flat-square)
![python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)
![category](https://img.shields.io/badge/🏷️%20category-etl--tools-blue?style=flat-square)
[![docs](https://img.shields.io/badge/🔗%20docs-dbt--docs-blue?style=flat-square)](https://docs.getdbt.com)


## 🌟 Overview

- **What it is:** dbt (data build tool) is an open-source framework for transforming raw data into reliable models using SQL and software engineering best practices.
- **Why it matters:** It enables modular, version-controlled, and testable SQL workflows directly in your data warehouse.
- **Ideal Use Cases:** ELT transformation, building data marts, analytics engineering, automated testing and documentation.
- **Main Alternatives:**
  - **Dataform** – GCP-native transformation tool
  - **SQLMesh** – CI/CD-first with advanced diffing
  - **Airflow** – general-purpose orchestration with SQL support


![architecture](https://github.com/dbt-labs/dbt-core/blob/202cb7e51e218c7b29eb3b11ad058bd56b7739de/etc/dbt-transform.png)



## Table of Contents
1. [Quick Start](#quick-start)
2. [Installation & Setup](#installation--setup)
3. [Project Structure](#project-structure)
4. [Core Concepts](#core-concepts)
5. [Model Development](#model-development)
6. [Testing](#testing)
7. [Documentation](#documentation)
8. [Advanced Features](#advanced-features)
9. [Best Practices](#best-practices)
10. [CLI Commands Cheat Sheet](#cli-commands-cheat-sheet)
11. [Jinja & Macros](#jinja--macros)
12. [Performance Optimization](#performance-optimization)
13. [Troubleshooting](#troubleshooting)

---

## Quick Start

### What is dbt?
dbt (data build tool) enables data teams to transform data in their warehouse by simply writing select statements. dbt handles turning these select statements into tables and views with proper dependency management, testing, and documentation.

### Core Philosophy
- **T**ransform data using SQL SELECT statements
- **T**est your data transformations
- **D**ocument your data models
- **V**ersion control everything

---

## Installation & Setup

### Install dbt Core
```bash
# Install via pip (recommended)
pip install dbt-core

# Install with specific adapter (choose one)
pip install dbt-snowflake
pip install dbt-bigquery
pip install dbt-redshift
pip install dbt-postgres
pip install dbt-databricks

# Verify installation
dbt --version
```

### Initialize Project
```bash
# Create new project
dbt init my_project

# Navigate to project
cd my_project

# Set up profiles.yml (connection details)
dbt debug
```

### profiles.yml Configuration
```yaml
my_project:
  outputs:
    dev:
      type: snowflake
      account: your-account
      user: your-username
      password: your-password
      role: your-role
      database: DEV_DB
      warehouse: DEV_WH
      schema: analytics
      threads: 4
    prod:
      type: snowflake
      account: your-account
      user: your-username
      password: your-password
      role: your-role
      database: PROD_DB
      warehouse: PROD_WH
      schema: analytics
      threads: 8
  target: dev
```

---

## Project Structure

```
my_dbt_project/
├── dbt_project.yml          # Project configuration
├── profiles.yml             # Connection profiles (keep secure!)
├── packages.yml             # dbt packages dependencies
├── analyses/                # Analytical SQL files
├── data/                    # Seed files (CSV data)
├── macros/                  # Reusable SQL functions
├── models/                  # dbt models (SQL files)
│   ├── staging/            # Raw data cleaning & standardization
│   ├── intermediate/       # Business logic transformation
│   ├── marts/             # Final business-ready models
│   └── schema.yml         # Tests and documentation
├── seeds/                  # CSV files for reference data
├── snapshots/             # SCD Type 2 tables
├── tests/                 # Custom data tests
├── logs/                  # dbt run logs
├── target/               # Compiled SQL (gitignored)
└── dbt_packages/         # Downloaded packages (gitignored)
```

---

## Core Concepts

### Models
Models are SQL SELECT statements that create tables or views in your warehouse.

#### Model Types
- **View** (default): Virtual table, computed at query time
- **Table**: Materialized data, stored physically
- **Incremental**: Appends/updates only new data
- **Ephemeral**: CTE used by other models, not materialized

### Materializations
```sql
-- In model file or dbt_project.yml
{{ config(materialized='table') }}
{{ config(materialized='view') }}
{{ config(materialized='incremental') }}
{{ config(materialized='ephemeral') }}
```

### Sources
External data that dbt doesn't create, defined in `sources.yml`:
```yaml
sources:
  - name: raw_data
    description: "Raw data from application database"
    tables:
      - name: users
        description: "User information"
        columns:
          - name: id
            description: "Primary key"
            tests:
              - unique
              - not_null
```

### References
```sql
-- Reference another model
SELECT * FROM {{ ref('my_model') }}

-- Reference a source
SELECT * FROM {{ source('raw_data', 'users') }}
```

---

## Model Development

### Basic Model Structure
```sql
-- models/staging/stg_users.sql
{{ config(materialized='view') }}

WITH source_data AS (
    SELECT 
        id,
        email,
        created_at,
        updated_at
    FROM {{ source('raw_data', 'users') }}
)

SELECT 
    id AS user_id,
    email,
    created_at,
    updated_at
FROM source_data
WHERE email IS NOT NULL
```

### Incremental Models
```sql
-- models/marts/user_events.sql
{{ config(
    materialized='incremental',
    unique_key='event_id',
    on_schema_change='fail'
) }}

SELECT 
    event_id,
    user_id,
    event_type,
    created_at
FROM {{ source('events', 'user_events') }}

{% if is_incremental() %}
  WHERE created_at > (SELECT MAX(created_at) FROM {{ this }})
{% endif %}
```

### Model Configuration
```sql
-- In model file
{{ config(
    materialized='table',
    sort='created_at',
    dist='user_id',
    pre_hook="GRANT SELECT ON {{ this }} TO role_read_only",
    post_hook="CREATE INDEX IF NOT EXISTS idx_user_id ON {{ this }} (user_id)"
) }}
```

---

## Testing

### Built-in Tests
```yaml
# models/schema.yml
models:
  - name: users
    columns:
      - name: user_id
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - unique
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['active', 'inactive', 'pending']
```

### Custom Generic Tests
```sql
-- tests/generic/test_positive_values.sql
SELECT *
FROM {{ ref('my_model') }}
WHERE {{ column_name }} <= 0
```

### Singular Tests
```sql
-- tests/assert_user_email_domains.sql
SELECT email
FROM {{ ref('users') }}
WHERE email NOT LIKE '%@company.com'
  AND email NOT LIKE '%@partner.com'
```

### Data Quality Tests
```yaml
models:
  - name: sales_summary
    tests:
      - dbt_utils.expression_is_true:
          expression: "total_revenue >= 0"
      - dbt_utils.not_null_proportion:
          at_least: 0.95
```

---

## Documentation

### Model Documentation
```yaml
# models/schema.yml
models:
  - name: users
    description: "Customer user accounts"
    columns:
      - name: user_id
        description: "Primary key for users"
      - name: email
        description: "User's email address"
      - name: created_at
        description: "Timestamp when user was created"
```

### Generate Documentation
```bash
# Generate docs
dbt docs generate

# Serve docs locally
dbt docs serve --port 8080
```

### Advanced Documentation
```yaml
models:
  - name: user_metrics
    description: |
      ## User Engagement Metrics
      
      This model calculates key user engagement metrics including:
      - Daily active users
      - Session duration
      - Feature adoption rates
      
      ### Business Logic
      - Users are considered active if they have at least one session
      - Sessions timeout after 30 minutes of inactivity
    meta:
      owner: "data-team@company.com"
      tags: ["daily", "user_engagement"]
```

---

## Advanced Features

### Snapshots (SCD Type 2)
```sql
-- snapshots/users_snapshot.sql
{% snapshot users_snapshot %}
    {{
        config(
            target_schema='snapshots',
            unique_key='id',
            strategy='timestamp',
            updated_at='updated_at',
        )
    }}
    SELECT * FROM {{ source('raw_data', 'users') }}
{% endsnapshot %}
```

### Seeds
```bash
# Load CSV data
dbt seed

# Load specific seed
dbt seed --select my_seed_file
```

### Hooks
```yaml
# dbt_project.yml
models:
  my_project:
    pre-hook: "{{ create_audit_table() }}"
    post-hook: "{{ grant_select('role_analyst') }}"
```

### Variables
```yaml
# dbt_project.yml
vars:
  start_date: '2023-01-01'
  exclude_test_users: true
```

```sql
-- In model
WHERE created_at >= '{{ var("start_date") }}'
{% if var("exclude_test_users") %}
  AND email NOT LIKE '%test%'
{% endif %}
```

---

## Best Practices

### 1. Project Organization
```
models/
├── staging/           # 1:1 with source tables
│   ├── base/         # Light transformation
│   └── schema.yml    
├── intermediate/      # Purpose-built transformations  
│   └── schema.yml
└── marts/            # Business-ready models
    ├── core/         # Key business entities
    ├── finance/      # Department-specific
    └── marketing/
```

### 2. Naming Conventions
```sql
-- Staging models
stg_[source]__[table]     -- stg_salesforce__contacts

-- Intermediate models  
int_[entity]_[verb]       -- int_users_aggregated

-- Mart models
[entity]_[grain]          -- users_daily, orders_summary
```

### 3. Model Configuration Best Practices
```yaml
# dbt_project.yml
models:
  my_project:
    +materialized: view
    staging:
      +materialized: view
    intermediate:
      +materialized: ephemeral  
    marts:
      +materialized: table
```

### 4. Performance Optimization
```sql
-- Use appropriate materializations
{{ config(materialized='incremental') }}

-- Optimize incremental models
{% if is_incremental() %}
  WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
{% endif %}

-- Use appropriate clustering/sorting
{{ config(
    materialized='table',
    sort=['date_column', 'id'],
    dist='user_id'
) }}
```

### 5. Data Quality Standards
```yaml
# Every primary key should be tested
- name: user_id
  tests:
    - unique
    - not_null

# Test business logic
- name: total_amount
  tests:
    - dbt_utils.expression_is_true:
        expression: ">= 0"
```

### 6. Version Control Best Practices
```bash
# .gitignore
target/
dbt_packages/
logs/
profiles.yml    # Contains credentials

# Use environment variables for credentials
export DBT_PASSWORD="your_password"
```

---

## CLI Commands Cheat Sheet

### Basic Commands
```bash
# Run all models
dbt run

# Run specific model and downstream
dbt run --select my_model+

# Run specific model and upstream  
dbt run --select +my_model

# Run models in folder
dbt run --select staging

# Run models with tag
dbt run --select tag:daily

# Full refresh (rebuild incremental)
dbt run --full-refresh

# NEW in v1.10: Sample mode for faster development
dbt run --sample
```

### Testing Commands
```bash
# Run all tests
dbt test

# Test specific model
dbt test --select my_model

# Test source freshness
dbt source freshness

# NEW in v1.10: Enhanced validation
dbt compile --strict
```

### Development Commands
```bash
# Compile without running
dbt compile

# Debug connection
dbt debug

# Parse project
dbt parse

# Generate documentation
dbt docs generate && dbt docs serve
```

### Package Management
```bash
# Install packages
dbt deps

# Clean project
dbt clean

# Show project info
dbt list
```

### Advanced Selection
```bash
# Modified models since last run
dbt run --select state:modified

# Models that failed last run
dbt run --select result:error

# Complex selections
dbt run --select "staging,tag:hourly,+my_model"
```

---

## Jinja & Macros

### Basic Jinja
```sql
-- Variables
{% set my_var = 'value' %}
SELECT '{{ my_var }}' as column_name

-- Loops
{% for item in ['a', 'b', 'c'] %}
  SELECT '{{ item }}' as letter
  {% if not loop.last %} UNION ALL {% endif %}
{% endfor %}

-- Conditionals  
{% if target.name == 'prod' %}
  SELECT * FROM production_table
{% else %}
  SELECT * FROM {{ ref('dev_table') }}
{% endif %}
```

### Custom Macros
```sql
-- macros/get_date_parts.sql
{% macro get_date_parts(date_column) %}
    EXTRACT(year FROM {{ date_column }}) as year,
    EXTRACT(month FROM {{ date_column }}) as month,
    EXTRACT(day FROM {{ date_column }}) as day
{% endmacro %}

-- Usage in model
SELECT 
    id,
    {{ get_date_parts('created_at') }},
    amount
FROM {{ source('sales', 'orders') }}
```

### Useful dbt_utils Macros
```sql
-- Generate surrogate key
{{ dbt_utils.generate_surrogate_key(['col1', 'col2']) }}

-- Get column values as list
{% set statuses = dbt_utils.get_column_values(ref('orders'), 'status') %}

-- Pivot table
{{ dbt_utils.pivot('status', dbt_utils.get_column_values(ref('orders'), 'status')) }}

-- Date spine
{{ dbt_utils.date_spine(
    datepart="day",
    start_date="'2023-01-01'",
    end_date="current_date"
) }}
```

---

## Performance Optimization

### 1. Choose Right Materializations
```sql
-- High-volume, frequently queried: table
{{ config(materialized='table') }}

-- Light transformations, infrequently queried: view  
{{ config(materialized='view') }}

-- Large datasets with regular updates: incremental
{{ config(materialized='incremental') }}

-- Building blocks for other models: ephemeral
{{ config(materialized='ephemeral') }}
```

### 2. Optimize Incremental Models
```sql
{{ config(
    materialized='incremental',
    unique_key='id',
    incremental_strategy='merge',  -- or 'append', 'delete+insert'
    on_schema_change='sync_all_columns'
) }}

SELECT * FROM {{ source('events', 'raw_events') }}
{% if is_incremental() %}
  WHERE event_time > (SELECT MAX(event_time) FROM {{ this }})
{% endif %}
```

### 3. Use dbt-utils for Performance
```sql
-- Efficient deduplication
{{ dbt_utils.deduplicate(
    relation=source('raw', 'events'),
    partition_by='user_id',
    order_by='created_at desc'
) }}
```

### 4. Warehouse-Specific Optimizations
```sql
-- Snowflake
{{ config(
    cluster_by=['date', 'user_id'],
    automatic_clustering=true
) }}

-- BigQuery
{{ config(
    partition_by={'field': 'date', 'data_type': 'date'},
    cluster_by=['user_id', 'status']
) }}

-- Redshift  
{{ config(
    sort=['date', 'user_id'],
    dist='user_id'
) }}
```

---

## Troubleshooting

### Common Issues & Solutions

#### 1. Model Not Found
```
Model 'my_model' not found
```
**Solution**: Check model name, ensure it's in the models directory, run `dbt parse`

#### 2. Circular Dependencies  
```
Circular dependency detected
```
**Solution**: Review model dependencies with `dbt list --resource-type model --output json`

#### 3. Compilation Errors
```
Compilation Error: Undefined variable
```  
**Solution**: Check variable names, ensure proper Jinja syntax

#### 4. Schema Changes
```
Schema mismatch in incremental model
```
**Solution**: Use `--full-refresh` or set `on_schema_change='sync_all_columns'`

#### 5. Memory Issues
```
Out of memory error
```
**Solution**: Reduce model complexity, use incremental materialization, optimize SQL

### Debug Commands
```bash
# Check connection and setup
dbt debug

# Compile without running to check syntax
dbt compile --select my_model

# Show compiled SQL
dbt show --select my_model

# Run with verbose logging
dbt run --select my_model --log-level debug
```

### Log Analysis
```bash
# Check recent logs
tail -f logs/dbt.log

# Search for errors
grep -i error logs/dbt.log
```

---

## dbt Core 1.10 New Features (2025)

### Sample Mode for Development
```bash
# Run with sample data for faster iteration
dbt run --sample

# Configure sampling in model
{{ config(
    materialized='table',
    sample_size=1000
) }}
```

### Enhanced Validation
```bash
# Stricter validation during compilation
dbt compile --strict

# Validate project configuration
dbt parse --validate
```

### Micro-batch Incremental Strategy
```sql
{{ config(
    materialized='incremental',
    incremental_strategy='microbatch',
    event_time='created_at',
    batch_size='hour'
) }}
```

### Cost Management Features
```yaml
# Monitor query costs
models:
  my_project:
    +query_cost_threshold: 100
    +warn_on_large_queries: true
```

---

## Quick Reference Cards

### Model Materialization Decision Tree
```
Data Volume: High → Table or Incremental  
Data Volume: Low → View or Ephemeral

Update Frequency: High → Incremental
Update Frequency: Low → Table

Query Frequency: High → Table  
Query Frequency: Low → View

Complexity: High → Table
Complexity: Low → View
```

### Test Types Quick Guide
```yaml
# Data integrity tests
- unique              # No duplicates
- not_null           # No missing values  
- accepted_values    # Values in allowed list
- relationships      # Foreign key constraints

# Data quality tests (dbt_utils)
- expression_is_true      # Custom conditions
- not_null_proportion     # Acceptable null rate
- unique_combination      # Composite uniqueness
- at_least_one           # Minimum row count
```

### Performance Checklist
- [ ] Use appropriate materialization
- [ ] Implement incremental loading for large tables  
- [ ] Add clustering/partitioning for warehouse
- [ ] Use ephemeral for intermediate transformations
- [ ] Optimize join conditions and filters
- [ ] Monitor query performance and costs
- [ ] Use sample mode during development
- [ ] Test with realistic data volumes

---

**Happy dbt Development! 🚀**

*For the latest updates and documentation, visit [docs.getdbt.com](https://docs.getdbt.com)*
