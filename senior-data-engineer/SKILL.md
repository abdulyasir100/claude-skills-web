---
name: senior-data-engineer
description: Production-grade data engineering expertise. Covers ETL/ELT pipeline design, batch vs streaming architecture, data modeling (dimensional, data vault), quality frameworks (Great Expectations, dbt tests), Spark tuning, Airflow/Prefect orchestration, and Kafka streaming patterns.
---

# Senior Data Engineer

Production-grade data engineering for building scalable, reliable data systems.

## Activation Triggers

- Pipeline design: ETL/ELT processes, data ingestion, extraction strategies
- Architecture decisions: Batch vs streaming, Lambda/Kappa patterns, lakehouse design
- Data modeling: Dimensional models, slowly changing dimensions, data vault
- Quality assurance: Validation frameworks, freshness monitoring, data contracts
- Performance optimization: Spark tuning, query optimization, pipeline execution

## Tech Stack

- **Languages**: Python, SQL, Scala
- **Orchestration**: Airflow, Prefect, Dagster
- **Transformation**: dbt, Spark
- **Streaming**: Kafka, Spark Structured Streaming
- **Storage**: S3, GCS, Delta Lake
- **Warehouses**: Snowflake, BigQuery, Redshift
- **Quality**: Great Expectations, dbt tests, data contracts
- **Monitoring**: Prometheus, Grafana

## Core Workflows

### Batch ETL Pipeline
1. Schema definition via SQL
2. Automated extraction configuration
3. dbt model creation with incremental logic
4. Data quality test setup
5. Airflow DAG orchestration
6. Validation procedures

### Real-Time Streaming
- Event-driven architectures using Kafka
- Spark Structured Streaming
- Schema validation with JSON schemas
- Late-arriving data handling via watermarking
- Dead letter queue patterns for error resilience

### Data Quality Framework
- Great Expectations integration
- dbt test suites
- Data contract specification
- Quality monitoring dashboards (completeness, freshness, duplicates)

## Architecture Decision Framework

- **Batch vs Streaming**: Real-time requirements, data volume, processing complexity
- **Lambda vs Kappa**: Two codebases (Lambda) vs single codebase (Kappa)
- **Warehouse vs Lakehouse**: Cost, schema flexibility, ecosystem maturity

## Common Troubleshooting

| Issue | Solution |
|-------|----------|
| Task timeouts | Resource allocation tuning |
| Memory errors | Executor scaling |
| Kafka consumer lag | Partition management |
| Duplicates | Deduplication logic |
| Schema drift | Merge strategies |
| Performance bottlenecks | Incremental materialization, query optimization |
