---
date: 2024-04-16
tags:
    - fact
hubs:
    - "[[bigquery]]"
---

# BigQuery update schema

```bash
bq show --schema --format=prettyjson dataset.table > schema.json

# Modify schema
vim schema.json


bq update --schema schema.json dataset.table
```
