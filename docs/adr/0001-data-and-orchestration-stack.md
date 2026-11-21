# 0001 — Data and orchestration stack

- **Status:** Accepted
- **Date:** 2026-11-21

## Context

The dataset is small by industry standards: roughly 10^5 lap records per season, plus telemetry and weather. Storage is not a scaling problem; **operability and traceability are**.

The project must run on modest hardware, be reproducible from a clean checkout, and expose the concepts that matter in a real MLOps stack: asset lineage, partitioned reprocessing, data versioning, experiment tracking.

## Decision

- **Storage and query:** Parquet files partitioned by season, event and session, queried with **DuckDB**. No database server.
- **Orchestration:** **Dagster**, modelling the pipeline as software-defined assets instead of a DAG of tasks.
- **Data versioning:** **DVC**, with an S3-compatible remote for raw snapshots and model artifacts. Git holds only pointers.
- **Tracking and registry:** **MLflow**, self-hosted, with the backend store and the artifact store kept separate.

## Consequences

- No database to operate, back up or patch; DuckDB reads Parquet in place.
- Asset definitions make lineage, partitions and backfills first-class, which matches how the data is actually shaped: one partition per event.
- Dagster introduces a real learning curve; resources and IO managers must be designed deliberately.
- DVC requires discipline: the working copy and the remote can silently diverge. A `dvc push` check belongs in CI.
- Running MLflow with a separate artifact store means owning at least one object store; MinIO provides the S3 API locally.

## Alternatives considered

- **Airflow** — industry default, but task-centric, operationally heavy for a single developer, and offers no native data lineage.
- **Prefect** — excellent developer experience and a simpler model, but weaker on asset lineage, partitioning and backfills, which are the point here.
- **PostgreSQL or ClickHouse** — a server and its backups for a workload that fits in memory. Unjustified.
- **Delta Lake or Apache Iceberg** — table formats with ACID and time travel, valuable at a scale and concurrency this project will not reach.
- **lakeFS** — bucket-level branching is elegant, but it is another service to run and the workflows here are linear.
