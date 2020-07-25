---
description: How to test indexes in MySQL without permanently applying them.
---

# Profiling

## Temporary indexes

The following snippet shows how to temporarily create an index, in order to evaluate its effect on execution plans.

```sql
-- source: https://www.mysqltutorial.org/mysql-transaction.aspx/
-- 1. disable automatic COMMITs within transactions
SET autocommit = 0;

-- 2. start a new transaction
START TRANSACTION;

-- EXPLAIN
EXPLAIN SELECT *; -- your query

-- 3. add index
ALTER TABLE foo ADD INDEX index_name (a,b,c);

-- EXPLAIN again to see improvements
EXPLAIN SELECT *; -- your query

-- 4. Reset your changes
ROLLBACK;
```

## Avoiding caching

MySQL has multiple levels of caching: first, query results themselves can be cached; second, recently loaded tables might be kept in memory. For accurate benchmark times, you want to avoid caching, and you want to flush tables from memory.

```sql
SET profiling = 1; -- enable profiling for current session

SET SESSION innodb_max_dirty_pages_pct = 0;
SET SESSION query_cache_size = 0;
SET SESSION query_cache_type = 0;

SELECT SQL_NO_CACHE foo...; -- do stuff with SQL_NO_CACHE after SELECT

FLUSH TABLES; -- reset between test runs

SHOW PROFILES\G -- print profiling information for session
```

# Tricks

## [Covering indexes](https://dev.mysql.com/doc/internals/en/optimizer-index-join-type.html)

If all columns in the select list are in an index (order-independent), the optimizer will be able to retrieve values from the index itself rather than the actual row, avoiding the overhead of loading the row into memory. This can help for very wide tables.

# Notes

* Replicas can have different indexes than primary ([source](https://stackoverflow.com/questions/4412656/can-you-index-tables-differently-on-master-and-slave-mysql)). Though ideally you shouldn't need to run this on a production replica at all.
