---
date: 2021-10-13
tags:
  - course
hubs:
  - "[[data-engineering]]"
  - "[[gcp]]"
urls:
  - https://www.udemy.com/course/google-cloud-professional-data-engineer-get-certified/
---

# GCP Data Engineer - Get Certified 2021

Instructor: Dan Sullivan


## 01 Introduction
- How to study for exam
    - Read study guide
    - Read GCP docs
    - Practice using GCP
    - Do qualification / practice tests

## 02 Cloud Storage for Data Engineering

- Security
    - Data in gs is automatically encrypted
    - You can manage these keys yourself
    - Activity and data access can be tracked

- Cloud storage
    - Loading data < 1Tb use gsutil or console
    - Loading data over 1Tb use Google Transfer Service
        - Can be used for recurring transfer

- Access controls
    - Recommended to use uniform bucket level access with IAM
        - Can be applied to group of files with given prefix in bucket
    - Fine-grained is legacy system
    - Signed URLs can grant time-limited read/write
        - Can provide access to those outside or IAM
    - Signed policy documents control what can be uploaded by file size, type, etc..

- Lifecycle policy
    - Actions executed when condition applies, e.g. delete an object at date, only keep certain number of versions of object, change storage class (nearline -> coldline -> archive)
    - Conditions include age, created before, is live, matches storage class, number of newer versions


## 03 Relational Databases - Cloud SQL

- Good for structured data
    - e.g. ecom application, insurance claim processing, manufacturing environment

- ACID transactions
    - atomicity - full transaction occurs or it doesn't
    - consistency - database must be consistent before and after transaction
    - isolation - multiple transactions can occur independently without messing either up
    - durability - once transaction is marked success, then data can be trusted even in system failure

- Very strong consistency - two people looking at data are very unlikely to see different results. Not always the case

- SQL and DML (Data Manipulation Language)

- When not to use relational
    - Data warehousing -> bq
    - Unstructured data (text, videos images) -> gs
    - Semi-structured data - document db like filestore
    - High volume, low latency writes -> wide column like bigtable

- Limits
    - Regional data store with multiple zones optional for availability
    - Uses one of Postgres, MySQL, SQL server
    - Up to 30Tb per db

- Console
    - Can monitor memory, storage, CPU, etc.. usage in timeseries chart

## 04 Relational Databases - Cloud Spanner

- Better option than Cloud SQL if need global distribution, highly available
- Horizontally scalable (number of nodes)
    - Up to 2 Tb per node
    - Automatically handles replication

- Best performance when workload is well balanced over nodes
    - Hot spotting is when one node is doing much more work than others
    - e.g. auto-increment value or timestamps; many close in time will be allocated to the same node
    - Better to use hash value of primary key

- Interleaved tables mix data from parent and child tables to improve efficiency
    - e.g. person to address, order to order line info

## 05 NoSQL Databases_ Cloud Firestore

- Manged NoSQL
- Not great for IOT due to bounded ingestion limits
- Datastore mode
    - Backend for server apps
    - Does not require syncing many connections
- Native mode
    - Good for mobile apps
    - Support large number of connections

- Entity is key-value structure of related things
    - Similar to a row in SQL

- Set of similar entities is a "kind"
    - Similar like to a table in SQL

- Namespaces group kinds
    - Similar to schema in SQL

- Index
    - All queries require an index
    - There is no scanning in firestore
        - All queries are answered by looking at values in indexes
    - Single field and composite indexes supported
    - Automatic single-field indexes created on atomic values (int, float, etc..), maps, arrays
    - Composite
        - Not automatic (create manually)
        - Needed when multiple attributes in "where" clause
    - Cannot use range filters on different fields
    - Cannot use !=

- Can use SQL-style query with "GQL"
    - Where clauses work with one value by referencing automatically crated built-in index
    - Multiple value where clause requires composite indexes

- Atomic transactions
    - Either all succeed or all fail

- Serializable isolation
    -  Transactions see snapshot of database
    -  Data read by transaction cannot be modified concurrently
    -  Writes in a transaction cannot be read later on in the transaction


