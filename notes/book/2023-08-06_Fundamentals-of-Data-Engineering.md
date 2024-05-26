---
date: 2023-08-06
tags:
    - book
hubs:
    - "[[data-engineering]]"
---

# Fundamentals of Data Engineering

Author: Jose Reis & Matt Housley

## Definitions

- Data engineering
    - Development, implementation and maintenance of systems and processes that take in raw data and produce well structured information for downstream use cases
    - At the intersection of security, data management, DataOps, data architecture, orchestration and software engineering
- Data architecture
     - Design of systems to support the evolving data needs of an enterprise
- Data pipeline
    - A combination of architecture, systems, and processes that move data through the stages of the data engineering lifecycle


## Data Engineering Lifecycle

### Lifecycle considerations
- Source systems
    - How is data persisted in source system?
    - What rate is data generated?
    - How often are there errors, missing data, duplicates, etc..?
    - How are schema changes handled?
    - How frequently to pull data from source system?
    - Who is the data provider responsible for the source?
    - Does reading from the source effect it's performance?
    - Does the source have upstream dependencies?
- Storage
    - Can read/write throughputs required be achieved?
    - Are you using the storage system optimally? How does it work?
    - Can it handle anticipated future scale?
    - Is it a pure storage (object storage) or also queryable (data warehouse)?
    - How are you tracking master data, golden records data quality, data lineage for data governance?
    - Where is data located? How are you tracking regulatory compliance and data sovereignty?
- Ingestion
    - What volume of data typically arrives?
    - What format is the data in?
    - What is the data destination?
    - Is realtime streaming needed or is batch OK? What about micro-batch, e.g. each minute?
- Transformation
    - What's the cost and ROI of the transformation?
    - Is the transformation as simple and self-isolated as possible?

### Lifecycle misc
- Interoperability
    - Integration via APIs is more common than databases
        - e.g. ingest from salesforce API, store in s3, use snowflake API to load into table and then run transformation, offload to s3 where spark can consume
        - done with python scripts & orchestration tools
- DataOps
    - Automation - change mangement, CI/CD, configuration as code (DAGs)
    - Observability and monitoring - catch data issues before downstream users
    - Incident response - rapidly identify and fix root causes

## Data architecture
- Foundations of data architecture
    - Agility - flexible and easily maintainable, can evolve in response to business needs
    - Loosely coupled systems, e.g. APIs
        - Bezos 2002 API Mandate
            - 1. All teams must expose their data and functionality through service interfaces
            - 2. Teams must communicate through these interfaces
            - 3. No other interprocess communication allowed, e.g. direct linking, reads, shared memory, etc..
            - 4. The technology doesn't matter, HTTP, Pubsub, etc..
            - 5. All service interfaces must be designed to be externalizable, i.e. could be used by the outside world


## Source systems
- Types of source systems
    - Files
    - APIs
    - OLTP (transactional processing) application databases
    - OLAP (analytical processing) database
    - CDC (change data capture) events
    - Logs
    - Messages and streams
- Undercurrent impacts for source systems
    - Security
        - Is data in source encrypted at rest? During transit?
        - Do you have secure access? VPN, SSH keys, password manager with SOO provider, etc..
    - Data management
        - Governance - know who manages the source
        - Quality, schema (expect change)
    - DataOps
        - Observe source for data quality issues and have plan for response

## Storage
- Data catalog
    - Centralized metadata store
    - Data engineer is responsible for setup and maintenance
- Separation of compute from storage
    - e.g. AWS EMR with S3 and HDFS
        - Spin up HDFS clusters with EMR using s3 as file system
        - Save intermediate data to HDFS (EMR SSD drives attached to CPUs)
        - Final results written back to s3 for downstream consumers
    - e.g. Apache Spark querying Parquet data stored in s3
- Data storage lifecycle
    - Create automated lifecycle policies for data to reduce storage costs
        - e.g. If data has not been accessed for 1 month, move to infrequent access storage, if 180 days move to archival storage

## Ingestion
- Stream error handling
    - Dead-letter queues for unsuccessfully ingested events (e.g. too large, expired past TTL) - events are rerouted for later analysis and processing
- Object storage
    - Most optimal and secure way to handle file exchange
- Schema changes
    - Typically difficult and time consuming to change downstream schemas
    - Authors speculate that Git inspired tooling could enable multiple versions of tables to exist with different schemas, upstream transformations, etc.. There can be a "development" table which is tested before being merged into the "main" table
