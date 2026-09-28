# DataPass Factory V1 — orchestration decision (revised)

Date: 2026-09-28
Decision authoring context: GPT-5.6 Sol in ChatGPT, not GPT-6.

This revision supersedes the earlier decision that made Dagster the mandatory Factory V1 runtime.

## Final decision

**Factory V1 should use our own bounded local orchestrator, built from the existing Mosaic / Factory Lab control-flow engine and graph UI.**

Dagster is not a required Factory V1 dependency. It may remain a future optional adapter or export target if we later need its richer scheduling/assets/runs ecosystem.

The reason is the final Factory scope:

- 100% local;
- no hosting;
- no managed service;
- no remote deployment;
- no Kubernetes requirement;
- manual/in-app runs are enough for V1;
- bounded node vocabulary;
- local files + DuckDB/DuckLake;
- dbt/dlt/Python/Polars/Pandas/sklearn;
- optional FastAPI/Redis/Docker only for scenarios that genuinely need them.

Under these constraints a general external orchestrator adds more infrastructure than product value.

---

# 1. Fabric is the conceptual reference

Microsoft Fabric Data Factory uses pipelines as the orchestration layer: activities are connected through dependencies and can use success/failure/completion paths, retries, parameters, conditions, loops, schedules/triggers, run IDs and monitoring. Transformation activities may invoke notebooks, SQL, Dataflows, Spark jobs and dbt while the pipeline remains the end-to-end workflow.

That mental model is almost exactly what Factory needs, except our activities execute local technologies.

---

# 2. Existing donor: Mosaic already contains most of the control-flow kernel

The current datapass-mosaic-vscode repository already contains:

### UI

- SharedGraphCanvas;
- FactoryPipelines.tsx;
- PipelineSurface.tsx;
- activity boxes and settings;
- status rendering;
- run/task result panels;
- left-to-right DAG layout;
- draggable positions.

### Control-flow engine

runtime/factorylab/engine.py already models:

- activity dependencies;
- success/failure/completed dependency conditions;
- retries and retry delay;
- timeouts;
- inactive/skipped states;
- If;
- Switch;
- ForEach;
- Until;
- child pipelines;
- parameters;
- variables;
- run IDs;
- activity/run state;
- result propagation.

### Local execution boundary

The engine already has a Workspace protocol and can execute some work activities against the local catalog while simulating unsupported activities.

This means we should not start orchestration from a blank page. We should extract the useful control-flow semantics, remove learning/simulation assumptions from the production Factory path, and add real local adapters.

---

# 3. Target architecture

~~~text
Factory project
    |
    v
Factory Semantic DAG
    |
validation / compiler
    |
    v
Factory Local Orchestrator
    |
    +--> dlt / dbt
    +--> DuckDB / DuckLake
    +--> Polars / Pandas
    +--> Python / sklearn
    +--> quality / export
    |
    v
observed local run receipts
~~~

The same Semantic DAG drives the Factory graph, execution, Common IR, lineage, statuses and findings.

---

# 4. The global DAG remains above dbt

~~~text
generator
   ↓
dlt ingest
   ↓
dbt group
   ├─ sources
   ├─ staging
   ├─ intermediate
   ├─ marts
   └─ tests
   ↓
Polars/Pandas features
   ↓
sklearn train/score
   ↓
publish/demo
~~~

dbt remains an expandable nested transformation DAG. Factory should ingest manifest.json and run_results.json to expose model dependencies, sources, tests, materializations, statuses and file links.

---

# 5. What our own orchestrator means

It does not mean recreating Airflow, Dagster or Kubernetes.

Factory V1 is intentionally bounded.

Required V1 semantics:

- graph validation;
- cycle refusal;
- dependency ordering;
- parallel execution of independent ready nodes;
- success/failure/always edges;
- retries;
- timeout;
- cancellation;
- run/activity state;
- timestamps/duration;
- stdout/stderr/log references;
- deterministic run IDs;
- normalized outputs/receipts;
- manual run;
- run selected;
- run from here;
- retry failed.

Useful later, not required for first vertical slice:

- If;
- ForEach;
- Switch;
- Until;
- local schedules while Factory is open.

Explicitly not V1:

- distributed workers;
- backfill platform;
- high availability scheduler;
- remote agents;
- worker queue cluster;
- multi-tenant server;
- Kubernetes executor;
- cloud deployment;
- mandatory OS daemon.

---

# 6. Runtime implementation

Recommended scheduler: Python asyncio / task graph.

Flow:

1. validate the Semantic DAG;
2. determine ready nodes;
3. launch independent ready nodes concurrently up to a local limit;
4. collect result state;
5. unlock downstream nodes;
6. handle retry/timeout/cancellation;
7. persist normalized receipts.

For project code and external tools, prefer subprocess isolation rather than running everything in the orchestrator process.

Each adapter receives explicit cwd, executable/entrypoint, bounded environment, timeout, cancellation and captured stdout/stderr.

Simple local persistence:

~~~text
.factory/
  runs/
    <run-id>/
      run.json
      events.jsonl
      logs/
        <step-id>.log
~~~

No Postgres, Redis or metadata service is needed for run metadata.

---

# 7. V1 activity adapters

- generator — trusted Factory scenario/generator;
- duckdb-sql — real local SQL;
- dbt — allowlisted dbt Core commands and artifact capture;
- dlt — explicit local ingestion entrypoint/pipeline;
- python — explicit trusted local Python;
- polars — semantic specialization of Python;
- pandas — semantic specialization of Python;
- sklearn — train/score specialization;
- quality — SQL/dbt/Python checks;
- export — CSV/Parquet/local output.