## 06 NoSQL Databases_ Bigtable

- Wide column database
    - Like Hbase or Cassandra
        - Supports Hbase API
    - Sparse multi-dimensional array is base abstraction
        - analogous to table, key-value pairs in other DB models
        - sparse meaning arrays have few values in them


- Use cases
    - Petabyte scale data
    - Low-latency writes
    - Analytics data
    - Read based on keys (e.g. row key, scan set of rows)
    - Time series
        - IoT
        - Finance
        - Monitoring

- Scales linearly
    - Performance increases linearly as nodes are added



- Row keys are like primary keys in relational DBs
    - Bigtable does not support joins
    - Used to determine where data is written
        - Bigtable writes data to multiple nodes (servers)
        - Work is distributed across nodes
        - Multiple tables (string sorted tables - SSTables) exist in each node
        - Data is sharded into blocks and stored on Collosus file system
        - Nodes only store metadata, not actual data!
    - Ideally these evenly distribute reads and writes across nodes
        - Want to avoid hotspotting of metadata written to nodes and sharded data written to Collosus filesytem
            - Avoid linearly incrementing row keys
            - Avoid low cardinality attributes
    - Good design patterns
        - Concatenate multiple attributes
            - Field promotion
                - The higher the cardinality the more you want them near the front of the key
        - Start with high cardinality attributes and then add time
            - e.g. IoT sensor ID + date and hour
        - Low cardinality attributes and then add reverse time
            - e.g. IoT sensor ID + reversed date (minute, hour, day, month, year)

- Design considerations
    - Denormalize data
    - Each table has only one index based on the row key
        - In contrast to e.g. firestore which can have many
    - Columns grouped into families
        - Supports up to 100 column families
        - Group similar attributes in a family
            - e.g.
                - Street address, city, state
                - Product name, price, description
    - Operations atomic at the row level
        - Not true atomicity - some rows may fail if updating many
    - Related entities should be stored in adjacent rows to improve reads
    - Empty columns take up no space! (hence the sparse db model)
    - Small tables are problematic
        - Try to use fewer, wider tables
        - Issues
            - Latency of sending requests to many tables
            - More difficult to load balance many small tables

- Recommended patterns for timeseries data
    - Keep names of columns and column families short
        - These are stored with the data (due to nature of sparse representation)
    - Use tall and narrow tables
        - One row for each measurement
    - Better to add rows rather than update them
        - Only want to update when fixing an error
    - Design row key to support query pattern
        - Need to use multiple tables in order to support multiple query patterns
    - Large rows and/or columns can degrade performance
    - Avoid using transactions when writing data



## 07 Analytical Databases_ BigQuery Data Management

- Overview
    - Serverless
    - Petabyte scale
    - Not relational but supports SQL
    - Tables are stored in columnar format

- Views
    - Typically created for join queries

- Federated data access
    - Federated queries access data in other dbs
        - Cloud storage (gs)
            - Parquet
            - ORC
        - Bigtable, CloudSQL
        - Spreadsheet


- Nested data types
    - Arrays
        - Can contain structs, but not arrays
        - Declare as ARRAY\<T\>, e.g. ARRAY\<int64\>
    - Sructs
        - Can be compared with = or != and \[not\] in logic
        - Declare as STRUCT\<T\> e.g. STRUCT\<a int64, b string\>
    - Unnest example query

    ```sql
    select page.title
    from table, unnest(col.arrayOfStructsField) page
    where page.name in ('page1', 'page2');
    ```

- Access controls
    - Dataset or table level
    - Need bigquery.datasets.update for write and bigquery.datasets.get for read
    - Table access needs policies
        - Can be reader writer or owner
        - bigquery.tables.setIamPolicy
    - Views are always read only

