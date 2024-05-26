---
date: 2021-10-13
tags:
  - course
hubs:
  - "[[data-engineering]]"
  - "[[bigquery]]"
urls:
  - https://www.udemy.com/course/bigquery/
---

# BigQuery for Big Data Engineers   Master BigQuery Internals

Instructor: J Garg

## Section 1: Introduction to GCP
- Why GCP?
    - Separation of compute and storage
    - Highly available
    - Ease of scalability

- Services overview
    - From most to least customizable
        - Compute Engine is barebones VM / server
        - App Engine is abstraction of Compute Engine
        - Cloud Functions are serverless

- Storage
    - Cloud Storage
        - Filesystem type storage
    - Cloud SQL
        - Good for small datasets
        - Managed DB for MySQL, Postgres, SQL server
        - Builtin backups, per-second billing
    - Cloud BigTable
        - High throughput, append only NoSQL
        - High performance and high volume needs (pentabyte-scale)
        - Hadoop integration
    - Cloud Datastore
        - NoSQL structured data that is accessed through key-value pairs
        - Like Bigtable but with ACID transactions, SQL-like queries
    - Cloud Spanner
        - Good for huge structured transactional data
        - Combines benefits of relational DB with horizontal scalability of NoSQL

- Big Data Services
    - BigQuery
        - Data warehouse service
    - Cloud Dataflow
        - Batch and stream processing
        - Based on apache beam
    - Cloud Dataproc
        - Cluster of VM machines with hadoop and spark installed
        - Easy integration with other GCP services like Big table, Cloud storage, etc...

- Big data end-to-end cloud system
    -  Input layer
        - First stop on GCP
        - gs / Cloud SQL / BigTable / Cloud Spanner
    -  Ingestion layer
        -  Getting data onto GCP
        -  Cloud Pub/Sub, Compute Data Transfer, Cloud IoT Core, Cloud Filestore
    -  Processing layer
        - Move data from input layer to storage
        - Cloud Dataflow / Dataproc / Dataprep
    - Storage layer
        - Compute storage / BigQuery / BigTable

## Section 2: Introduction to BigQuery

- BigQuery features
    - Auto backups with 7 day rolling window
    - Data encrypted and hence highly secure

- Architecture
    - Uses Google core services
        - Dremel execution engine
        - Colossus (columnar storage) file system
    - Storage and compute are separated, meaning that ingesting data does not affect queries

## Section 4: Using Bigquery Dashboard Operations
- Queries are automatically cached once run
- Can query pub/sub streams from BigQuery console
- Caching features & limits
    - Only exact replica queries are cached
    - Cache invalidated if table changes
    - Queries with functions like current_timestamp() are not cached
    - Responses over 10gb are not cached
- Maximum bytes billed can set upper limit on charges
- Wildcard queries
    - e.g. have tables sales_table1, sales_table2, ... then can query with "select \* from \`proj.dataset.sales_table\*\`"
    - This allows data to be split into many datasets and still queries nicely
    - Matches * in table name and makes available in  _TABLE_SUFFIX SQL variable
    - BigQuery automatically infers the schema of ALL tables in wildcard based on most recently created table matching
- Schema auto detection
    - Scans a sample (~100) rows of table
    - Dates/timestamps must be ISO
    - Will detect headers, but they are not required

## Section 5: Efficient Schema Design for BigQuery
- BigQuery performs best on denormalized data
    - NO star schemas
    - Nested columns help with this (use datatype = RECORD in bigquery schema)
- Nested column query example equivalent of "select *"
    ```sql
    select * from `dataset.table`
    cross join unnest(nestedParent) as a
    where a.nestedField = "Match"
    ```

    - Comma in from statement is shorthand for cross join
    - Reduce cost by selecting fewer columns (not *)

## Section 6: Operations on Datasets & Tables
- Data transfer service
    -  Run schedule jobs to import data daily, weekly, etc.. Importing data is free
    - 1000/day in-region per table, 100k/day per project
- Can select all columns "EXCEPT" in SQL select!
    ```sql
    select * EXCEPT(col) from table;
    ```

- SQL query + overwrite table in order to change column names, column type or drop columns
    - This will scan full table and may be more expensive than re-creating the table
    - e.g. (noting that operations retain column order)

    ```sql
    select * EXCEPT(col), col as new_col from table;
    select *, EXCEPT(col), cast(col as STRING) as col from table;
    select * EXCEPT(col) from table;
    ```