- Data privacy
    - Do you need to ingest sensitive data? Avoid issues by discarding it early in the data ingestion pipeline
    - Common practice to apply tokenization to anonymize identities for model training and analytics
        - If possible, hash data at ingestion time
        - Thoughtlessly hashing without salting and using other strategies may be insufficient (e.g. if an email can be hashed and cross referenced against your table then it's not that secure)
- DataOps
    - Monitor data pipelines for uptime, latency, data volumes
    - Build monitoring systems from the outset
    - For streaming, be aware of number of events generated per time interval of relevance and size of each event
    - Do you have good testing and deployment practices?
    - Data-quality testing
        - Logs to capture history of data change
        - Checks for nulls, field cohesion (e.g. unexpected category appears)
        - Statistical tests
        - Good exception handling on ingestion errors

## Queries, Modeling and Transformations
- Commits
    - Know how your database handles reads and writes
        - ACID, transactions, row locking (Postgres)
        - No write concurrency at all (BigQuery)
- Streaming data
    - CDC approach is critically limited (monitoring changed to production table to build analytics table)
    - Kappa architecture treats stream storage as real-time transport layer and database
        - Requires windowing technique - session, fixed-time, sliding, etc..
- Data model
    - Consistent labels using language of your business
- Normalization
    - Remove database redundancy and ensure referential integrity
    - DRY (don't repeat yourself) applied to database
    - BigQuery is denormalized
- Data warehouse
    - Inmon "subject oriented" data warehouse
        - e.g. ecommerce
        - source systems (orders, inventory, marketing) -> normalized data warehouse -> data marts (sales, marketing, purchasing) -> reports
- Transformations
    - Trade offs between SQL and Python/Spark
        - How difficult is it to do with SQL?
        - How readable and maintainable will it be with SQL?
        - Can transformation code be pushed into a custom library for future reuse across organization?

## Serving data for analytics
- Trust
    - A loss of trust is the silent death knell for a data project
    - Once trust is gone, earning it back is insanely difficult
    - Trust is built on *data validation* and *data observability*
- Data products
    - Getting started
        - Who will use the data, and how?
        - What do they expect when using it?
        - Who can I ask for help with understanding the data?
- Analytics
    - Operational analytics
        - Using data to take immediate action (unlike typical analytics, which is about finding actionable insights)
        - e.g. real-time application monitoring, alerting via text, email, etc..
    - Embedded analytics
        - Dashboards embedded in the application itself
        - Need low data latency, query performance and high concurrency
- Metrics layer
    - A tool for maintaining and computing business logic
    - Can live in BI tool or software that builds transformational queries, e.g. dbt
    - Helpful because provides more trust and democratizes understanding in the metrics

## Security and privacy
- One security or data breach can leave your business dead in the water and wreck your reputation. And it's your fault
- Strategies
    - Only collect sensitive data if needed downstream
    - Make it a part of the company culture, stay aware of best practices, keep it simple
    - Use principle of least privilege - minimum required access for minimum acceptable timespan
    - Have backups of data
- Example security polic
    - Credentials
        - Use single-sign-on (SSO) when possible to avoid passwords
        - Use multifactor auth
        - Don't re-use passwords
        - No credentials in code, ever
    - Devices
        - Have device management procedures to remotely wipe if lost
        - Use multifactor auth
        - Sign in to device using company email and credentials
        - Be careful screen sharing
        - Use "do not disturb" to prevent messages appearing when they shouldn't
    - Software
        - Restart browsers when update alert appears
        - Run minor OS upgrades regularly
        - Wait at least a week or two before upgrade to new major OS
- Technology
    - Encrypt data at rest and in transit when possible (e.g. HTTPS)
    - Monitor with event logging
        - Access - whose looking at what, and from where
        - Resources - disk usage patterns should stay regular
        - Billing - know about spikes in costs
        - Excess permissions - revoke long unused credentials
    - Networking
        - Understand what IP addresses and ports are open, and to who
    - Software
        - Be aware of what software libraries you're using and keep them up to date to avoid security vulnerabilities

## Future of data engineering
- Will remain in high demand as companies need to invest in solid data foundation before moving on to ML / AI
- Decline of complexity and rise of easy-to-use data tools
    - Have already seen this significantly, e.g. serverless, cloud managed databases, data exports, "direct connectors"
    - Data engineers will spend less time on low-level tasks in the data engineering lifecycle (managing servers, configs, etc..) and "enterprisey" data engineering will become more prevalent
- Could gradually coalesce around a handful of data interoperability standards
    - e.g. object storage as batch interface layer between data services
    - e.g. progressive file formats like Parquet and Avro replacing CSV (interoperability issues) and JSON (poor performance)
    - e.g. standard APIs for metadata cataloging (describing schemas and heirarchies)
    - Benn Stancil called for the emergence of standardized data APIs for building data pipelines and data applications
- Improved orchestration frameworks
    - Better data integration and awareness (e.g. data cataloging & lineage), IaC capabilities like Terraform (e.g. infrastructure specifications embedded in code), deployment features like GitHub actions
- Specializations will arise
    - ML data engineer, e.g. ML engineer who knows algorithms, model training and optimization techniques who can operationalize the full ML process, including monitoring data pipelines and data quality
    - Software data engineer, e.g. working on analytics data applications that require deeper understanding of data
- Rise of realtime data
    - Previously seen as expensive novelty, but will grow as business identify try utility and accessibility (e.g. cloud offerings) increases
    - ETL will shift toward STL (stream, transform, load)
    - This live data stack will be powered by OLAP databases purpose built for streaming
    - Data modelling will start happening more in the source generation systems for streaming
- Databases designed to handle OLTP and OLAP, to support applications and analytics simultaneously
- New class of tooling that allows interaction with spreadsheets on OLAP backend

## Continued learning
- Identify domain experts who can mentor you through the use of new technologies
- Read the latest books, blog posts and documentation
- Attend meetups and talks
- Keep an eye on vendor announcements

## Data serialization
- Row-based
    - CSV
        - Error prone and perform poorly
        - Include technical description of the serialization configuration for your CSV files
    - XML
        - Legacy only, slow, inefficient
    - JSON
        - Standard for APIs
        - Native support in Showflake, BigQuery, SQL Server
    - JSONL
        - Useful for storing data right after it is ingested from API or application
        - Poor performance
- Columnar
    - General benefits
        - Can read from subset of fields
        - Storing same field together allows for more efficient encoding (e.g. looking for common values and tokenizing)
        - Better data compression due to repeated values
    - General drawbacks
        - Accessing individual records is difficult
        - Updating data is a nightmare
    - Parquet
        - Designed for read/write in data lake
        - Bakes in schema
        - Supports nested fields
        - Can be used with various compression algorithms that have read/write trade offs
    - ORC
        - Legacy, was popular with Hive
    - Apache Arrow (in-memory serialization)
        - Software can store data in memory using complex objects and pointers, or using densely packed structures like Fortran and C arrays
        - Densely packed structures were limited to simple types (e.g. INT64) or fixed-width (e.g. VARCHAR(256))
        - Apache Arrow rethinks serialization by using a binary format that is suitable for in-memory processing and export over network
        - We can avoid needing to serialize and deserialize data when writing and reading data
        - Each column gets its own chunk of memory
        - Nested data is stored with shredding - each location in the schema of JSON gets its own column of memory
        - e.g. use case; store data file on disk, swap it directly to program address space (by using "virtual memory" i.e. in place, not in memory), and begin running a query against the data without any deserialization. We can easily increase performance by swapping chunks of the file into memory as we scan it and swapping them back out to avoid overflow
        - Being adopted by Apache Spark
        - Spawned Dremio, a new data warehouse product
- Compression
    - Lossless
        - Algorithms looks for redundancy and repetition in data then reencode data to reduce redundancy
        - traditional e.g. gzip, bzip2
        - new gen e.g. Snappy, Zstandard, LZFSE, LZ4 (common in data lakes)
    - Lossy
        - Similar idea but removed information
        - e.g. audio, images, video

## Cloud networking
- Cloud platforms allow incoming data for free but charge for outgoing (engress) data
    - Widely critisized and increases "stickyness" of platform
    - Can apply to data passing between availability zones in cloud platform itself
- Availability zone is smallest unit of network topology made visible to customers
    - May actually consists of multiple data centers
    - Typically get best performance if using a single zone
- Region is collection of two or more availability zones
    - Object storage typically regional (data passing between zones is invisible and not charged)
- GCP multi-region
    - Allows more availability
    - free data engress cross regions
    - Deliver data efficiently to users within multiregion
- VPC
    - Direct connection to cloud offers higher bandwidth and lower latency
    - Discounted engress (e.g. 2 cents/gig vs 9 cents/gig on AWS)
- CDN
    - Dramatic performance enhancement delivering data assets to public or customers
    - Offered by cloud providers or others, e.g. Cloudflare
    - Work best when delivering same data repeatedly



