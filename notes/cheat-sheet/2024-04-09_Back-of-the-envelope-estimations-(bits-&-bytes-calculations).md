---
date: 2024-04-09
tags:
    - cheat-sheet
hubs:
    - "[[system-design]]"
urls:
    - https://www.mauriciopoppe.com/notes/computer-science/system-design/back-of-the-envelope-calculations/
    - https://colin-scott.github.io/personal_website/research/interactive_latency.html
---

# Back of the envelope estimations (bits & bytes calculations)

## Bits and bytes

```
1 byte = 8 bits

1 kb = 10^3 bytes = 1,000 bytes
       = 1e-3 kb
       = 1e-6 mb
       = 1e-9 gb
       = 1e-12 tb
       = 1e-15 pb

       = 1 ASCII CHARACTER
       = 1/4 of a LONG INT (4 bytes)
       = 1/8 of a DOUBLE FLOAT (8 bytes)
```

## Latency

```
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1KB with Zippy                  2,000   ns
Read 1 MB sequentially from memory       6,000   ns        6 us .006 ms  10^-2 ms
SSD Random read                         16,000   ns       16 us .016 ms
Read 1 MB sequentially from SSD        100,000   ns      100 us   .1 ms  10^-1 ms
HDD Random read                      3,000,000   ns    3,000 us    3 ms
Read 1 MB sequentially from HDD      1,000,000   ns    1,000 us    1 ms  10^0  ms
Send 1 MB bytes over 1 Gbps network 10,000,000   ns   10,000 us   10 ms  10^1 ms
Read 1 MB sequentially from 1 Gbps  10,000,000   ns   10,000 us   10 ms  10^1 ms
Round trip within same datacenter      500,000   ns      500 us   .5 ms
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms
```

2017 numbers from https://colin-scott.github.io/personal_website/research/interactive_latency.html

## Cost

```
What    Amount  $/Month
CPU          1      $10
Memory    1 GB       $1
SSD       1 GB     $0.1
HDD       1 GB    $0.01
S3, GCS   1 GB    $0.01
Network   1 GB    $0.01
```

## Notes

```
• 1 SSD read ≈ 10 memory reads
• 1 HDD read ≈ 10 SSD reads
• Writes are 40 times more expensive than reads
• 1 request per second ≈ 100k requests / day
• 1 request per second ≈ 2.5M requests / month
• 10 requests per second ≈ 1M requests per day
• 6-7 world-wide round trips per second
• 2000 round trips per second within a data center
```