- Restore table using SQL query (set overwrite table in BigQuery console query settings)

    ```sql
    select * from table for SYSTEM_TIME as of TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR)
    ```

## Section 7: Execution Plan of BigQuery
- Query plan
    - Queries are run in memory
    - The plan is made by identifying stages of query and running them in parallel when possible
    - Query plan may be adjusted while query is running

## Section 8: Partitioned Tables in BigQuery
- Partition based on what columns will be used most often in where clause
- Warehousing example
    - Partition by month and only perform daily updates on most recent partition
- Can set option that forces queries to use date filters
- Pull metadata for partitions with legacy SQL:

    ```sql
    select * from [dataset.table$__PARTITIONS_SUMMARY__]
    ```

- Find invalid partition fields:

    ```sql
    select * from [dataset.table$__NULL__]
    select * from [dataset.table$__UNPARTITIONED__]
    ```
- Max number of partitions is only 4,000!

- Partition expiry
        Partitions can be set to expire like this

    ```sql
    alter table table_name set options (partition_expiration_days=10)
    ```

- Data is removed from table after partition expires

- Avoid doing calculations on partition column in where clause
    - Bad form
        ```sql
        select * from table where TIMESTAMP_ADD(date, INTERVAL 1 DAY) > '2021-01-01'
        ```

    - Good form (note how partition column "date" is alone on left hand side)

        ```sql
        select * from table where date > TIMESTAMP_SUB(TIMESTAMP('2021-01-01'), INTERVAL 1 DAY)
        ```

- Creating too many partitions will slow down performance and table will perform like non-partitioned table. In these cases look to create clustered table instead


## Section 9: Clustered Tables in BigQuery

- Work well with partitioned tables
    - Partitions are like folders in the filesystem
    - Clusters are like files within those folders

- Cost considerations
    - Partitioning more useful when considering cost guarantees or setting partition expiry
    - Clusters will have multiple values so querying is not as efficient as partitioning

- When to cluster?
    - Unique data or many distinct values

- Best practices
    - Query table in same order as cluster columns defined (in where clause)
    - Like partitioned tables, avoid calculations in where clause

## Section 10: Loading & Querying External Data Sources
- Loading data with gs
    - Most common approach to loading data is cloud storage, i.e. `gs`
    - Avro is best data source for BigQuery
    - Compressed files load slower but have obvious benefits (storage / bandwidth)

- External tables
    - Can load as Table type = "External table" to be able to query the data directly from gs
    - "Link the table metadata with the cloud storage file"
    - Cannot give estimated byte scan
    - Find gs location by selecting `_FILE_NAME` as column

## Section 11: Views in Bigquery

- Basic properties
    - No physical data - only result of query on table
    - View schema is frozen once created
    - Read only
    - Deleting base table breaks view

- Benefits
    - Prevent direct access to underlying table
    - Can be used to hide columns (or rows) from base table
    - Can be used as abstraction on complicated query
    - Free of cost in terms of storage

- Tactics
    - Create view in different dataset (since bq access is grated at the dataset level)


## Section 12: Materialized Views in BigQuery

- Basic properties
    - Stored physically on disk
    - Does not refer to base table when queried
    - Auto-refresh option to periodically re-compute view
    - If a query can use a materialized view, bigquery will automatically do this for you by re-routing the query through the view

- Use case
    - ETL / BI pipelines
    - Aggregated queries

- Syntax
    - Basic

    ```sql
    CREATE MATERIALIZED VIEW bigquery-demo-285417.dataset1.names_demo_mv1
    OPTIONS (enable_refresh = true)
    AS SELECT
    ...
    ```

    - Does not support HAVING on aggregate queries, or any kind of filtration or re-aggregation (ie. with inner select)

- Patterns
    - Ad-hoc query with subset of aggregation

    ```sql
    create materialized view ___ as
    select
        thing, ...
        sum(thing2),
        avg(thing3),
        count(*)
    from table
    group by thing;

    -- Ad-hoc subset query will
    -- call materialized view
    select
        thing, count(*)
    from table
    group by thing;
    ```

    - Ad-hoc query with computation on grouping keys

    ```sql
    -- Ad-hoc subset query will
    -- call materialized view
    select regexp_contains(thing, "stuff")
    from table
    group by thing;
    ```

    - Ad-hoc query filters on grouping column

    ```sql
    -- Ad-hoc subset query will
    -- call materialized view
    select thing, count(*)
    from table
    where thing = "stuff";
    ```

