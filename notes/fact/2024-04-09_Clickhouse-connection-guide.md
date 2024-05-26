---
date: 2024-04-09
tags:
    - fact
hubs:
    - "[[data-engineering]]"
---

# Clickhouse connection guide

## JDBC

1. Download JDBC driver from [GitHub release page](https://github.com/ClickHouse/clickhouse-java/releases). I used `clickhouse-jdbc-0.6.0-patch3-all.jar`
2. Configure with SQLWorkbenchJ
3. Use JDBC URL with format in clickhouse [JDBC specification](https://clickhouse.com/docs/en/integrations/java): `jdbc:ch://HOST_NAME/DB_NAME`

## Bash client

Install with e.g. [brew](https://formulae.brew.sh/cask/clickhouse)

```bash
clickhouse client --user=USER --password=PASSWORD --database=DATABASE --port=9001 --host=HOST

```


