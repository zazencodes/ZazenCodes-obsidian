---
date: 2024-01-09
tags:
    - video
hubs:
    - "[[data-engineering]]"
    - "[[data-lake]]"
urls:
    - https://www.youtube.com/playlist?list=PL-gIUf9e9CCtGr_zYdWieJhiqBG_5qSPa
---

# Apache Iceberg 101 Course

## Learn the Problem & Solution Behind Iceberg's Origin Story
- Tables are composed of files, the "old way" of doing this in a data lake is Hive format
    - Pros: highly compatible, more efficient than full scans, file format agnostic, full partition rewrites
    - Cons: small updates are inefficient, no safe updates to multiple partitions, multiple jobs not safe, directory listings take a long time, users need to know physical layout of table (to filter properly, avoid full table scans)
- **Iceberg**
    - Has list of files in metadata (an iceberg table == a canonical list of files)
    - Is a table format specification
    - Is a set of APIs and libraries for interacting with that specification
    - **Benefits**: efficiently make smaller updates (at the file-level), snapshot isolation for transactions, faster planning and execution, abstracts physical storage (e.g. hidden partitioning), evolve schemas and partitions over time

## Data Lakehouse & Iceberg Explained
- **Status quo**
    - Data source: fetch data
    - Data lake: put all data here (storage is cheap)
    - Data warehouse: put subset of data here (expensive)
        - Problems: cost, vendor lock-in, tool lock-out, data drift
- **Data lakehouse alternative**
    - Data source: fetch data
    - Data lake: put all data here
    - Apache Iceberg: the data lake IS the data warehouse
        - **Benefits**: inexpensive, same performance, tool availability
- **Components** of a data lakehouse
    - Storage layer (e.g. s3), file format (e.g. parquet), table format (e.g. Iceberg), catalog (e.g. Hive), engines (reading and writing to Iceberg at scale, connects to reporting tools, e.g. dremio, spark)

## Architecture Overview
    - **metadata files** define the table (i.e. what manifest lists to look at)
    - **manifest lists** define a snapshot of the table (i.e. what manifests to look at)
    - **manifest** is a group of files (includes metadata like partition info, file info like format, record count, null counts, etc..)

## Step-by-Step Guide to Apache Iceberg Transactions
- Create table
![[Pasted image 20240110125041.png]]
- Insert data
![[Pasted image 20240110125052.png]]
- Update data
![[Pasted image 20240110125157.png]]
- Select
![[Pasted image 20240110125259.png]]
- Reading from a snapshot
![[Pasted image 20240110125358.png]]

## Learn How to Use Catalogs for Data Management
- Tracks what the **latest metadata** is
- Options
    - Nessie: git like functionality, cloud managed service (Arctic)
    - Hive: good option if already have Hive metastore running
    - AWS Glue: good option for interop with AWS

## Understanding Copy-on-write and Merge-on-read
- **Copy-on-write**: creates new file - impose cost on update
![[Pasted image 20240110145904.png]]
- **Merge-on-read**: marks which rows change - impose cost on read
![[Pasted image 20240110150027.png]]
- Default behavior is copy-on-write
- When to use?
    - Copy-on-write
        - Daily batch jobs, write speed not priority
    - Merge-on-read
        - Streaming and high-frequency batch jobs where write speed is important


## Table Tuning for Apache Iceberg - Table Properties Explained
- Set maximum metadata retention
![[Pasted image 20240110150625.png]]
- Specify columns to track metrics for (reduce footprint of large tables)
![[Pasted image 20240110150716.png]]

## Time Series Analysis for Time Travel
```sql
select t at <timestamp>;
select t as snapshot '<snapshot-id>';
```

## How to Maintain Iceberg Tables
- Expire snapshots
```sql
CALL prod.system.expire_snapshots(
    table => 'db.sample',
    older_than => now(),
    retain_last => 10
)

CALL hive_prod.system.expire_snapshots(
    'db.sample',
    TIMESTAMP '2023-06-30 00:00:00.000',
    100
)
```
- Compaction (re-writing data files and manifests, avoid too many small files)
```sql
CALL catalog_name.system.rewrite_data_files(
    table => 'db.sample',
    strategy => 'sort',
    sort_order => 'id DESC NULLS LAST,name ASC NULLS FIRST'
)

CALL catalog_name.system.rewrite_manifests('db.sample')
```
- Delete orphan files
```sql
CALL catalog_name.sustem.remove_orphan_files(table => 'db.sample')
```

## Hard Deletions & GDPR Compliance
- Easy for copy-on-write (just delete old files)
- Encryption-based deletion (drop encryption key so data is impossible to read)


