---
date: 2023-05-03
tags:
  - book
hubs:
  - "[[gcp]]"
  - "[[bigquery]]"
  - "[[dataflow]]"
---

# Google Cloud Cookbook

# Index
- BigQuery
- Dataflow

# BigQuery

### Merging Tables without Duplicates

Merge data without duplication using `ON...WHEN MATCHED`
```sql
MERGE
    prod_data_table_id a
USING
    new_data_table_id b
ON
    a.join_key = b.join_key
    WHEN NOT MATCHED
    THEN
        INSERT ROW
```

### Selecting the Top-1 Result

BigQuery specific optimization instead of row_number

```sql
SELECT b.* FROM (
    SELECT
        ARRAY_AGG(
            a ORDER BY a.date ASC LIMIT 1
        )[OFFSET(0)] b
    FROM (
        SELECT
            date, d1, d2, ...
        FROM
            table_id
    ) a
);
```

### Restoring Deleted Version of Table
```
>>> date -u +%s
1681824687
>>> bq --location=us cp table_id@1681824400000 restored_table_id
```

# Dataflow

### A Basic Streaming Job

- Want to write sub/sub json to bigquery
- Setup the topic, bigquery table and a gs bucket for temp storage
- Create new dataflow job with "Pub/Sub Topic to BigQuery" template

### Streaming Pub/Sub Topic into BigQuery Merged Table

- Setup pub/sub topic with schema
```bash
>>> PROJECT_ID=<my_project>
>>> gcloud pubsub topics create events
>>> bq mk --location us weblog
>>> bq load --autodetect $PROJET_ID:weblog.userprofile ./bq_data.csv
>>> bq head -n 10 $PROJECT_ID:weblog.userprofile

>>> cat schema.yaml
- column: event_timestamp
  description: Pub/Sub event timestamp
  mode: REQUIRED
  type: TIMESTAMP
- column: ip
...

gcloud data-catalog entries update --lookup-entry='pubsub.topic.$PROJECT_ID.events' --schema-from-file=schema.json
export PROJECT_ID=$PROJECT_ID
python publish_pubsub.py
```
- Stream into BigQuery merge with Dataflow SQL UI
```sql
select
    events.event_timestamp,
    events.ip,
    ...
    table.age_bin,
    table.opted_into_marketing,
    ...
from pubsub.topic.project.events as events
inner join
bigquery.table.project.dataset.table as table
on events.user_id = table.id;
```

# AI

### Training a Model in SQL with BQML

```sql
create or replace model
    mydataset.mymodel
    options(model_type='linear_reg', input_label_cols=['fare_amount']) as
    select
        fare_amount,
        ...
    from mydataset.mytrainingdata
;

select * from my.evaluate(model mydataset.mymodel);

select * from ml.predict(model mydataset.mymodel, (
    select fare_amount, ... from mydataset.mytestdata
));
```

