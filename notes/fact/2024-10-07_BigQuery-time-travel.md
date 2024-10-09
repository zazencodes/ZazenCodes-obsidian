---
date: 2024-10-07
tags:
    - fact
hubs:
    - "[[bigquery]]"
---

# BigQuery time travel

## Query data at a point in time

```sql
SELECT *
FROM `mydataset.mytable`
  FOR SYSTEM_TIME AS OF TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 HOUR);
```

- 7 day window, by default (can set to 2-7 days)
    - Using a shorter time travel window lets you save on storage costs when using the physical storage billing model. These savings don't apply when using the (default) logical storage billing model.

## Restore a table from a point in time

- Use if the table has been deleted
- Offset options:
    - tableid@TIME where TIME is the number of milliseconds since the Unix epoch.
    - tableid@-TIME_OFFSET where TIME_OFFSET is the relative offset from the current time, in milliseconds.
    - tableid@0: Specifies the oldest available historical data.


```bash
bq cp mydataset.table1@1624046611000 mydataset.table1_restored
```


