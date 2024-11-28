---
date: 2024-11-17
tags:
    - cheat-sheet
hubs:
    - "[[database]]"
    - "[[postgresql]]"
---

```bash
# Basic Connection
psql -U username -h hostname -d database  # Connect to specific database
psql -U postgres                          # Connect as postgres user
\q                                        # Quit psql
\c dbname                                 # Connect to database
\l+                                       # List all databases (and their sizes)
\dt+                                      # List all tables (and their sizes)
\d tablename                              # Describe table


# Database Operations
CREATE DATABASE dbname;                   # Create new database
DROP DATABASE dbname;                     # Delete database
CREATE USER username WITH PASSWORD 'pw';  # Create new user
ALTER USER username WITH PASSWORD 'pw';   # Change password

# Table Operations
CREATE TABLE table_name (                 # Create new table
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    created_at TIMESTAMP
);
DROP TABLE table_name;                    # Delete table
ALTER TABLE table_name ADD column_name datatype;  # Add column
ALTER TABLE table_name DROP column_name;          # Remove column

# Basic Queries
SELECT * FROM table_name;                 # Select all records
SELECT * FROM table_name LIMIT 10;        # Select first 10 records
INSERT INTO table_name (col1, col2) VALUES ('val1', 'val2');  # Insert record
UPDATE table_name SET col1='val' WHERE condition;             # Update records
DELETE FROM table_name WHERE condition;                       # Delete records

# Backup & Restore
pg_dump dbname > backup.sql              # Backup database
psql dbname < backup.sql                 # Restore database
pg_dump -Fc dbname > backup.dump         # Compressed backup
pg_restore -d dbname backup.dump         # Restore compressed backup

# DigitalOcean
SET default_transaction_read_only = OFF;

# Size of all tables in schema (sorted)
SELECT
    table_name,
    pg_size_pretty(pg_total_relation_size(quote_ident(table_name))),
    pg_total_relation_size(quote_ident(table_name))
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY 3 DESC;
```



