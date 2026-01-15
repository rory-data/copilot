---
name: principal-data-engineer
description: Expert-level guidance on data platform architecture, pipeline design patterns, and engineering rigor. Use this skill when designing new data platforms, reviewing complex DAGs, or establishing engineering standards.
---

# Principal Data Engineer

## Overview

This skill provides the strategic and technical depth expected of a Principal Data Engineer. It moves beyond "making it work" to "making it scale, endure, and deliver value." Use this skill for architectural decisions, high-stakes code reviews, and establishing robust engineering patterns.

## Core Capabilities

### 1. Data Platform Architecture

Focus on the "-ilities": Scalability, Reliability, Maintainability, and Observability.

- **Design for Failure**: Assume every component will fail. Build retries, dead-letter queues, and circuit breakers.
- **Idempotency**: All pipelines must be re-runnable without side effects.
- **Decoupling**: Separate compute from storage; separate orchestration (Airflow) from execution (Spark/Snowflake/dbt).
- **Cost Awareness**: Design schemas and compute usage (e.g., partition strategies) to minimize cost at scale.

### 2. Pipeline Engineering Standards

Enforce strict standards for Airflow and Python code.

- **No Top-Level Code**: Strictly adhere to Airflow best practices to prevent scheduler overload.
- **Atomic Tasks**: Each task should do one thing. If it fails, it should be clear what failed.
- **Functional Patterns**: Prefer clear inputs and outputs over shared global state.
- **Testing**:
  - **Unit**: Test transform logic in isolation.
  - **Integration**: Test DAG integrity and component connectivity.
  - **Data Quality**: Validate data "in-flight" (pre-condition/post-condition checks).

### 3. Data Quality & Observability

Quality is not an afterthought; it is a pipeline dependency.

- **Data Contracts**: Use **datacontract-cli** with ODCS to define explicit contracts between producers and consumers.
- **Data Quality Checks**: Use **Soda** for declarative data quality validation integrated into pipelines.
- **SLA/SLO Monitoring**: Alert not just on failure, but on lateness (missing SLAs).
- **Data Lineage**: Ensure transformations are traceable from source to sink (OpenLineage, dbt docs).

### 4. Composable Data Stack

Leverage the composable data stack—swap any component without rewriting the entire pipeline.

- **Ingestion**: Prefer `dlt` for robust, schema-aware ELT. Use `dlt init <source> <destination>` to scaffold pipelines.
- **Processing**: Default to **DuckDB**, **Polars**, and **Apache Arrow** for single-node processing (faster/cheaper than Spark for small/medium data).
- **Embedded OLAP**: Use **DuckDB** for local development, testing, and file-based querying (S3/Parquet).
- **Portable Code**: Use **Ibis** to decouple transformation logic from execution engines (run on DuckDB locally, Snowflake in prod).
- **Open Table Format**: **Apache Iceberg** for lakehouse architectures—schema evolution, time-travel, partition evolution.
- **Transformation**: **dbt** for SQL-first transformations with built-in testing and documentation.
- **Data Contracts**: **datacontract-cli** with Open Data Contract Standard (ODCS) for producer/consumer agreements.

## Usage Guidelines

### When to use

- **Architectural Reviews**: "Review this proposed architecture for the new streaming platform."
- **Complex Debugging**: "The scheduler is lagging, and tasks are getting stuck. Help diagnose."
- **Standard Setting**: "Create a template for a standardized ingestion pipeline."

### Key Questions to Ask

- "Is this pipeline idempotent? What happens if I run it twice?"
- "How do we backfill historical data with this design?"
- "What is the recovery time objective (RTO) for this dataset?"

## Resources

### references/

- **[architecture-patterns.md](references/architecture-patterns.md)**: Common patterns for batch and streaming architectures.
- **[data-quality-checklist.md](references/data-quality-checklist.md)**: A checklist for ensuring data reliability.
- **[apache-arrow.md](references/apache-arrow.md)**: Arrow as the data spine—zero-copy, polyglot interoperability, ADBC, Flight, Parquet.
- **[single-node-vs-spark.md](references/single-node-vs-spark.md)**: When to use Polars/DuckDB vs Apache Spark. Decision trees and practical guidance.
- **[composable-data-stack.md](references/composable-data-stack.md)**: Guidance on dlt, Polars, DuckDB, and Ibis.

### scripts/

- **[validate_dag_integrity.py](scripts/validate_dag_integrity.py)**: Utility to check for common Airflow anti-patterns (e.g., top-level code).
