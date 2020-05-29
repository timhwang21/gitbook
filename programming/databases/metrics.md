# Metrics

This will be a loosely organized crash-course collection of facts about metrics in Rails.

## Characteristics of analytical queries

Analytical queries tend to...

- Have lots of JOINs
- Have lots of GROUP BYs
- Aggregate subqueries
- Concerned with many records, but only a small subset of any given record (columnar access benefits)

On a higher level, analytical queries are differentiated from "regular" application DB operations as being OLAP(online analytical processing) vs OLTP (online transaction processing).

### OLTP

OLTP deals with transactions (generally short and cheap), and needs high thoroughput. Primary concerns are query speed, data integrity (locking, etc.)

### OLAP

OLAP deals with historical data and analytical queries (generally infrequent and expensive). Data doesn't necessarily need to be up-to-date to the millisecond (or even complete -- some loss might be okay as noise). Can take minutes to complete.

## Dealing with analytical queries

There are many ways to approach this problem, and widely accepted solutions change quickly over time. For example, whereas in the past people would recommend ETL pipelines for heavy jobs, now it is "throw everything into Snowflake / BigQuery."

Some high level performance approaches, roughly in orders of complexity:

### Optimizing existing DB usage

Postgres, MySQL etc. can get remarkably far with proper indexing, etc. when it comes to analytics queries -- you probably don't have as much data as you think. However, benchmark!

#### Application level caching

Not much to say. Simply load the relevant data into Redis or something and query against that. Pay $$$. This is a more generic approach that can easily be combined with others (e.g. do computations in BigQuery but then store in Redis for reads).

#### Read replicas

Strategies like read replicas help, and are more accessible than ever -- Rails 6 has built-in multi-DB support. (Query objects are a nice way of specifying DB access at the controller level. Or, you can have an entire controller / namespace / etc. dedicated to just analytics, and specify that everything there uses the RO DB.

#### Vertical scaling

Up to a certain point (approx. 1TB data, 500M rows / table as a rule of thumb), you can just throw more money at this problem and vertically scale.

Strengths:

- No new infrastructure needed (no cross cloud, custom ETLs...)

Weaknesses:

- No real weaknesses besides these approaches failing once you hit a certain scale

### Rollup tables

Rollup aggregate data that you need to power metrics every minute / hour / day / whatever.

Can use [HyperLogLog](https://www.citusdata.com/blog/2017/04/04/distributed_count_distinct_with_postgresql/) to avoid some of the weaknesses and to simplify data storage.

[Tutorial](https://www.citusdata.com/blog/2017/06/30/efficient-rollup-with-hyperloglog-on-postgres/)

Strengths:

- No new infrastructure needed (just another table)
- Works well with data retention policies (can still do analytics after data has been scrubbed)
- The tables tend to be small so you can keep them in memory
- Rollup tables are easier to design in a way that supports a wider range of queries than materialized views
- More future proof -- maps better to future transition to data warehouse

Weaknesses:

- If you want to calculate distinct counts constrained by combinations of columns, you might have to duplicate rollup data into separate rollup tables
- Slicing and dicing in unintended ways can be hard (e.g. slice by hour when you roll per minute, count a different metric)
- Double counting (can be solved by aggregating at the end of every rollup period, but cannot provide results until period ends, and backfilling is harder)

### Materialized views

Downsides are that as you start supporting more analytics use cases, the amount of analytics Additionally, this is more logic that exists outside of the application level, further complicating deploys, etc.

Strengths:

- Performance
- Live queries

Weaknesses:

- Amount of views needed can easily grow exponentially as business logic becomes more complex

### Specialized analytics solutions

#### Analytics databases

Column-oriented DBs can perform better due to matching the common characteristics of analytic queries. Vertica, Spanner, etc. Kind of a halfway step to a full-blown data warehouse.

Strengths:

- Can give similar benefits of data warehouses at a much cheaper price

Weaknesses:

- Can require operations engineers; generally not managed (need to deal with configuring physical server etc.)

#### Data warehouses

Data warehouses are intended for analytics; however in addition to being used by analysts, they can power applications as read-only data sources.

"Big" data = petabyte-level ([common RDBMS can do well up to ~1TB](https://statsbot.co/blog/modern-data-warehouse/)).

Snowflake charges by execution time; BigQuery charges by query.

Strengths:

- Optimized for this use case

Weaknesses:

- $
- Additional support / maintenance / complexity burden of infrastructure (cross-cloud, etc.)

## Loose decision making framework

- Start with "transparent" application and DB level optimizations
- Based on business needs (real-time needed? etc.), decide on additional layers like materialized views, rollup tables, etc.
- Based on scale needs, decide on specialized analytics infrastructure

## Analytics in Rails

- Use separate controllers / namespacing instead of having `FooController#metrics` endpoints. Analytics logic should be distinct from business logic.
- Endpoint responses are usually very tailored for specific usecases, so are optimiezd for performance and for specific charts. Standardization is of lesser importance.
- Raw SQL will likely be more flexible and readable. ActiveRecord is optimized for application-level stuff (where the builder pattern is more useful).
