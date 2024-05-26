---
date: 2024-02-29
tags:
    - fact
hubs:
    - "[[sql]]"
    - "[[database-concepts]]"
---

# SQL-order-of-operations

1. **FROM**
2. **WHERE**
3. **GROUP BY**
4. **HAVING**
5. window clauses
6. **SELECT**
7. **DISTINCT**
8. **UNION**
9. **ORDER BY**
10. **LIMIT**

This explains why, e.g.

- you cannot reference calculated values from SELECT in the WHERE or GROUP BY: these are executed first.
- you can filter on GROUP BY fields using HAVING
- you can ORDER BY fiels in your SELECT
- window clauses can only be used in SELECT and ORDER BY

