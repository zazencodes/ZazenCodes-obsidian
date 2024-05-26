---
date: 2023-09-16
tags:
    - book
hubs:
    - "[[data-engineering]]"
---
https://github.com/newfront/spark-moderndataengineering

Author: Scott Haines

"A Hands-On Guide for Building Mission-Critical Streaming Applications"

# Index
- General
- Data lakes
- Spark applications
- Airflow

# General
- **Data wrangling**: As a data engineer, your job hinges on the ability to easily wrangle data. You will be expected to think quick and figure out ways to ingest, parse, and validate new sources of data - transforming it into clean, well structured and reliable datasets (pg. 59)
- **Data Catalogs**: Data discovery is key to success for data-driven companies. As a data engineer, this should really matter to you (pg. 154)
- **Know the size of your data**: Understanding the size of the data you are working with, not only the total number of records, but also the memory and IO (bandwitdth) overhead (pg. 207)
- **Two kinds of problems**:
    - Bad upstream data and issues as data moves through pipelines
    - Meeting time requirements (data SLAs) (pg. 255)
- **Upstream data modeling**: In the real world, it's not uncommon for data engineers to be tasks with the job of creating the structured data formats and even writing the upstream systems producing the data fed into their own data domains (own the data contract, understand the data lineage) (pg. 331)



# Data lakes
- Pg 15-20
- Reliable elastic file system capable of scaling horizontally to meet storage needs of many datasets
- Unified source of truth for raw, unprocessed data across the organization, centralized staging area for data to be reliably re-read, re-processed, and transformed into actionable data (pg. 367)
- Data organized into dataset directories, often partitioned by date
- Medallion architecture (Bronze -> Silver -> Gold) denotes progressively improved structure and quality of data
    - use the notion of a bronze database for raw data
    - use the notion of a silver database for reliable data
    - use the notion of a gold database for production data
e.g.
```
- bronze
    - customerratings
        - _SUCCESS
        - part-000-823821.snappy.parquet
- common
    - customers
        - _SUCCESS
        - part-000-823221.snappy.parquet
- silver
    - coffee_orders
        - date=2023-09-05
            - part-000-842821.snappy.parquet
            - part-001-842821.snappy.parquet
            - part-002-842821.snappy.parquet
        - date=2023-09-06
            - part-000-341821.snappy.parquet
            - part-001-341821.snappy.parquet
            - part-002-341821.snappy.parquet
```
- Having source-of-truth data stored in a central place can be one of you most critical data assets. Valuable for answering questions, re-building tables, recovering from issues like downstream corruptions, etc..
## MinIO
- s3 clone that can run locally or in cloud, fully compatible with s3 API




# Spark applications

### Batch applications
- Chapter 7, source code: https://github.com/newfront/spark-moderndataengineering/tree/main/ch-07/app
- First (and most simple) batch application, result of many years of lessons learned, set up for future success (pg. 243)

## Testing
- Chapter 7, source code above
- Not just "nice to have", it's a requirement of the job
- Running spark in test environment opens doors to rapid application development
- Writing tests saves money on debugging in production

## Stream applications
- Stateful stream applications use checkpoints - analogous to save points, in case of failures. Events streaming in while spark is offline will get picked up when the application is turned on (pg. 350)
- Kafka provides similar function to data lake - source of truth event data without loss (pg. 367)
- One or seven days are common retention periods for Kafka topics, outside of which data is deleted from Kafka (pg. 378)




# Airflow

## Airflow on docker
- Chapter 8, source code: https://github.com/newfront/spark-moderndataengineering/tree/main/ch-08/airflow
- Need to include java library if running spark tasks (e.g. testing locally on mac)