- Partitioned tables
    - Improves performance and lowers cost
    - Ingestion time partitions create `_PARTITIONTIME` which is date-based timestamp
    - Dates/timestamps can also be used to partition data
        - Each partition holds one day of data
        - `__NULL__` partition holds data whose partition column is null
        - `__UNPARTITIONED__` partition holds values in column outside allowed range, which can be specified at the table level
    - Integer partitions can be specified with start, end, step range
    - Require partition filter can be set to force readers to use the partition
        - Set at the table level
        - Queries will require where clause that involves partition col

- Sharding
    - Creating a new table for each day or partition
    - Naming convention = `[TABLE_NAME_PREFIX]_YYYY_MM_DD`
    - Partitioning is preferred over sharding
        - Less overhead / metadata
        - Better performance
    - UNION queries needed to scan multiple tables

- Clustered tables
    - Data sorted based on values in one or more columns
    - Can improve performance of aggregations
    - Can reduce scanning when cluster columns are involved in where clause
    - Can only be used on partitioned tables
    - Automatic reclustering occurs in the background after new data is added


- Loading data into bigquery
    - Tools
        - gs
        - other google services
        - local
        - streaming
        - DML bulk loads
        - BQ IO transform in cloud dataflow

    - Methods
        - BQ GUI
        - BQ API
        - BQ command line
            - CLI operations must be done using the bq command - not the gcloud command

    - Formats
        - Avro binary is preferred
            - Compressed or uncompressed
        - Parquet
            - Good if size of file is a concern since it gets better compression than Avro
        - CSV / JSON
            - Keep these uncompressed to significantly increase ingestion speed

- Schema
    - Can be set in JSON format when loading
    - BQ can auto-detect


- BQ transfer service
    - Moves data into BQ from other SaaS services
    - No servers (managed)
    - Schedule loads
    - Access other data sources through connectors
        - e.g. Adobe analytics, kafka, ...

## 08 Migrating a Data Warehouse

- Likely to find this topic on exam
    - Should read more on data transfer prior to exam
        - https://cloud.google.com/architecture/dw2bq/dw-bq-schema-and-data-transfer-overview

- Data warehousing is designed for analytics
    - In contrast to OLTP dbs
        - e.g. for managing inventory
        - these tend to be *sources* of data for data warehousing
    - Focused on querying and reporting

- How to migrate
    - Plan
        - Prepare and discover workloads and use cases of data to move
        - Prioritize the use cases
            - What data should be moved when?
        - Define a measure of success
            - How much data is moved over? How many reports have been moved?
        - Create a planning document with the above and other details
    - Execute
        - Migrate data, schema and downstream/upstream operations (anything else based on data warehouse)
        - Verify and validate you get the same reports / results after migration

- Schema and data transfer
    - Transfer tables
    - Downstream processes that consume data from the warehouse
    - Focus on groups of tables and downstream processes that are related to similar business area / report
    - Do not update schema while transferring!
    - Transfer process at a high level
        - Start by setting up ETL job or using BigQuery Transfer Service to batch transfer the legacy data warehouse
        - Duplicate downstream processes from BigQuery
        - Duplicate upstream processes into BigQuery
        - Legacy upstream and downstream remains intact
            - Should see same results in reports from each data source
        - Turn off legacy system

- Managing a Data warehouses
    - Data pipelines
        - Applications that process data through a series of steps
        - Used for ETL, data enrichment or real-time analysis
        - In a data warehouse the pipelines read, apply translations and write them
        - Common variation is ELT (extract load transform)
            - Take source data, write to sink (staging area in data warehouse), then apply translations and push to production area of data warehouse!

    - Data analytics
        - Descriptive
            - Analyze real-time or historical data
        - Predictive
            - Estimate likelihood of future events
        - Prescriptive
            - Add recommendations on top of predictive analytics

    - Data governance
        - Process of managing data through full lifecycle
            - From time you get it to time you dispose of it
        - 3 components - People, processes and tools
        - Data Access
            - Authorization via roles / identity
            - Can have short-lived or long-lived credentials
        - Encryption
            - All data is encrypted at rest in GCP
            - Google can manage keys or you can
            - Encryption in transit should be considered
            - Crypto-deletion or crypto-shredding
                - Concept of deleting key to data to make encrypted data useless
        - Data discovery
            - Metadata describes data
                - Origin, where it's been (providence), owner, quality
            - Data Catalog is GCP service that manages this metadata
                - Real-time and batch syncs
                - Google search index


