---
date: 2023-08-29
tags:
    - fact
hubs:
    - "[[database-concepts]]"
    - "[[sql]]"
---

# SQL Concepts

- Aggregation
    - All aggregates ignore nulls, except `count(*)`
 - Logical execution
    - Query is executed as follows:
        - FROM
        - WHERE
        - GROUP BY
        - HAVING
        - SELECT
        - ORDER BY
    - WHERE clause cannot have aggregates, that's what HAVING is for
    - SELECT fields/aggs available in ORDER BY part of query (nowhere else)
- Joins
    - Cross join - cross product
    - Inner join - 1) cross product + 2) ON criteria
    - Outer join - 1) cross product + 2) ON criteria + 3) outer rows
    - ON filters will remove "outer rows" before step 3 of outer join
    - WHERE is applied after all join steps
    - Generally, just use ON for join conditions and WHERE for filtering


