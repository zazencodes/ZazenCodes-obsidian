---
date: 2024-05-16
tags:
    - fact
hubs:
    - "[[sql]]"
urls:
    - https://www.youtube.com/watch?v=f9lbm7HA7pw
---

# SQL joins - ON vs WHERE

- Joins execute before where clause ([[2024-02-29_SQL-order-of-operations]])
- Outer joins work by doing inner join first, and then adding outer rows

- Given the two facts above:
    - For inner joins
        - ON and WHERE will return equivalent results
    - For outer joins
        - ON filters before adding outer rows (i.e. it will not filter them)
        - WHERE filters after adding outer rows

