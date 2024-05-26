---
date: 2024-04-10
tags:
    - code-snip
hubs:
    - "[[sql]]"
    - "[[data-engineering]]"
---

# Second highest value in SQL question

Basic solution, using cte + window function
```sql
WITH click_sums AS (
  SELECT name, SUM(clicks) AS clicks FROM <TABLE_ID>
  WHERE date = "2024-04-08" GROUP BY name
),
name_ranks as (
  SELECT name, RANK() OVER(ORDER BY clicks DESC) nr FROM click_sums
)
SELECT * FROM name_ranks
WHERE nr = 2;
```

More advanced solution with cross join to fetch source data for 2nd highest value record
```sql
WITH fltr_data AS (
  SELECT * FROM <TABLE_ID>
  WHERE date = "2024-04-08"
),
click_sums AS (
  SELECT name, SUM(clicks) AS clicks
  FROM fltr_data
  GROUP BY name
),
name_ranks AS (
  SELECT name, RANK() OVER(ORDER BY clicks DESC) nr FROM click_sums
)
SELECT * FROM fltr_data, name_ranks
WHERE fltr_data.name = name_ranks.name
AND name_ranks.nr = 2;

```

Note: `ROW_NUMBER` works too, but it will arbitrarily pick one in case of a tie
