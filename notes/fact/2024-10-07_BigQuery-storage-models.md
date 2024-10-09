---
date: 2024-10-07
tags:
    - fact
hubs:
    - "[[bigquery]]"
urls:
    - https://medium.com/cts-technologies/bigquery-storage-billing-models-c2fc48aa8d13
---

# BigQuery storage models

- **Logical storage** is the default billing model
- **Physical storage** is a newer billing model

- Key differences:
    - Billing
        - Logical storage bills based on uncompressed size, while physical storage bills based on compressed size.  
        - Physical storage can be [significantly cheaper](https://medium.com/cts-technologies/bigquery-storage-billing-models-c2fc48aa8d13) than the logical size, especially for data that compresses well.
    - Time travel:
        - Logical storage includes time travel storage at no additional cost.
        - Physical storage charges for time travel separately.

- Query to determine storage savings (requires `roles/bigquery.admin` role)

```sql
DECLARE active_logical_gib_price FLOAT64 DEFAULT 0.02;
DECLARE long_term_logical_gib_price FLOAT64 DEFAULT 0.01;
DECLARE active_physical_gib_price FLOAT64 DEFAULT 0.04;
DECLARE long_term_physical_gib_price FLOAT64 DEFAULT 0.02;

WITH
 storage_sizes AS (
   SELECT
     table_schema AS dataset_name,
     -- Logical
     SUM(active_logical_bytes) / power(1024, 3) AS active_logical_gib,
     SUM(long_term_logical_bytes) / power(1024, 3) AS long_term_logical_gib,
     -- Physical
     SUM(active_physical_bytes) / power(1024, 3) AS active_physical_gib,
     SUM(active_physical_bytes - time_travel_physical_bytes - fail_safe_physical_bytes) / power(1024, 3) AS active_no_tt_no_fs_physical_gib,
     SUM(long_term_physical_bytes) / power(1024, 3) AS long_term_physical_gib,
     -- Restorable previously deleted physical
     SUM(time_travel_physical_bytes) / power(1024, 3) AS time_travel_physical_gib,
     SUM(fail_safe_physical_bytes) / power(1024, 3) AS fail_safe_physical_gib,
   FROM
     `region-europe-west2`.INFORMATION_SCHEMA.TABLE_STORAGE_BY_PROJECT
   WHERE total_logical_bytes > 0
     AND total_physical_bytes > 0
     -- Base the forecast on base tables only for highest precision results
     AND table_type  = 'BASE TABLE'
     GROUP BY 1
 )
SELECT
  dataset_name,
  -- Logical
  ROUND(
    ROUND(active_logical_gib * active_logical_gib_price, 2) +
    ROUND(long_term_logical_gib * long_term_logical_gib_price, 2)
  , 2) as total_logical_cost,
  -- Physical
  ROUND(
    ROUND(active_physical_gib * active_physical_gib_price, 2) +
    ROUND(long_term_physical_gib * long_term_physical_gib_price, 2)
  , 2) as total_physical_cost
FROM
  storage_sizes
```




