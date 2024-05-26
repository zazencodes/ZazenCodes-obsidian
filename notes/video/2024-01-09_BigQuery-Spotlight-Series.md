---
date: 2024-01-09
tags:
    - video
hubs:
    - "[[database-concepts]]"
    - "[[bigquery]]"
urls:
    - https://www.youtube.com/playlist?list=PLIivdWyY5sqLAbIdmcMwsxWg-w8Px34MS
---

# BigQuery Spotlight Series

## What are user-defined functions?
- UDFs can be JS or SQL
e.g.
```sql
create temp function cleanse_string (text STRING)
  returns STRING
  as (regexp_replace(lower(trim(text)), '[^a-zA-Z0-9]+', ''));

select `project-id`.udf_library.cleanse_string(string_field) from <table>;
```
- Can be temporary or persistent

## How does BigQuery story data?
- Partition by low cardinality field
- Cluster by high cardinality field
- Data is sorted by cluster, so that you don't have to scan a whole partition

## How does query processing work in BigQuery
- SQL is executed by passing data between distributed storage (colossus) and compute cluster using a distributed memory shuffle to store intermediary results
- Execution details
    - Wait phase - waiting for slots to become available or for previous stage to start writing results
    - Read - reading data from distributed storage (colossus) or shuffle
    - Compute - evaluating SQL
    - Write - shuffle output or final result is written
- How to use this?
- Focus on stages dominating resource
    - e.g. join stage using more input than output
- Data skew
    - one worker handling more data than others
- Filtering early can help
- High CPU
    - Approximate functions can help

## Strategies for optimizing your BigQuery queries
- Only use necessary columns - no `select *`!
    - consider `select * except(col1, ccol2, ...)`
- Use clusters for columns where sorting would be beneficial (commonly applied filters)
- Group by
    - Groupby after joining, unless the group by results in a large reduction of data size to join
    - Group by as late and seldom as possible
    - Nest repeated data and use array functions instead of group by, e.g. `ARRAY_LENGTH`
- Joins
    - Large joins are done by hashing left and right tables and sending matching keys to the same workers
    - Small joins are done using broadcasts - the smaller table is sent to all workers to join on large table
    - Place the large table first in the join query
    - Clustering on common join keys is helpful
- Where clause
    - The order of where clauses matters - include the filters that eliminate the most data first (presumably, also the filters that are the least computationally intense)

## Understanding BigQuery data governance
- Steps for data governance
    - Understand the data
    - Categorize the data
        - add labels, e.g. "has_ppi": true
        - attach tags to columns (schematized tagging in data catalog)
    - Transform sensitive data, e.g. de-identify PII
    - Access policies
    - Ongoing monitoring
- DLP
    - scan data in (bigquery, cloud storage, etc..) for automated governance tagging
- Onboarding pipeline example for handling sensitive data
    - Stage in BQ
    - Scan with DLP
    - Store results to DLP data catalog tags
    - De-identify sensitive comments (if DLP reveals sensitive data)
    - Save in BQ
- Policy example
    - ecomm-team@mycompany.com
    - Grant access as needed to projects, datasets, or authorized views
    - Data catalog can be used to control access at column level (without needing a view)
    - Row level access can be controlled as well (without needing a view), e.g. `create row access policy activewear_team on <table> grant to ("<email>") filter using (category="Active")`
- Common monitoring patterns: check duplicate data, missing data, etc..