## 9 Caching Data and Cloud Memorystore

- Cloud services for Redis and Memcache
    - DBs used for caching
        - low-latency, in memory data
        - Fontend for database; cache results of common queries
        - Online games store player information
        - Stream processing buffer (temporary store incoming)

- Redis
    - Connect from app engine, cloud functions, compute VMs, kubernetes clusters
    - Scale as needed
        - Up to 30gb storage
    - Basic tier has no replication (e.g. cross zone) - advanced has this feature
    - Persisting to disk available in OS version (not Cloud Memorystore)
        - Point in time snapshots
        - Logs of write operations
    - Rich datatype options
        - Strings, lists, sets, sorted sets, hashes, bitmaps

- Memcached
    - Distributed in-memory key-value store
    - Use for reference data, db query caching, session caching
    - Instance is one cluster
        - max 20 nodes
        - 1-32 vCPUs per node
        - 1gb-256gb of memory per node, max 5TB in instance
    - Useful for large caches that need to be highly available (fast) but limited data types
        - A good basic key-value string -> string map


## 10 Workflows and ETL_ELT

- Cloud composer
    - DAGs and dagrun logs stored in cloud storage (gs)
    - Streaming logs available in Log Viewer
        - scheduler, web server, workers
    - Deployed in environments
        - Collections of GCP service components based on Kubernetes engine
        - Tenant project resources
            - Fully Google managed
                - CloudSQL for airflow metadata
                - App Engine Flexible for web server
        - Customer project resources
            - Cloud Storage for DAGs, logs, plugins, data dependencies, ...
            - Kubernetes engine for deploying scheduler, worker nodes and CeleryExecutor
            - Redis for message broker on CeleryExecutor

- Cloud Data Fusion
    - Managed service based on CDAP open source platform
    - Code-free "drag and drop" ETL/ELT, over 150 data source connectors (internal within GCP and external)
    - Basic / enterprise (with additional features / support)


## 11 Cloud Pub_Sub and Data Pipelines
- Cloud Pub/Sub is a managed messaging service
    - One app writes message, another reads it
    - Done async
        - Very useful for decoupling services
- Topic is structure for storing messages
- Subscription allows application to read messages in queue
    - Configuration options
        - How long before subscription expires if no activity in queue
        - Max time after message ingestion to wait for acknowledgment before sending failure notification to queue
            - Failure results in message being re-added to the queue
        - Message retention duration



## 12 Cloud Dataflow and Data Pipelines
- Apache Beam runner managed service
    - Horizontally scalable
    - Often used for ETL/ELT
    - Stream and batch processing
    - Supports windowing operations for stream processing
        - e.g. moving averages of sensor data
- Jobs can run continuously (stream processing) or on schedule
- Pre-defined job templates in GCP Data Flow platform
    - e.g. pubsub to BQ, word count from set of text files, etc..
- Monitoring jobs
    - From console
        - GUI graph, list of jobs running, job status, error logs, etc...
        - Job states
            - Running / not started / queued / cancalling / draining (in process of closing) / updating / succeeded / failed
        - Job details
            - Graph, metrics, info, logs, worker logs (compute nodes), job error reporting

- Troubleshooting
    - Check error logs
    - Look for pipeline construction errors
    - Look for errors in job validation
    - Slow running pipelines or no output

## 13 Cloud Dataproc and Data Pipelines



## 14 Special Considerations in Distributed Systems
## 15 Monitoring and Logging
## 16 Security and Compliance
## 17 Introduction to Machine Learning
## 18 Building ML Models
## 19 Deploying and Monitoring ML Models
## 20 Analytical Databases_ BigQuery ML
## 21 Conclusion
## 22 Practice Exam