- Refreshing
    - Manually

    ```sql
    CALL BQ.REFRESH_MATERIALIZED_VIEW('bigquery-demo-285417.dataset1.names_mv')
    ```

    - Automatically refreshed within 5 minutes of change to base table
        - This can be set with refresh_interval_minutes option (min = 1 minute, max = 7 days)

    ```sql
    set options (enable_refresh = true, refresh_interval_minutes = 60)
    ```

- Basic tactics
    - Should be in the same dataset as the base table
    - Supports limited set of aggregations in view definition
    - Does not support join or unest
    - Each table can have 20 materialized views

- Best practices
    - Use filter field in groupby
    - Use date range filter to exclude date not usually needed
    - Partition materialized view if it's big
    - Consider refreshing manually when base table updates are done on a schedule (e.g. ETL)
    - If needing to join and aggregate at the same time, it's better to create a materialized view for the aggregate and then join on that. Join queries will be faster this way since the agg is pre computed


## Section 13: BQ Command Line

- Basic commands
    - `bq show`
        - List available projects
    - `bq ls`
        - List datasets
    - `bq show dataset`
        - List dataset info
    - `bq show --schema --format prettyjson dataset.table`
        - Show schema for table
    - `bq shell`
        - Start bigquery shell, can run above commands without "bq"

- Query commands
    - `bq query --use_legacy_sql=false "select * from dataset.table"`
        - Useful options
            - `--dry_run` : flag to validate query (see bytes billed, if query will fail) but do not run
            - `--require_cache` : only run if cached result available
            - `--append_table` : append query results to table (true/false)
            - `--destination_table` : table to write
            - `--replace` : overwrite destination (true/false)
    - `bq load dataset.table /Users/alex/Downloads/file.csv name:string,gender:string,count:integer`
    - `cp` and partitions
        - Copy partitions into destination table (append mode)
            - `bq cp -a dataset1.table$partition dataset2.dest_table`
        - Select partitions available in table
            - `select * from [dataset1.table$__PARTITIONS_SUMMARY__]*`


## Section 14: Python Client Library of BigQuery

## Section 15: Build end-to-end Data Pipelines

- Hook up CSV file to dashboard
- Need to clean data after initial ingestion
- Using BQ for DB
- Apache Beam
    - About project
        - Uniform API
        - Pipelines can be created with many languages
    - Load and transform data Example

    ```python
    import apache_beam as beam
    from apache_beam.options.pipeline_options import PipelineOptions, StandardOptions

    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument('--input', dest='input', required=True, help='...')
    args = parser.parse_args()

    path_args, pipeline_args = parser.parse_known_args()

    input_pattern = path_args.input

    options = PipelineOptions(pipeline_args)
    p = beam.Pipeline(options=options)
    cleaned_data = (
        p
        | beam.io.ReadFromText(input_pattern, skip_header_lines=1)
        | beam.Map(remove_last_column)
        | beam.Map(lambda x: x.lower())
        | beam.Map(remove_special_characters)
        | beam.Map(lambda x: f"{x},1")
    )

    delivered_orders = (
        cleaned_data
        | 'delivered filter' >> beam.Filter(lambda row: row.split(,)[8].lower() == "delivered")
    )

    other_orders = (
        cleaned_data
        | 'Underlivered Filter' >> beam.Filter(lambda x: x.split(',')[8].lower() != 'delivered')
    )

    (
        cleaned_data
        | 'count total' >> beam.combiners.Count.Globally()
        | 'total map' >> beam.Map(lambda x: f'Total Count: {x}')
        | 'print total' >> beam.Map(print(row))
    )

    def remove_last_column(row):
        cols = row.split(",")
        item = str(cols[4])
        if item.endswith(":"):
            cols[4] = item[:-1]
        return ",".join(cols)

    def remove_special_characters(row):
        ...transform logic
        return result
    ```


    - parse_known_args
        - This parses anything not explicitly specified and stores as 2nd item in return function

- Continuing example above
    - Load to GCP using beam

    ```python
    client = bigquery.Client()
    dataset_id = f"{client.project}.name"
    try:
        client = get_dataset(dataset_id)
    except:
        ... code to create dataset

    def to_json(string):
        fields = string.split(',')
        return {
            'col1': fields[0],
            ...
        }

    DLIVERED_TABLE_SPEC = f"project_name:{dataset_id}"
    (
        delivered_orders
        | 'delivered to json' >> beam.Map(to_json)
        | 'write delivered' >> beam.io.WriteToBigQuery(
            DLIVERED_TABLE_SPEC,
            schema='customer_if:STRING,date:STRING,...'
            create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED,
            write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND,
            additional_bq_poarameters={'timePartitioning': {'type': 'DAY'}}
        )
    )

    from apache_beam.runners.runner import PipelineState
    ret = p.run()
    if ret.state == PipelineState.DONE:
        print(":DONE")
    else:
        print("ERROR")
    ```


