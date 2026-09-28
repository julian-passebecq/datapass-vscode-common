# DataPass Factory V1 — execution architecture gate for Codex

Date: 2026-09-28  
Consolidation authoring context: GPT-5.6 Sol in ChatGPT, not GPT-6.

## Status

**OPEN ARCHITECTURE GATE — Codex Tech Lead must decide after spikes.**

This file supersedes both earlier provisional conclusions:

1. "Dagster is the mandatory V1 orchestrator."
2. "Build our own bounded orchestrator as the final V1 decision."

Both were reasonable intermediate ideas, but the full repository audit uncovered a stronger prior candidate: **Duckle**, plus substantial existing Mosaic/FactoryLab control-flow code.

No implementation agent should greenfield a generic ETL/orchestration layer until this gate is resolved.

---

# 1. Stable requirements independent of implementation

Factory V1 is:

- 100% local;
- no hosting product;
- no managed cloud dependency;
- no MotherDuck requirement;
- no Kubernetes/k3s core dependency;
- no fake Spark;
- no Spark simulator;
- no MinIO/S3 emulation by default;
- no mandatory Docker;
- DuckDB/local Parquet first;
- DuckLake when lakehouse semantics are useful;
- dbt Core/dbt-duckdb;
- dlt;
- Polars/Pandas;
- scikit-learn;
- optional FastAPI/Redis/Docker Compose only when a scenario needs those semantics.

The Factory global DAG sits **above dbt**.

dbt remains an expandable nested model/transformation DAG.

The UI must show a Fabric/Data Factory-like local pipeline:
- activities;
- dependencies;
- success/failure paths;
- parameters;
- retries;
- run state;
- logs/metrics;
- nested dbt graph;
- selected-node details.

Common Semantic IR remains authoritative above any runtime.

---

# 2. Fabric mental model

Fabric Data Factory is the conceptual UX reference:

~~~text
pipeline
  -> activity
  -> activity
  -> branch/control flow
  -> downstream activity
~~~

Activities may represent:
- data movement;
- SQL;
- notebooks/Python;
- dbt;
- dataflow;
- conditions/loops;
- web/API operations.

Factory maps that locally:

~~~text
Factory pipeline
  -> dlt
  -> DuckDB/DuckLake
  -> dbt
  -> Polars/Pandas/Python
  -> sklearn
  -> quality/export
~~~

The implementation may use Duckle, extracted Mosaic control flow, or a hybrid, but the semantic contract should not change.

---

# 3. Candidate A — Duckle adapter-first

External repository:

`slothflowlabs/duckle`

Historical DataPass decision:

`julian-passebecq/datapasscontrol/control/decisions/duckle-factorylab.json`

That decision explicitly says:

> treat Duckle as the primary lower-layer candidate and stop greenfield generic executor work until an adapter/fork spike is complete.

Verified/relevant capabilities from current Duckle source/docs:

- dual license MIT OR Apache-2.0;
- local/self-hosted;
- React / @xyflow visual canvas;
- Git-friendly plain pipeline JSON;
- DuckDB execution;
- DuckLake I/O;
- many sources/transforms/sinks/validators/control/code nodes;
- generated SQL/plan;
- per-node status;
- row counts;
- timings;
- previews;
- column lineage;
- run history;
- schedules/triggers;
- child pipelines/control flow;
- dbt execution;
- headless runner/server;
- MCP;
- single-machine/local-first positioning.

This overlaps heavily with the lower execution layer we were considering building.

## Required Duckle spike

Before rejecting or adopting it, Codex must prove:

1. translate a minimal Factory semantic pipeline to Duckle JSON;
2. execute CSV/Parquet -> select/filter/join -> DuckDB/DuckLake;
3. obtain generated SQL/plan;
4. obtain node preview;
5. obtain node rows/timings/status/errors;
6. obtain lineage or identify exact API gap;
7. exercise one control node;
8. exercise DuckLake source/sink behavior;
9. invoke/coordinate dbt and inspect artifacts;
10. measure startup/package/process overhead;
11. inspect runner/server security boundary;
12. list Fabric-like semantics that require our own adapter/control layer;
13. confirm stable machine-readable integration surface rather than UI scraping.

Preferred adoption strategy if successful:

~~~text
Factory UI / Semantic DAG
        ↓
Factory-to-Duckle adapter
        ↓
Duckle JSON / runner
        ↓
DuckDB / DuckLake
~~~

Default is adapter-first.

Do not fork Duckle merely for branding/layout changes.

Fork only if a measured required capability is inaccessible through supported boundaries.

---

# 4. Candidate B — extract/evolve Mosaic / FactoryLab control flow

Existing donor repositories contain substantial work.

Important paths already identified:

- `datapass-mosaic-vscode/runtime/factorylab/engine.py`;
- `runtime/datapass_runtime/factory_workspace.py`;
- `SharedGraphCanvas`;
- `FactoryPipelines.tsx`;
- `PipelineSurface.tsx`;
- `ducklabms_code/apps/web/src/foundation/GraphCanvas.tsx`;
- `WorkbenchSurface.tsx`;
- Fabric migration-source pipeline UI;
- `fastapi-fabric` semantic/control backend.

Existing control-flow concepts include:

- dependencies;
- success/failure/completed conditions;
- retries;
- retry delay;
- timeout;
- skipped/inactive;
- If;
- Switch;
- ForEach;
- Until;
- child pipelines;
- parameters;
- variables;
- run IDs;
- activity state;
- result propagation;
- local Workspace adapter boundary.

## Required extraction spike

1. isolate control-flow core from learning/simulation assumptions;
2. execute one real DuckDB activity;
3. execute a dbt group;
4. execute dlt;
5. execute trusted Python/Polars;
6. prove timeout/cancel/retry;
7. prove independent-node concurrency;
8. persist normalized local run receipt;
9. estimate lines/modules to own long-term;
10. list connector/dataflow functionality that would still need to be built.

This option gives maximum semantic/UI control but may duplicate a large amount of Duckle functionality.

---

# 5. Candidate C — hybrid

A potentially strong architecture is:

~~~text
Factory Semantic DAG + Fabric-like control flow
        ↓
our small control/orchestration layer
        ↓
Duckle for dataflow/data movement/preview/lineage
        +
direct adapters for dbt / Python / ML when appropriate
        ↓
DuckDB / DuckLake
~~~

Benefits:
- preserve our activity vocabulary/UX;
- reuse Duckle's expensive ETL/dataflow/connectors layer;
- avoid forcing every operation through Duckle;
- retain Common Semantic IR authority.

Risks:
- two execution paths;
- duplicate run-state normalization;
- more adapter complexity.

Codex must measure this rather than assume it is optimal.

---

# 6. Dagster / Airflow / Meltano / Hop

## Dagster

Good local orchestrator and useful catalog alternative.

Not assumed Factory core because:
- second graph/object model;
- additional daemon/web UI/project layer;
- mapping and event normalization overhead;
- Factory scope is intentionally local and bounded.

Keep as future adapter/export possibility.

## Airflow

Important real-world orchestrator.

Use for:
- DataPass analysis/import;
- client projects already using Airflow;
- catalog/architecture comparison.

Not Factory V1 core.

## Meltano

Useful ELT/plugin ecosystem.

Keep as:
- catalog alternative;
- import/project adapter.

Not core unless a concrete integration gap proves otherwise.

## Apache Hop

Strong visual ETL engine/UI.

Possible future:
- import .hpl/.hwf;
- encapsulated Factory activity;
- optional visual ETL module.

Do not create a second global scheduler authority.

---

# 7. Kubernetes / k3s / Docker

Kubernetes/k3s orchestrate services/containers, not the semantic ETL dependency graph.

They are not Factory core.

Docker Compose is optional service lifecycle for scenarios requiring FastAPI/Redis/etc.

Pure DuckDB/dbt/dlt/Polars projects should run without Docker.

---

# 8. Frontend strategy

Do not greenfield a graph/editor framework.

Audit/reuse:

- Mosaic SharedGraphCanvas;
- FactoryPipelines;
- PipelineSurface;
- ducklabms GraphCanvas/WorkbenchSurface;
- DataPass Hop code/understanding split;
- dbt artifacts for nested dbt DAG.

Factory UI should bind run state to the same Semantic DAG:

~~~text
[Generate] ✓
    ↓
[dlt] ✓
    ↓
[dbt] running
   ├─ staging ✓
   ├─ mart running
   └─ tests waiting
    ↓
[ML] waiting
~~~

External runtime UIs are optional diagnostics, not the primary Factory product UI.

---

# 9. ADR decision criteria

Codex must score evidence qualitatively across:

- amount of reusable proven code;
- local startup simplicity;
- package/install footprint;
- Windows compatibility;
- process isolation;
- cancellation/timeout;
- real ETL coverage;
- DuckDB/DuckLake fit;
- dbt fit;
- dlt fit;
- Python/Polars/sklearn fit;
- control flow;
- preview;
- lineage;
- run receipts;
- UI integration;
- security;
- license/attribution;
- maintenance burden;
- vendor/project churn risk;
- testability;
- ability to preserve Common Semantic IR authority.

No hidden numeric winner is required.

---

# 10. Gate output required from Codex

Codex must produce:

`ADR-FACTORY-EXECUTION-001.md`

with:

- current evidence;
- Duckle spike result;
- Mosaic extraction spike result;
- hybrid analysis;
- chosen architecture;
- interface diagram;
- process diagram;
- error/cancel model;
- security boundary;
- version pinning strategy;
- licensing notes;
- migration path;
- revisit conditions.

Until this ADR exists, implementation agents may work on Common Semantic IR, catalog, analyzers and UI extraction, but must not commit Factory to a generic executor architecture.
