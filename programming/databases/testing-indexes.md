---
description: How to test indexes in MySQL without permanently applying them.
---

# Testing indexes

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

# Tricks

## [Covering indexes](https://dev.mysql.com/doc/internals/en/optimizer-index-join-type.html)

If all columns in the select list are in an index (order-independent), the optimizer will be able to retrieve values from the index itself rather than the actual row, avoiding the overhead of loading the row into memory. This can help for very wide tables.

# Notes

* Replicas can have different indexes than primary ([source](https://stackoverflow.com/questions/4412656/can-you-index-tables-differently-on-master-and-slave-mysql)). Though ideally you shouldn't need to run this on a production replica at all.