Optional later:

- FastAPI service;
- Redis service/stream;
- Docker Compose service group;
- Apache Hop pipeline;
- Dagster export/adapter;
- Airflow import/adapter.

Several semantic activity types may share one subprocess implementation. The type exists so DataPass/Factory can understand the project, not because each type needs its own scheduler.

---

# 8. Why Dagster is no longer mandatory

Dagster is credible and useful, but mandatory Dagster would introduce:

- a second asset/object model;
- a second graph UI;
- webserver/daemon processes;
- another project configuration layer;
- mapping from our Semantic DAG to Dagster definitions;
- another run/event model that we immediately normalize back into Factory;
- extra install/version surface.

That cost made sense when Factory might become a general orchestration platform. It no longer makes sense after the scope was tightened to a local-only demonstrator.

Use Dagster later only if a real need appears that is expensive to reproduce in our bounded kernel.

---

# 9. Why not Airflow

Airflow solves a much larger problem: persistent scheduler, metadata database, webserver, executors/workers, backfills and rich scheduling semantics.

Keep Airflow for DataPass analysis/import and for real client repositories that already use it. Do not make it Factory V1 core and do not port Mosaic's Airflow simulator.

---

# 10. Why not Kubernetes / k3s

Kubernetes and k3s orchestrate containers/services. They do not provide the data-pipeline semantics we need: dbt model dependencies, ETL activity success/failure edges, local lineage or step parameters.

For one laptop they add cluster lifecycle, networking, storage abstraction, manifests and security/configuration overhead.

Final rule: k3s/Kubernetes is not Factory core.

If a future product needs a Kubernetes architecture lab, keep that as a separate optional scenario/profile rather than the normal execution path.

---

# 11. Docker still has a place

Docker is not the ETL orchestrator.

Docker Compose is an optional service-lifecycle adapter for scenarios that genuinely need local services such as FastAPI + Redis.

Pure DuckDB/dbt/dlt/Polars projects should run with no Docker.

---

# 12. dlt and dbt responsibilities

dlt = ingestion:

~~~text
extract -> normalize -> load
~~~

dbt = SQL model transformation/testing:

~~~text
sources -> staging -> intermediate -> marts/tests
~~~

Factory Local Orchestrator = end-to-end control flow:

~~~text
source -> dlt -> dbt -> Python/ML -> export
~~~

---

# 13. Apache Hop

Hop remains interesting as an optional visual ETL pipeline activity/import format because it provides a genuine local visual data-transformation engine.

Factory does not need Hop to get an ADF-like project canvas because Mosaic already contains that canvas and control-flow vocabulary.

If added later, treat a Hop pipeline as an encapsulated activity/subgraph rather than a second global orchestration authority.

---

# 14. UI strategy

Do not adopt an external orchestrator UI as Factory's main frontend.

Reuse what we already have:

- Mosaic SharedGraphCanvas / FactoryPipelines / PipelineSurface for the global DAG;
- DataPass Hop for source-file milestones and line-synchronized understanding;
- dbt artifacts for the nested dbt DAG;
- Factory run receipts for live status and timing.

Example:

~~~text
[Generate] ✓
    ↓
[dlt ingest] ✓ 1.2s
    ↓
[dbt group] running
    ├─ stg_orders ✓
    ├─ fct_orders running
    └─ tests waiting
~~~

This is closer to our target than embedding Airflow or Dagster UI.

---

# 15. Relationship to Fabric

Factory should intentionally feel similar to Fabric Data Factory at the orchestration level:

- activity nodes;
- dependencies;
- success/failure paths;
- parameters;
- retries;
- control flow;
- run history;
- selected-node settings.

Local mappings:

~~~text
Fabric Copy/Data movement -> dlt / DuckDB / file adapter
Fabric SQL activity       -> DuckDB SQL
Fabric dbt activity       -> dbt Core
Fabric Notebook/Python    -> trusted Python / Polars / Pandas
ML step                   -> sklearn
Web/API                    -> FastAPI/request activity
~~~

Factory is therefore a genuine local data-factory workbench, not a wrapper around a generic scheduler.

---

# 16. Local minimalism

Preferred order:

1. local files + Parquet;
2. DuckDB;
3. DuckLake when lakehouse semantics matter;
4. dbt-duckdb;
5. dlt;
6. Polars/Pandas;
7. sklearn;
8. Factory Local Orchestrator;
9. FastAPI only if API semantics matter;
10. Redis only if queue/cache/stream/state matters;
11. Docker Compose only if service processes matter.

No default:

- MotherDuck;
- MinIO;
- S3 emulator;
- Kafka;
- Airflow;
- Dagster;
- Meltano;
- Kubernetes/k3s;
- Spark;
- fake Spark;
- Postgres metadata database.

---

# 17. V1 acceptance criteria

1. Factory Semantic DAG validates independently from any third-party orchestrator.
2. Mosaic-derived graph renders supported activities.
3. Real local execution uses our bounded scheduler.
4. Independent nodes can execute concurrently.
5. Success/failure dependency conditions work.
6. Retries, timeout and cancellation work.
7. dbt is an expandable nested DAG.
8. dlt is a real ingestion node.
9. DuckDB/DuckLake/dbt/Polars/Pandas/sklearn run without Docker.
10. Run receipts are local and labelled basis=observed, environment=local.
11. DataPass Hop can explain a selected source file beside the DAG.
12. No fake Spark.
13. No Kubernetes.
14. No mandatory Docker.
15. No remote service.
16. No third-party orchestration daemon is required.

This is the revised Factory V1 orchestration decision.