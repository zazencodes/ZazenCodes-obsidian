---
date: 2024-11-24
tags:
    - video
hubs:
    - "[[system-design]]"
urls:
    - https://github.com/sirupsen/napkin-math
    - https://youtu.be/IxkSlnrRFqc?si=wyGcOCJMM8ur8UnE
---

# Advanced napkin math framework

## Base rates

### Cost

Basic costs

| resource | cost |
|----------|------|
| cpu | $10 / core / month |
| memory | $1 / gb / month |
| ssd | $0.1 / gb / month |
| disk | $0.01 / gb / month |
| cloud-storage | $0.01 / gb / month |
| network | $0.01 / gb / month |

Detailed costs

| What                | Amount | \$ / Month | 1y commit \$ /month | Spot \$ /month | Hourly Spot \$ |
| --------------------| ------ | ---------  | ------------------ | ------------- | ------------- |
| CPU                 | 1      | \$15       | \$10                | \$2            |  \$0.005       |
| GPU                 | 1      | \$5000     | \$3000              | \$1500         |  \$2           |
| Memory              | 1 GB   | \$2        | \$1                 | \$0.2          |  \$0.0005      |
| Storage             |        |            |                    |               |               |
| ├ Warehouse Storage | 1 GB   | \$0.02     |                    |               |               |
| ├ Blob (S3, GCS)    | 1 GB   | \$0.02     |                    |               |               |
| ├ Zonal HDD         | 1 GB   | \$0.05     |                    |               |               |
| ├ Ephemeral SSD     | 1 GB   | \$0.08     | \$0.05             | \$0.05        |  \$0.07        |
| ├ Regional HDD      | 1 GB   | \$0.1      |                    |               |               |
| ├ Zonal SSD         | 1 GB   | \$0.2      |                    |               |               |
| ├ Regional SSD      | 1 GB   | \$0.35     |                    |               |               |
| Networking          |        |            |                    |               |               |
| ├ Same Zone         | 1 GB   | \$0        |                    |               |               |
| ├ Blob              | 1 GB   | \$0        |                    |               |               |
| ├ Ingress           | 1 GB   | \$0        |                    |               |               |
| ├ L4 LB             | 1 GB   | \$0.008    |                    |               |               |
| ├ Inter-Zone        | 1 GB   | \$0.01     |                    |               |               |
| ├ Inter-Region      | 1 GB   | \$0.02     |                    |               |               |
| ├ Internet Egress † | 1 GB   | \$0.1      |                    |               |               |
| CDN Egress          | 1 GB   | \$0.05     |                    |               |               |
| CDN Fill ‡          | 1 GB   | \$0.01     |                    |               |               |
| Warehouse Query     | 1 GB   | \$0.005    |                    |               |               |
| Logs/Traces    ♣    | 1 GB   | \$0.5      |                    |               |               |
| Metrics             | 1000   | \$20       |                    |               |               |
| EKM Keys            | 1      | \$1        |                    |               |               |

Blob read/write

 | Operation               | 1M      | 1000     |
 |----------------|---------|----------|
 | Reads          | \$0.4   | \$0.0004 |
 | Writes         | \$5     | \$0.005  |
 | EKM Encryption | \$3     | \$0.003  |


### Application

Key numbers relevant to your business

| Resource | Measurement |
|---|---|
|Median and P99 response time|x ms|
|Throughput|x RPS|
|Transactions|x RPS|
|Customers|x|
|...| |

### Performance

Assuming 64-bit machines, 64 byte single-threaded memory, 8 KiB SSD / disk

| Operation                           | Latency     | Throughput | 1 MiB  | 1 GiB  |
| ----------------------------------- | -------     | ---------- | ------ | ------ |
| Sequential Memory R/W               | 0.5 ns      | 10 GiB/s   | 100 μs | 100 ms |
| Network Same-Zone                   |             | 10 GiB/s   | 100 μs | 100 ms |
| ├ Inside VPC                        |             | 10 GiB/s   | 100 μs | 100 ms |
| ├ Outside VPC                       |             | 3 GiB/s    | 300 μs | 300 ms |
| Hashing, crypto-safe                | 100 ns      | 1 GiB/s    | 1 ms   | 1s     |
| Random Memory R/W                   | 50 ns       | 1 GiB/s    | 1 ms   | 1s     |
| Fast Serialization                  | N/A         | 1 GiB/s    | 1 ms   | 1s     |
| Fast Deserialization                | N/A         | 1 GiB/s    | 1 ms   | 1s     |
| Sequential SSD read                 | 1 μs        | 4 GiB/s    | 200 μs | 200 ms |
| Sequential SSD write, -fsync        | 10 μs       | 1 GiB/s    | 1 ms   | 1s     |
| Sequential SSD write, +fsync        | 1 ms        | 10 MiB/s   | 100 ms | 2 min  |
| Decompression                       | N/A         | 1   GiB/s  | 1 ms   | 1s     |
| Compression                         | N/A         | 500 MiB/s  | 2 ms   | 2s     |
| Sorting (64-bit integers)           | N/A         | 200 MiB/s  | 5 ms   | 5s     |
| Sequential HDD Read                 | 10 ms       | 250 MiB/s  | 2 ms   | 2s     |
| Blob Storage GET, 1 conn            | 50 ms       | 500 MiB/s  | 2 ms   | 2s     |
| Blob Storage PUT, 1 conn            | 50 ms       | 100 MiB/s  | 10 ms  | 10s    |
| Random SSD Read                     | 100 μs      | 70 MiB/s   | 15 ms  | 15s    |
| Serialization                       | N/A         | 100 MiB/s  | 10 ms  | 10s    |
| Deserialization                     | N/A         | 100 MiB/s  | 10 ms  | 10s    |
| Proxy: Envoy/ProxySQL/Nginx/HAProxy | 50 μs       | ?          | ?      | ?      |
| Network within same region          | 250 μs      | 2 GiB/s    | 500 μs | 500 ms |
| Premium network within zone/VPC     | 250 μs      | 25 GiB/s   | 50 μs  | 40 ms  |
| {MySQL, Memcached, Redis, ..} Query | 500 μs      | ?          | ?      | ?      |
| Random HDD Read                     | 10 ms       | 0.7 MiB/s  | 2 s    | 30m    |
| Network NA Central <-> East         | 25 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network NA Central <-> West         | 40 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network NA East <-> West            | 60 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network EU West <-> NA East         | 80 ms       | 25 MiB/s   | 40 ms  | 40s    |
| Network EU West <-> NA Central      | 100 ms      | 25 MiB/s   | 40 ms  | 40s    |
| Network NA West <-> Singapore       | 180 ms      | 25 MiB/s   | 40 ms  | 40s    |
| Network EU West <-> Singapore       | 160 ms      | 25 MiB/s   | 40 ms  | 40s    |


### Examples

#### Snapshot-restore failover

Is it feasible to fail voer a simple, 16GiB in-memory database by dumping it to disk, sending it over the network and restoring on target machine?

| task | time | note |
|---|---|---|
| Read 1 GiB of sequential memory | 100ms | intial database read |
| Write 1 GiB to SSD | 500ms | |
| Transfer 1 GiB between cloud regions | 1 minute (150Mbit/s) | limiting step |
| Read 1 GiB from SSD | 250ms | |
| Write 1 GiB from SSD | 250ms | |
| Write 1 GiB of RAM in 64-bit increments | 1.5 seconds | final database write |

#### Debugging an existing system

Why did it take 2-3 seconds to serve a response for some Australian merchants?



