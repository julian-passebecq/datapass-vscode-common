# DataPass Factory V1 — orchestration decision

Date: 2026-09-28  
Decision authoring context: GPT-5.6 Sol in ChatGPT, not GPT-6.

This document closes the open question: what sits above dlt/dbt/Polars/sklearn in Factory, and how do we keep the local prototype simple while still having a real visible DAG?

## Decision

**Dagster OSS is the reference execution/orchestration adapter for Factory V1.**

Factory still owns a small provider-neutral **semantic project DAG** so the Common Engine, Prototype Cloud, Hub and Factory UI share the same graph vocabulary. Dagster is the first runtime that executes that DAG.

The hierarchy is:

```text
Factory semantic DAG
  |
  +-- generate / source
  +-- dlt ingestion
  +-- dbt transformation group
  |      +-- internal dbt model DAG
  +-- DuckDB SQL
  +-- Polars/Pandas transforms
  +-- sklearn features/train/score
  +-- quality
  +-- optional FastAPI/Redis services
  +-- publish/demo outputs
          |
          v
Dagster OSS orchestration/runtime
```

The Factory UI renders the semantic DAG directly. Dagster's own UI is an optional deep operational view, not a mandatory embedded browser.

## Why Dagster

Factory needs an orchestrator that is:

- local/open-source friendly;
- developer-oriented;
- able to show an asset/job graph;
- appropriate for data assets rather than only cron/task scheduling;
- able to orchestrate Python code;
- able to sit above dbt rather than compete with dbt;
- able to integrate dlt ingestion;
- usable without Kubernetes;
- usable without a managed cloud service.

Dagster fits this better than the other candidates for Factory V1.

Official/current documentation reviewed for this decision:

- Dagster local quickstart and UI use `dg dev` for a complete local process/web UI.
- Dagster has first-class dbt integration.
- dlt documents native Dagster integration via `dagster-dlt`.
- Dagster models data assets, jobs, schedules, sensors and resources, which maps well to Factory's project graph.

## Why dlt is not the orchestrator

dlt remains highly useful, but its responsibility in Factory is:

```text
extract
normalize
load
```

It should be one node/group in the global DAG.

Example:

```text
Generate API data
   -> dlt ingest
   -> DuckLake Bronze
   -> dbt Silver/Gold
```

dlt state/load receipts become evidence and metrics for the semantic engine. dlt does not own the entire Factory project lifecycle.

## Why dbt is not the global orchestrator

dbt already has a very useful DAG, but it is the **transformation/model DAG**.

Factory needs to represent work before and after dbt:

```text
generator
  -> API
  -> dlt
  -> dbt
  -> Polars features
  -> sklearn training
  -> scoring
  -> report/dashboard
```

Therefore dbt is an expandable nested graph in the global Factory DAG.

Factory should ingest dbt artifacts such as manifest/run-results to expose:

- models;
- sources;
- tests;
- dependencies;
- statuses;
- materializations;
- lineage.

The UI should allow:

```text
global DAG
  -> click "dbt Gold"
     -> expand dbt model DAG
        -> click model
           -> source SQL + semantic understanding
```

## Why not Airflow V1

Airflow is powerful and should remain an import/adapter possibility later, especially when a real client repo already uses it.

It is not the Factory V1 reference because:

- setup and operational surface are larger than required for a local investor/demo prototype;
- scheduler/webserver/database concepts add friction;
- many Factory demos do not need production scheduler semantics;
- Factory wants a fast local asset/project understanding loop first.

Airflow support later can include:

- static DAG analysis in DataPass;
- import/mapping into Common Semantic IR;
- optional execution adapter when a project genuinely needs real Airflow.

Do not port Mosaic's Airflow simulator into Factory.

## Why not Meltano V1

Meltano is useful for ELT project/plugin management and Singer-style connectivity, but it overlaps with responsibilities we already assign to:

- dlt for ingestion;
- dbt for transformation;
- Dagster for orchestration;
- Factory/Common Catalog for tool recipes.

Current Meltano documentation still uses an orchestrator plugin and documents Airflow as its standard orchestration path. For Factory V1 that adds another project/plugin abstraction without solving a missing core need.

Meltano may later be:

- a cataloged alternative;
- an importer;
- a project adapter for client repos already using Meltano.

It is not a Factory core dependency.

## Why not build our own scheduler

Earlier planning proposed a "tiny Factory DAG runner".

Retain the **typed Factory DAG contract**, but do not turn it into a home-grown scheduler if Dagster can execute the graph.

Factory-owned responsibilities:

- strict DAG schema;
- stable semantic IDs;
- dependencies;
- nested groups/subgraphs;
- validation;
- source/evidence references;
- display/layout hints;
- mapping to Common Semantic IR;
- run receipt normalization.

