---
date: 2020-03-09
tags:
    - code-snip
hubs:
    - "[[python]]"
    - "[[sql]]"
    - "[[postgresql]]"
---

# SQLAlchemy run raw SQL

```python
def get_engine():
    import sqlalchemy
    import os
    # os.environ["DB_URL"] = "postgresql://USER:PASS@HOST:PORT/DATABASE"
    engine = sqlalchemy.create_engine(os.environ["DB_URL"])
    return engine

def close_engine(engine):
    engine.dispose()


# ALTERING / UPDATING
def update_table(engine):
    sql_cmd = """
    begin;
    -- SQL GOES HERE
    commit;
    """
    with engine.connect() as conn:
        result = conn.execute(sql_cmd)


# READING
def read_table(engine)
    import pandas as pd
    sql_cmd = "select * from target_table"
    df = pd.read_sql(sql_cmd, engine)



# WRITING
def s3_upload_df(df, table_name, filename, s3_id, s3_secret, s3_bucket):
    import uuid
    import s3fs
    import os
    # os.environ["S3_ID"] = "id"
    # os.environ["S3_SECRET"] = "secret"
    bytes_to_write = df.to_json(None, orient="records", lines=True).encode()
    fs = s3fs.S3FileSystem(key=os.getenv("S3_ID"), secret=os.getenv("S3_SECRET"), use_ssl=False)
    filename = f"{uuid.uuid4()}.json"
    s3_path = "s3://{}/{}/{}".format(s3_bucket, table_name, filename)
    with fs.open(s3_path, "wb") as f:
        f.write(bytes_to_write)
    return s3_path

def upload_table(df, engine, s3_bucket):
    df[:0].to_sql('target_table', engine, if_exists='fail', index=False) # schema = public
    s3_path = s3_upload_df(df, table_name="target_table", s3_bucket)
    sql_cmd = f"""
    copy {table_name} from '{s3_path}'
    with credentials as 'aws_access_key_id={aws_access_key_id};aws_secret_access_key={aws_secret_access_key}'
    json 'auto' truncatecolumns
    """
    with engine.connect() as conn:
        result = conn.execute(sql_cmd)
```