- Create view in pipeline

    ```python
    view = "daily_orders"
    dataset_ref = client.dataset('dataset_name')
    view-ref = dataset_ref.table(view_name)
    view_to_create = bigquery.Table(view_ref)
    view_to_create.view_query = f'select * from {DLIVERED_TABLE_SPEC} where __PARTITIONDATE = DATE(current_date())'
    view_to_create.view_use_legacy_sql = False

    def view_not_exists():
        ...some logic

    if view_not_ exists():
        client.create_table(view_to_create)
    ```

- Install beam on google server and run code

    ```bash
    pip install apache-beam
    python code.py --input gs://daily_food_orders/file.csv --temp_location gs://temp/loc
    # note: temp_location arg gets parsed and send to beam pieline via "PipelineOptions(rgs)"
    ```


- Schedule beam pipeline with airflow

    - Run on Data Flow platform in GCP

    ```python
    from airflow import models
    from datetime import datetime, timedelta
    from airflow.contrib.operators.dataflow_operator import DataFlowPythonOperator

    detault_args = {'start_date', '2021-01-01', 'owner': 'Airflow', 'retries': 1, 'retry_delay': timedelta(seconds=60), 'dataflow_detaul_options': {
        'project': 'bigquery-demo-2382',
        'region': 'us-central1',
        'runner': 'DataflowRunner'
    }}

    with models.DAG('food_orders_dag', default_args=default_args, schedule_interval='@daily', catchup=False):
        ta = DaataFrlowPythonOperator(
            task_id='beamtask',
            py_file='ga://location/of/file.py',
            options={'input': 'gs://daily_food_orders/foold_daily.csv'}
        )
    ```



## Section 16: Build Streaming data Pipelines

- GCP PubSub manged service
    - Publishers are the application that creates and sends message to topic
    - Topics receive message and store it in persistent storage
    - where it can be passed to subscriber
    - Message is combination of data and attributes (like labels?) that publisher sends to topic
    - Subscriber consumes the topic
    - Subscriptions are names resource representing stream of messages from single topic to be delivered to subscriber
    - All messages in topic are delivered to subscriber after subscription


- PubSub architecture advantages
    - Ensure each message is delivered exactly once
    - Less points of failure
    - Easily scalable


- Python code to create topic

    ```python
    from google.cloud import pubsub_v1
    pub = oubsub_v1.PublisherClient()
    import time

    topic_name = 'topic/name'

    pub.create_topic(topic_name)
    with open('file.csv') as f_in:
        for line in f_in:
            data = line
            future = publisher.publish(topic_name, data=data)
            print(future.result())
            time.sleep(2)
    ```

- Stream with beam

    - Building off last example

    ```python
    ...

    options = PipelineOptions(pipeline_args)
    options.view_as(StandardOptions).streaming = True
    p = beam.Pipeline(options=options)

    cleaned_data = (
        p
        | beam.io.ReadFromPubSub(topic=input_pattern)
        | ... process
    )

    ...

    ret = p.run()
    ret.wait_until_finish() # this is because pubsub is unbounded method of data ingestion
    if ret.state == PipelineState.DONE:
        print("DONE")
    ```



    - Doing things with threads in python

    ```python
    def create_view():
        view_name = "view_name"
        dataset_ref = client.dataset("dataset_name")
        ...

    from threading import Timer
    t = Timer(5.0, create_view)
    t.start()
    ```

## Section 17: BigQuery Pricing

- Storage prices
    - Active storage - has been modified in past 90 days
        - First 10 gb free
        - $0.02/gig per month
        - e.g. $5 for 500gb for half a month

    - Long-term storage - has not been modified in past 90 days
        - $0.01/gig per month

    - Querying table does not affect storage - only updating it ; DML, loading new rows

- Query prices

    - On-demand prices
        - First Tb free, then $5 per Tb

    - Flat-rate prices also available (ongoing machine rental)
        - Does not include storage
        - 100 slots cost about $2000 per month

    - Flex slots also available
        - Pre-purchase compute time
        - Compute time used at runtime