Dagster-owned responsibilities:

- dependency execution;
- run state;
- orchestration;
- schedules if later needed;
- retries where configured;
- resource lifecycle;
- local operational logs.

This avoids rebuilding orchestration while preserving product independence.

## Dagster must remain an adapter, not the semantic source of truth

The semantic project model should not become "whatever Dagster Python code happens to contain".

Reasons:

1. Prototype Cloud needs to create proposals without importing Dagster.
2. DataPass must understand projects that use Airflow/Fabric/ADF/etc.
3. Hub needs provider-neutral capabilities.
4. A future Factory runtime may add another orchestrator.
5. The global graph must be serializable for AI/bridge/project comparison.

Recommended relationship:

```text
factory.project / Common IR
        |
        +-- compile/project to Dagster definitions
        |
        +-- Factory graph UI
        |
        +-- analysis/export
```

The first implementation does not need a universal compiler for every Dagster feature. Support the small Factory V1 node vocabulary.

## Local-minimalism rule

Before adding a service/database/runtime, ask:

> Can DuckDB, DuckLake, Parquet, Python or an in-process library already do this simply?

Priority:

1. DuckDB for analytical SQL/catalog/querying.
2. Local filesystem + Parquet for files.
3. DuckLake when lakehouse table semantics are actually useful.
4. Polars/Pandas for dataframe code.
5. dbt-duckdb for modeled SQL transformation.
6. dlt for ingestion.
7. sklearn for ML.
8. Dagster for the global DAG/orchestration.
9. FastAPI only for a meaningful API/service boundary.
10. Redis only for a meaningful queue/cache/stream/state boundary.
11. Docker Compose only when service processes are required.
12. MotherDuck only as optional remote/share target.

## Explicitly do not add by default

- MinIO;
- local S3 emulation;
- Postgres just to host metadata that DuckDB/SQLite/local files can handle;
- Kafka;
- Airflow;
- Kubernetes;
- Spark;
- fake Spark;
- distributed storage layers;
- multiple databases for architectural aesthetics.

A Factory scenario may opt into a component only because the scenario is trying to demonstrate the capability it represents.

## DuckLake instead of MinIO

For the V1 local lakehouse story, prefer:

```text
DuckLake metadata/catalog
  +
local filesystem Parquet
```

over:

```text
MinIO
  +
S3 configuration
  +
extra containers
  +
credentials/endpoints
```

The point of Factory is fast local materialization, not infrastructure cosplay.

If a future demo explicitly needs object-store semantics, add a separate optional object-storage profile.

## DAG visualization requirements

Factory must make the global DAG a first-class UI, not a hidden runtime implementation.

Global example:

```text
Generate telemetry
      |
      v
dlt ingest
      |
      v
Bronze DuckLake
      |
      v
dbt transformations
      |
      +----> Gold KPI --------+
      |                       |
      +----> Feature table -> sklearn train
                              |
                              v
                         predictions
                              |
                              v
                          dashboard
```

Clicking a node should show:

- native source file(s);
- semantic role;
- milestones;
- input/output datasets;
- run state;
- observed-local metrics;
- findings;
- evidence.

Clicking a dbt node should be able to expand the dbt subgraph.

## Reference execution profiles

### Minimal SQL demo

```text
generator -> DuckDB SQL -> dbt -> charts
```

No Docker. No Redis. No FastAPI.

### Local lakehouse demo

```text
generator -> dlt -> DuckLake -> dbt -> charts
```

No Docker required.

### ML demo

```text
generator -> DuckDB/DuckLake -> Polars -> sklearn -> predictions
```

No Docker required.

### Service/API demo

```text
generator -> FastAPI -> dlt -> DuckLake -> dbt
```

FastAPI may run as local process. Docker optional.

### Event-ish demo

```text
generator -> FastAPI -> Redis Stream -> consumer/dlt -> DuckLake
```

Docker Compose is appropriate for Redis/service isolation.

## Acceptance criteria for orchestration V1

1. Factory graph validates independently of Dagster.
2. The graph visibly sits above dbt.
3. A dbt step can expand into the dbt model DAG.
4. Dagster executes at least generator -> dlt/dbt -> Polars/sklearn in a real local fixture.
5. Factory records normalized run receipts with basis=observed and environment=local.
6. Factory UI remains usable without opening Dagster UI.
7. "Open in Dagster" can launch/route to local Dagster UI when available.
8. No Airflow, Kubernetes, Spark or MinIO is required.
9. Pure DuckDB/dbt projects run without Docker.
10. Common Semantic IR remains orchestrator-neutral.

This is the V1 orchestration decision unless a real implementation blocker appears.
