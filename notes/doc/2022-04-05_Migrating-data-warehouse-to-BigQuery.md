---
date: 2022-04-05
tags:
    - doc
hubs:
    - "[[database-concepts]]"
    - "[[bigquery]]"
urls:
    - https://cloud.google.com/architecture/dw2bq/dw-bq-migration-overview
---

# Migrating data warehouse to BigQuery


## Introduction and Overview

- Migrate in iterations (sprints)
    - Prioritize use cases based on
        - Cost saving opportunities
        - Fix limitations (e.g. scale, compute) of current setup
        - Improved user experience (in final data product)
        - Low risk (less critical)

- Steps
    - Configure GCP and data governance
    - Migrate schema & data
    - Rewrite/update (SQL) queries
    - Migrate business applications
    - Migrate data pipelines
    - Optimize performance
    - Verify / validate data



## Schema and data transfer

- Evolving schema after transfer to BigQuery
    - Setup working use case of existing tables in BigQuery
    - Re-create tables with optimizations: partitioning & clustering and connect with upstream/downstream processes
    - Create "facade views" from tables
    - Transform downstream processes to use facade views instead of existing tables
    - Create evolved tables that use nested/repeated fields (denormalized) and write to these from upstream processes
    - Swap facade views to read from evolved tables, or remove from pipeline if they do not aggregate or filter data

- Data transfer techniques
    - a) Install migration agent GCP service on-premises and connect with data transfer service in GCP
    - b) Move data from on-premises to gs and use GCP services to ingest into BQ
    - Extracting source data
        - Use of AVRO is recommended because it encodes the schema along with the data
    - Ingesting to BigQuery from gs
        - If not modifying the data
            - One-time uploads from UI / bq command line tool / python / ...
        - If modifying the data
            - Cloud Data Fusion graphical tool
            - Dataflow for Apache Beam
            - Dataproc for Hadoop

- Schema updates
    - Common pattern is to partition by date or time ingested and cluster by columns frequently used for filtering
    - Denormalization
        - Using nested and repeated fields reduces number of joins (computation) and aligns schema with BigQuery's internal data representation (improving efficiency)



## Data governance

- Perimeter security assumes threats are only external to it's walls, but this model is flawed
    - Too many attack vectors to gain access to internal networks

- Bigquery data access
    - Typically done at the dataset level, then project and project folder, finally organization
    - Table-level permissions can be setup with IAM
        - Applied to users, groups or service accounts
        - Authorized views can grant access to partial data within a table
    - Virtual Private Cloud (VPC) service provided by google for restricting traffic to API services (e.g. bigquery, gs, etc..) in addition to IAM

    - GCP encrypts all data stored at rest by default
        - Data in transit is only guaranteed to be encrypted outside of GCP physical servers
        - Crypto deletion is when you delete the encryption key for data. The data still exists but is unrecoverable

- Data catalog GCP service
    - Metadata store, batch syncers, search index
    - Can automatically ingest from BigQuery, pub/sub, gs
    - Lineage is path taken by data from origin to destination
        - Can be automated with data catalog but must be implemented manually
        - Cloud Data Fusion product captures this automatically

- Data loss prevention (DLP)
    - Can scan data to identify PII

- Audit logs
    -  BigQuery automatically saves admin and data access logs to Cloud Logging service


## Data pipelines

- Migrating ETL between (after 'T')
    - Existing ETL pipelines can replace sink (step in between transformation and legacy data warehouse) with gs filesystem
    - Once data is in gs, Cloud Storage transfer can be used to schedule recurring loads into BigQuery, or use custom Cloud Functions

- Migrating data pipelines
    - Dataproc for hadoop jobs
        - Set cluster config at the job level, scale clusters as needed, only pay for resources used at runtime
    - Re-host existing pipelines with Kubernetes (GKE) or VMs
    - Dataflow for apache beam

## Reporting and analysis

- Network connectivity
    - BQ can be accessed via REST API or RPC-based BigQuery Storage API
    - Tableau and Google's products like Data Studio and Dataproc use the BQ Rest API
    - ODBC or JDBC drivers may be needed for some tools
    - Uses OAuth 2 access tokens

- Exploratory SQL analysis
    - BigQuery ML can be used to build and execute ML models with SQL
        - Can run models at scale (auto-scaling memory, CPU)
    - Dataflow SQL can be used to
        - Develop and run streaming pipelines from the BigQuery web UI
        - Run queries on pub/sub topics and join with other resources
    - Can deploy Jupyter Notebooks using GCP managed service Notebooks or with Dataproc
    - Can deploy Apache Zeppelin notebooks with Dataproc


## Performance optimization

- General performance considerations
    - Slots
        - A set number of maximum slots are available for actively running queries
        - The default slots are usually more than sufficient for customers
        - Additional slots however can be purchased using on-demand or flat-rate pricing
    - Use query plan and timeline to see stages, compute and timeline
        - Identify if certain stages dominate resources, e.g. a JOIN that uses more output rows than could instead be filtered out
    - Avoid external data sources that are large
        - Bad performance, results not caches
    - Query optimization strategies
    - Partitioning
        - Tables over 10gb should definitely be partitioned
    - Clustering
        - Sorted blocks eliminate scans of unnecessary data
    - Denormalization
        - Use nested (STRUCT type) and repeated (ARRAY type) fields
        - Localizes data to slots allowing execution to be done in parallel
        - Reduces network communication during shuffle phase
            - Shuffle is middle step on map reduce parallelized framework where mapped results are sorted and assigned to workers to perform the reduce step
        - Increases storage requirements (tradeoff)
        - Example given fact table fact_tab and dimension table dim_tab

                # create denomalized version of table
                # note: array_agg(struct(d.col1, d.col2, ...)) is array of dicts (i.e. columns in dim table)
                create table denormed_tab
                partition by date(date_col)
                cluster by cluster_key
                as
                select
                    f.col1, f.col2, ...
                    array_agg(struct(d.col1, d.col2, ...)) as dim_col
                from fact_tab f join dim_tab d
                on f.join_col = d.join_col and ...
                group by f.col1, f.col2, ...

                # selecting no longer requires joins
                # this will return records with arrays
                select col1, col2 ...
                from denormed_tab

                # this will unnest records with arrays into multiple records
                select d.col1, d.col2, ... dim_cols_unnest.*
                from denormed_tab as d
                join unnest(dim_cols) as dim_cols_unnest