- Loading and DML
    - Loading data is free
        - Streaming data load is not free
        - DML incurs change for rows scanned
            - Insert changed for sum of bytes processed for all columns referenced
            - Update charged for sub of bytes processed for all columns referenced by query + sum of bytes for all columns of the updated rows
            - Delete changed for sum of bytes processed for all columns referenced + sum of bytes for all columns of rows deleted
            - If partitioned, only charge within scanned partition


- BigQuery storage API
    - Charged even if query is canceled
    - $1.1/Tb

- Free operations
    - Loading from gs or local file
    - No network egress charge while copying from gs (if dataset is US multi-region)
    - Copying table, exporting data, deletion operations
    - Metadata operations like list, get path, update and delete cells
    - Querying pseudo columns like _PARTIITONTIME
    - Creating / replacing UDFs

- GCP product cost tool available online

## Section 18: Best Practices / Optimization Techniques
- Optimize query
    - What's the I/O - how many bytes?
    - Shuffling - how many bytes in each slot?
    - Computation - how much CPU time?
    - Outputs - what gets written
    - Query anti-patterns - are you following SQL best practices?

- Reduce bytes read
    - Avoid "select \*" to avoid reading too many bytes
    - Set max bytes bulled
    - Use partitioning and clustering where possible
    - De-normalize data to use nested & repeated fields
    - Create materialized views for aggregated queries
    - Use cached results when possible
    - Avoid external data source when performance matters

- Reduce shuffling size
    - REduce amount of data before join clause
        - Join is shuffle oriented operation
        - Join tables in decreasing order of size for optimal broadcasting

- Reduce CPU time
    - Write transformed data into another table and perform operations on top of new table
    - Use native UDFs (not e.g. JS)
    - Use approx methods e.g. `approx_count_distinct()` in place of `count(distinct)`
    - Use denormalized tables in place of join where possible


- Query outputs
    - Use limits
    - avoid repeated overwrites

- SQL best practices
    - Avoid self-joins, use window functions instead
    - Avoid cross joins, pre-aggregate data or use window functions instead
    - Repartition the data if partition is skewed, e.g. "Other" bucket is large
    - Avoid DMLs for single row modifications
    - Batch inserts, updates, etc..
    - Set expiration times if table won't be needed later
    -


## Section 19: BONUS - Different File Formats & Apache Beam

- File types

    - File format feature trade offs
        - Read speed
        - Write speed
        - Splittable (run parallel tasks on file)
        - Schema evolution
        - Compression
        - e.g.
            - Plain text file (e.g. CSV) usually writes the fastest but reads the slowest


    - Text files
        - Good write, bad read (one of slowest)
        - No block compression
        - Splittable (on newline character)
        - No metadata for schema evolution
            - Old fields cannot be easily deleted


    - Sequence file
        - Key-value pairs stored in binary format
        - Better write than text file
        - Support block compression
        - Splittable
        - No metadata for schema evolution
            - Old fields cannot be easily deleted


    - Avro
        - Preferred format for loading data in BQ
        - Uses JSON for defining data types and then serializes data in binary format
        - Average read/write speeds
        - Supports block compression
        - Splittable
        - Most importantly it supports full schema evolution
            - This makes avro best choice if we know schema is going to be evolving (data types, field names, new fields, delete fields, etc...)
            - Old files can still be read with new schema, after a change

        - Columnar file formats
            - Columns are stored adjacent in addition to rows

    - RC file
        - Columnar
        - Flat files of binary key-value pairs
        - Fast reads with slower writes
        - Block compression (significant)
        - Splittable
        - Designed for faster reads therefore no schema evolution possible

    - ORC files
        - Better version of RC
        - Even better compression
        - Still no schema evolution

    - Parquet
        - Columnar, stores nested data structures in flat columnar storage
        - Faster reads and slow writes
        - Supports very good compression codecs
        - Splittable
        - Limited schema evolution
            - New views can be added but old ones cannot be deleted


- Apache beam
    - Started in 2016 and has become top apache project
    - Unified API to process both **batch and streaming** data
    - Portable - can be run on any execution framework (like spark, flink, cloud dataflow, etc..) and written in any language
    - Batch processing
        - Data is collected over time and then data is pushed to processing engine
        - e.g.
            - Reporting aggregates
            - Finding loyal bank customers
        - Batch job run at regular intervals on "bounded" dataset
        - Concerned with throughput (how much data can be processed at once)
    - Stream processing
        - Data fed to processing engine as soon as it's created
        - e.g.
            - Fraud detection
            - Sentiment analysis on social media posts
        - Streaming job runs continuously on "unbounded" dataset as data becomes available
        - Concerned with latency (how fast can records be handled)



