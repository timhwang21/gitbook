# Metrics

This will be a loosely organized collection of facts about building analytics and metrics.

## Characteristics of analytical queries

Analytical queries tend to...

* Have lots of JOINs
* Have lots of GROUP BYs
* Aggregate subqueries
* Concerned with many records, but only a small subset of any given record \(columnar access, basically\)

On a higher level, the distinction between analytical queries and "regular" application DB operations is represented as OLAP \(online analytical processing\) vs. OLTP \(online transaction processing\).

### OLTP

OLTP deals with transactions \(generally short and cheap\), and needs high thoroughput. Primary concerns are query speed, data integrity \(locking, etc.\)

### OLAP

OLAP deals with historical data and analytical queries \(generally infrequent and expensive\). Data doesn't necessarily need to be up-to-date to the millisecond \(or even complete -- some loss might be okay as noise\). Can take minutes to complete.

## Dealing with analytical queries

There are many ways to approach this problem, and widely accepted solutions change quickly over time. For example, whereas in the past people would recommend ETL pipelines for heavy jobs, now it is "throw everything into Snowflake / BigQuery."

Optimizations can loosely be categorized into three scaling tiers:

1. Optimizing existing DB usage: standard app or DB level tricks to eke out more performance
2. Precomputation: precalculating some data and storing it in a format more efficient to query for analytical purposes
3. Specialized analytics solution: addition of new analytics-specific infrastructure

Loose decision making framework:

1. Start with "transparent" application and DB level optimizations and see how far you go
2. Based on business needs \(real-time needed? etc.\), decide on additional layers like materialized views, rollup tables, etc.
3. Based on scale needs, decide on specialized analytics infrastructure

### Optimizing existing DB usage

Postgres, MySQL etc. can get remarkably far with proper indexing, etc. when it comes to analytics queries -- you probably don't have as much data as you think. However, benchmark!

#### The "standard tricks"

Indexes. Composite indexes on the attributes you want to query on. Standard downside of indexes: slower writes, more space needed.

Writing raw SQL will likely be more flexible and readable than long ActiveRecord chains. ActiveRecord is optimized for application-level stuff \(where the builder pattern is more useful\).

Query optimization.

#### Application level caching

Not much to say. Simply load the relevant data into memory \(Redis etc.\) and query against that. This is a more generic approach that can easily be combined with others \(e.g. do computations in BigQuery but then store in Redis for reads\).

#### Read replicas

