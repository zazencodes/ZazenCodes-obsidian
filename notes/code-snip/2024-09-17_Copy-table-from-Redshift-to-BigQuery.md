---
date: 2024-09-17
tags:
    - code-snip
hubs:
    - "[[sql]]"
    - "[[data-engineering]]"
---

# Copy table from Redshift to BigQuery

1. Unload from Redshift

```sql
unload ('select * from [REDSHIFT_TABLE_NAME]')
to 's3://[BUCKET_ID]/[TABLE_ID]'
credentials 'aws_access_key_id=[KEY];aws_secret_access_key=[KEY]'
delimiter '\t'
allowoverwrite
;

```

2. Upload to BigQuery
```bash
TABLE_ID=
PROJECT_ID=
BUCKET_ID=

aws s3 cp s3://$BUCKET_ID/$TABLE_ID ~/Downloads/$TABLE_ID --recursive

gcloud config set project $PROJECT_ID

# gsutil cp -r s3://$BUCKET_ID/$TABLE_ID gs://$BUCKET_ID/$TABLE_ID

SCHEMA_FILE=
DATE_FIELD=
CLUSTER_FIELDS='field1,field2'

# Create schema file, e.g.
#
# [
#   {
#     "name": "col_1",
#     "type": "STRING",
#     "mode": "NULLABLE"
#   },
#   {
#     "name": "col_2",
#     "type": "INTEGER",
#     "mode": "NULLABLE"
#   },
#     ...
# ]
bq show --schema --format=prettyjson $TABLE_ID > $SCHEMA_FILE

bq mk --table --schema $SCHEMA_FILE \
    --require_partition_filter \
    --time_partitioning_field $DATE_FIELD \
    --clustering_fields $CLUSTER_FIELDS \
    $TABLE_ID

# Load each CSV chunk, i.e. 0000_part_00, 0001_part_00, etc...
bq load --source_format=CSV --quote "" -F='\t' [--allow_jagged_rows] $TABLE_ID ~/Downloads/$TABLE_ID/0000_part_00
```

