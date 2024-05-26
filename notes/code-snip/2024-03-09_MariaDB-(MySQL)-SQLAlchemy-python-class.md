---
date: 2024-03-09
tags:
    - code-snip
hubs:
    - "[[python]]"
---

# MariaDB (MySQL) SQLAlchemy python class

```python
from sqlalchemy import (
    create_engine,
    Table,
    MetaData,
    Column,
    Integer,
    Float,
    String,
    DateTime,
)

from urllib.parse import quote
import pandas as pd
from typing import List
from copy import deepcopy

import logging

logger = logging.getLogger(__name__)


TABLE_COLUMNS = dict(
    table_name=[
        Column("id", Integer, primary_key=True),
        Column("time_processed", DateTime),
        Column("email_id", Integer),
        Column("pred_label", String),
        Column("sentiment_model_1", String),
        Column("sentiment_score_1", Float),
    ],
)


def _get_table_columns(table_name: str) -> List[Column]:
    return deepcopy(TABLE_COLUMNS[table_name])


class MariaDBDataBase:
    def __init__(self, db_host, db_port, db_name, db_user, db_pass):
        self.db_url = (
            f"mysql+pymysql://{db_user}:{quote(db_pass)}@{db_host}:{db_port}/{db_name}"
        )

    def __enter__(self):
        self.db_engine = self._load_db_engine()
        return self

    def __exit__(self, *args, **kwargs):
        self.db_engine.dispose()

    def _load_db_engine(self):
        return create_engine(self.db_url)

    def read_sql_as_dataframe(self, query):
        logger.debug(query)
        df = pd.read_sql(query, con=self.db_engine)
        logger.debug(f"Loaded {len(df)} rows")
        return df

    def write_json_rows(self, json_rows, table_name):
        with self.db_engine.connect() as conn:
            res = conn.execute(
                Table(table_name, MetaData(), *_get_table_columns(table_name))
                .insert()
                .values(json_rows)
            )
            conn.commit()

    def run_sql(self, stmt):
        with self.db_engine.connect() as conn:
            res = conn.execute(stmt)
            conn.commit()
        return res
```
