---
date: 2024-11-24
tags:
    - video
hubs:
    - "[[data-lake]]"
    - "[[data-engineering]]"
urls:
    - https://github.com/garystafford/tickit-data-lake-demo
    - https://youtu.be/LAovcGquDc4?list=RQnwMCBbt3lEZ0aRcajXDyWBKklyw
---

# AWS data lake basic setup and tech stack

- Basic AWS-based data lake architecture example
- Data flows from different sources to a centralized, structured, and queryable storage system in s3
- This setup is common for organizations aiming to build scalable, serverless data lakes for modern data analytics.

### **Notable take aways for me**
- Files in s3 should be previewed using "Query with s3 select" (not by downloading the raw files)
- Athena SQL can create data lake tables (persisted parquet files) that are partitioned (files grouped into these folders) to optimize for query patterns
- Glue Jobs can run pyspark scripts to do data transformations at scale

### **Flow Explanation:**
1. Data from PostgreSQL, MySQL, and SQL Server is ingested using AWS Glue Crawlers (step 1).
2. Crawlers feed metadata into the AWS Glue Data Catalog (step 2).
3. AWS Glue Spark Jobs process and transform the data, saving it in layers (Bronze → Silver → Gold) within the S3-based data lake (steps 3 and 4).
4. Analysts and applications can query the transformed data via Amazon Athena for insights (step 4).

### **Benefits:**
   - Centralized and scalable storage for all types of data.
   - Metadata-driven architecture for easier discovery and transformation.
   - Cost-effective querying with Athena, eliminating the need for dedicated databases.
   - Clear separation of raw, refined, and aggregated data layers for better governance and efficiency.

### **1. Data Sources:**
   - **PostgreSQL (SaaS Provider):**
     Represents software-as-a-service (SaaS) data stored in a PostgreSQL database.
   - **MySQL (E-commerce Platform):**
     Contains transactional or e-commerce data stored in MySQL.
   - **SQL Server (CRM System):**
     Manages customer relationship management (CRM) data stored in a SQL Server database.

### **2. AWS Glue Crawlers:**
   - AWS Glue Crawlers are responsible for:
     - Automatically scanning the source data from PostgreSQL, MySQL, and SQL Server.
     - Extracting metadata such as schema definitions (column names, data types, etc.).
     - Creating or updating entries in the **AWS Glue Data Catalog**, which serves as a central metadata repository for all datasets.

### **3. AWS Glue Data Catalog:**
   - This catalog acts as the foundation for data discovery.
   - It stores information about the structure, location, and format of datasets scanned by the Glue Crawlers.

### **4. AWS Glue Spark Jobs:**
   - Glue Spark Jobs process and transform data:
     - These jobs are triggered to read raw data from the Glue Data Catalog.
     - Transformation steps might include filtering, cleaning, enriching, or aggregating the data.
   - The processed data is written to an Amazon S3-based data lake.

### **5. Amazon S3-Based Data Lake:**
   Data in the lake is stored in three distinct layers:
   - **Bronze (Raw Data):**
     - Raw data ingested as-is from the sources and stored in Apache Avro format.
   - **Silver (Refined Data):**
     - Intermediate-level data that has undergone cleaning and enrichment; stored in Apache Parquet format for better compression and performance.
   - **Gold (Aggregated Data):**
     - Fully transformed and aggregated data optimized for reporting and analytics; partitioned and stored in Apache Parquet format.

### **6. Querying with Amazon Athena:**
   - **Amazon Athena** provides ad-hoc querying capabilities on the S3-based data lake.
   - Users can write SQL queries directly against the Gold, Silver, or Bronze datasets for business intelligence, reporting, and analytics purposes.

