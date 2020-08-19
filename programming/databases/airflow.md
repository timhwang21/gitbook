---
description: Introduction to Airflow, a workflow management system from Airbnb open source
---

# Airflow

## What is it

While commonly associated with ETL, Airflow is actually a general-purpose workflow scheduler. Put simply, Airflow is a fit for _recurring_ tasks that need to be scheduled in relation to other tasks.

Good fits:

- ETL
- Monitoring and responding to cron jobs
- ML pipelines

Airflow DAGs are composed of _operators_ and _hooks_. Operators represent the runners of task to be performed. Graph nodes are composed of a task, and an operator that executes the task. Hooks allow external APIs to act as data inputs.

The entire DAG also has a scheduler, which controls how the entire DAG is run in a manner similar to cron.

If there are multiple entry points in the graph, they will execute in parallel.

## Philosophies

* All data processing is best expressed as code
* Data operations should be functional (idempotent) and treat data as immutable

# Followups

## Dagster + Airflow

[Fill]

Airflow runs a DAG of tasks over time.

Dagster can produce this DAG.

# Resources

* [Introduction to Apache Airflow -- Astronomer.io](https://www.astronomer.io/guides/intro-to-airflow/)
* [Getting started with Apache Airflow](https://towardsdatascience.com/getting-started-with-apache-airflow-df1aa77d7b1b)
