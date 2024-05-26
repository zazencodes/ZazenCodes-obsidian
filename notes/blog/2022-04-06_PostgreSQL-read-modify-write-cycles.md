---
date: 2022-04-26
tags:
    - blog
hubs:
    - "[[database-concepts]]"
    - "[[postgresql]]"
    - "[[sql]]"
urls:
    - https://www.2ndquadrant.com/en/blog/postgresql-anti-patterns-read-modify-write-cycles/
---

# PostgreSQL read modify write cycles

Imagine your code wants to look up a user’s balance, subtract 100 from it if doing so won’t make it negative, and save it.

- The wrong way (concurrent txns will produce wrong result):

    ```sql
    SELECT balance FROM accounts WHERE user_id = 1;
    -- in the application, subtract 100 from balance if it's above
    -- 100; and, where ? is the new balance:
    UPDATE accounts SET balance = ? WHERE user_id =1;
    ```

4 strategies to solve this:

- Avoiding the read-modify-write with a calculated update by doing calculation in pure SQL, e.g. update X set Y=Y-100

- Row level locking with SELECT ... FOR UPDATE. This will cause concurrent txn to wait on read, solving the issue.

    ```sql
    SELECT balance FROM accounts WHERE user_id = 1 FOR UPDATE;
    ```

- Use of SERIALIZABLE transactions. This will cause concurrent txns to fail on update, since same row is changed earlier. Needs to be handled by application code.

    ```sql
    BEGIN ISOLATION LEVEL SERIALIZABLE;

    SELECT balance FROM accounts WHERE user_id = 1;
    ```

- Optimistic concurrency control, otherwise known as optimistic locking. i.e. use versioning. This will cause concurrent txns to fail, needing to be handled by application code.

    ```sql
    SELECT balance, version FROM accounts WHERE user_id = 1;
    UPDATE accounts SET balance = 200, version = 2 WHERE user_id = 1 AND version = 1;
    ```

