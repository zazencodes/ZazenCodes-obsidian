---
date: 2021-10-13
tags:
    - doc
hubs:
    - "[[data-engineering]]"
    - "[[bigquery]]"
urls:
    - https://cloud.google.com/architecture/bigquery-data-warehouse
---

# BigQuery for Data Warehouse Practitioners

- Service model
    - Functions as Data warehouse (home for all analytical data), Data mart (organized into datasets), and Data lake (pooling data from all sources, federated queries in BQ allow for this)
    - Datasets can be regional, e.g. London or multi-regional e.g. US
    - CPU & RAM are provisioned at query runtime using slots
- Managing workflows
    - [Service quotas](https://cloud.google.com/bigquery/quotas) allow for consistent performance
        - There are many quotas for different operations - e.g. 100 concurrent interactive queries
        - Batch queries can be used instead of the default interactive, which do not count towards concurrent rate limit quotas but only run as resource are available - usually within a few minutes
- Monitoring data
    - Clustering is recommended over partitioning when
        - strict cost guarantees are needed
        - segment size is less than 1gb after partitioning
        - partitions are frequently modified
    - Materialized views periodically cache results of a query, they are always consistent with the base table
    - Max 1,500 load jobs per day, so either a) use batch inserts or b) use streaming inserts. Do not load individual records or micro-batches
    - Load method using gs - Cloud Functions watch for Cloud Storage event (file upload) into bucket, triggers pipeline with ETL job (transform then load into production table) or ELT job (load into staging table then transform)
        - Best pipeline method is Dataflow which runs Beam jobs and has BQ connector. Alternate is Dataproc which runs Spark
    - Bigquery data transfer service can pull data from Google ads and other google data sources, but also redshift, s3, etc..
    - Table modifications are ACID compliant
    - Handling Slowly Changing dimensions (SCD) with bigquery
        - Bulk update dimension table with merge and drop old data

                merge dataset.dim_table as main using
                dataset.temp_table as temp
                on main.match_col = temp.match_col
                when matched then
                    update set main.update_col = matched.update_col
                when not matched then
                    insert values(temp.col1, temp.col2...)

        - Use start_date and end_date columns to track dimensions
            - Use view to pick rows out with end_date = null to use for analytics, merging, etc..

                    create view current_dims as
                    select col1, col2...
                    from dataset.dims where end_Date is null;

        - Use an array for SCD column
            - Use view to pick latest dim value from array

                    create view current_dims as
                    select col1, col2 ..., scd_col.ordinal[array_length(scd_col)] as scd_col
                    from dataset.dims


- Querying data
    - Supports standard SQL and "legacy SQL" which is BQs alternate SQL version before they supported standard
        - Standard SQL extends regular SQL to allow e.g. UNNEST for arrays
    - Allows UDFs written in SQL or JavaScript
    - Can schedule queries to run on time or event trigger
    - Optimization (cost, computation)
        - Only reference cols relevant to query, since bq will scan full columns of all included
        - Avoid javascript UDFs
        - Use approx_count_distinct and other appoximate functions when possible
        - Have fact (large) table on left for joins and join in descending order of size

- Costs
    - Tables not changed for 90 days are categorized as long-term storage and the cost reduces by 50% to $0.01 / gb / month
        - This is not changed unless a DML statement is done. Reads are OK
    - Batch loads are not charged, but streaming inserts are
    - Reservation slots can be purchased with flat-rate pricing and assigned to projects