Strategies like read replicas help, and are more accessible than ever -- Rails 6 has built-in multi-DB support. \(Query objects are a nice way of specifying DB access at the controller level. Or, you can have an entire controller / namespace / etc. dedicated to just analytics, and specify that everything there uses the RO DB.

#### Vertical scaling

Up to a certain point \(approx. 1TB data, 500M rows / table as a rule of thumb\), you can just throw more money at this problem and vertically scale. However, you may eventually be spending more money than it costs to use a dedicated solution.

Strengths:

* No new infrastructure needed \(no cross cloud, custom ETLs...\)
* Conceptually simpler; easier to adapt to emerging needs

Weaknesses:

* No real weaknesses besides not working once you hit a certain scale

### Precomputation

#### Rollup tables

Rollup aggregate data that you need to power metrics every minute / hour / day / whatever. The data are stored in an easily queryable format, with aggregate statistics precomputed.

One large downside is the double-counting problem, where one event may exist in two different rollup periods. There are various workarounds, none of which are perfect:

* Only rollup data at the end of the rollup period. Conceptually simple, but you lose real-time updating \(only updates once every rollup\) and can be hard to backfill data due to needing to cumulatively update rollups \(e.g. if you are tracking totals and not deltas\).
* [HyperLogLog](https://www.citusdata.com/blog/2017/04/04/distributed_count_distinct_with_postgresql/) lets you slice your data in a more flexible manner; however, it is conceptually more complex, and is an approximation algorithm so doesn't work well with small datasets \(which begs the question, if your data are small why not do something simpler in the first place\). [Tutorial](https://www.citusdata.com/blog/2017/06/30/efficient-rollup-with-hyperloglog-on-postgres/)

Another weakness important to note is the difficulty of handling entities that can change over time. As a simple case, we have a `Processable` model that asynchronously transitions from `pending` to `processed`. If an item is rolled up while pending but is then later processed, we need to go back and update the old count. If the processing time is short and predictable, we can get away with simply looking an additional period \(or periods\) back and recomputing the rollup. However if the processing time is unpredictable \(e.g., anything requiring manual human action\), we cannot do this solely at the DB level and would have to handle rollups with logic like "query every item that has updated since the last rollup period; handle items created after the rollup period normally; take items created before the rollup period and reconcile them back into existing rollups".

Strengths:

* The tables tend to be small so you can keep them in memory
* Rollup tables are easier to design in a way that supports a wider range of queries than materialized views
* More future proof -- maps better to future transition to data warehouse

Weaknesses:

* Hard to handle data that changes over time
* Without HLL, slicing and dicing in unintended ways can be hard \(e.g. slice by hour when you roll per minute, count a different metric\); may need to store redundant rollup data in separate tables
* Without HLL, your data will always lag by the rollup period
* Bad data can propagate through rollup entries and be hard to fix or even detect

#### Materialized views

\[Fill. Needs more research.\]

Strengths:

* Performance
* Live queries

Weaknesses:

* Amount of views needed can easily grow exponentially as business logic becomes more complex
* Logic exists outside of application level, which complicating deploys and adds another surface area for issues

### Specialized analytics solutions

\[Fill. Needs more research.\]

#### Analytics databases

Databases can be optimized for analytics by targeting the specific characteristics of analytic queries. For example, Vertica is a column-oriented database \(making it easy to compute sums\) that can also be optimized for time-series data \(by storing new rows as deltas of previous row\).

Multidimensional DBs store data in a multidimensional array as opposed to a relational DB, and precompute a "data cube" which is an easily manipulated and queryable representation of the data.

Strengths:

* Can give similar benefits of data warehouses at a much cheaper price

Weaknesses:

* Can require dedicated operations engineers; generally not managed \(need to deal with configuring physical server etc.\)

#### Data warehouses

Data warehouses are intended for analytics; however in addition to being used by analysts, they can power applications as read-only data sources.

"Big" data = petabyte-level \([common RDBMS can do well up to ~1TB](https://statsbot.co/blog/modern-data-warehouse/)\).

Snowflake charges by execution time; BigQuery charges by query.

Strengths:

* Modern solutions are entirely managed

Weaknesses:

* $$$
* Additional complexity burden of infrastructure \(cross-cloud, etc.\)

## Time series queries

Surprisingly, there are not popular tools for simplifying app-level computations or post-processing related to time series data \(at least for Rails\).

### Aggregation period

Most databases have functions like `DAY()` or `WEEK()`, etc. to `GROUP BY`. However, with this approach it is hard to aggregate by periods that aren't predefined \(e.g. group by every 15 days\).

This can be done in a fairly complex way solely at the DB level via [temporary tables and other tricks](https://stackoverflow.com/questions/232387/in-sql-how-can-you-group-by-in-ranges). However, there is also an interesting way to do this at the app level.

The process involves taking the start and end points and the period, and computing all timestamps based on the period you want. Then, you can query each time range independently and combine the results. While this results in much more DB connections, it is an approach that is naturally map-reducable over a number of machines.

## Code style

* Composability and performance can be antithetical in analytics. If forced to choose between the two, copy-pasting + robust tests is usually better.

## Organization evolution

Organizational battle stories, growing pains, etc.

> "I think Materialized Views are like duct tape. You can solve just about any problem with a materialized view, but if you use too many of them, you get a thing that just sucks... My current strategy is to get everything out of the OLTP database as quickly as possible. Using streaming with services like Kinesis or Kafka or whatever is great for that. For me the target is S3. Once stuff is in S3, you can process it however you want. Process the files in Aurora, EMR, or Redshift. Whatever makes sense."
>
> "It is a nightmare if something is missing or incorrect and cascades into future metrics and even our internal analysts have a hard time tolerating this. For external customers you probably want cached computed metrics!"

## Analytics in Rails

* Use separate controllers / namespacing instead of having `FooController#metrics` endpoints. Analytics logic should be distinct from business logic.
* Endpoint responses are usually very tailored for specific usecases, so are optimized for performance and for specific charts. Standardization is of lesser importance.
* Raw SQL will likely be more flexible and readable. ActiveRecord is optimized for application-level stuff \(where the builder pattern is more useful\).
* Code data transforms by hand. Each result is likely bespoke enough where there isn't a one-size-fits-all formatting / data cleaning function.
* If using rollups, be careful that your jobs don't interfere with the operational application if they are expensive. You don't want an analytics-related job crippling unrelated parts of the app. Keep the analytics architecture decoupled from the application's critical path.

