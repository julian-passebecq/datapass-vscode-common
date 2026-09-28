# DataPass Factory — V1 product and implementation specification

Factory is the local executable-prototype product in the DataPass ecosystem.

Its purpose is:

> Take a data/cloud architecture idea, materialize a useful local version quickly, run it for real on local engines, inspect/edit it end to end, collect real local metrics, and demonstrate it without requiring a cloud account.

Factory is not a cloud emulator and is not a replacement for Fabric/Databricks/Azure.

---

# 1. Product positioning

Factory answers:

> Can we build and show a credible local version of this architecture quickly?

This is different from:

- DataPass V4: understand the real Git repo;
- Prototype Cloud: compare possible cloud architectures;
- Hub: route to the right tool/extension;
- Mosaic: teach/learn through labs and exercises;
- Cloudiagram: later publication/PPTX.

Factory is the executable local proof layer.

---

# 2. V1 principles

Factory V1 must optimize:

- time-to-first-demo;
- low setup cost;
- deterministic/reproducible local runs;
- inspectability;
- native files in Git;
- editability;
- understandable data flow;
- measurable outputs;
- optional services, not mandatory infrastructure;
- no cloud credentials;
- no fake distributed runtime.

Factory V1 must not optimize for:

- distributed production scale;
- Kubernetes parity;
- Spark cluster fidelity;
- enterprise orchestration parity;
- cloud service feature completeness.

---

# 3. Explicit final non-goals

These are deliberate decisions.

## No fake Spark

Factory V1 has:

- no fake Spark DataFrame API;
- no simulated Spark stages;
- no fake shuffle metrics;
- no virtual Spark cluster profile;
- no Spark-equivalent execution engine.

The existing Mosaic SparkLab stays in the separate Mosaic learning product.

If a future Factory version supports Spark, use:

- an explicit real Spark adapter;
- or imported/captured Spark plans and telemetry;
- or route the user to Mosaic for educational simulation.

Do not port SparkLab into Factory.

## No Kubernetes in V1

No required:

- kind;
- minikube;
- k3d;
- local cluster;
- Helm stack.

Docker Compose is sufficient for optional local services.

## Dagster is the reference orchestrator; Airflow is not V1 core

Factory owns a small provider-neutral **semantic DAG contract**, but **Dagster OSS is the reference orchestration/runtime adapter for V1**.

The global Factory DAG sits above dbt:

~~~text
Factory / Dagster DAG
  generator -> dlt -> dbt -> Polars -> sklearn -> publish
                         |
                         +-- dbt internal model DAG
~~~

Airflow can later be:

- an import/export adapter;
- a runtime adapter for projects that genuinely use Airflow;
- an external official tool route.

Do not port Mosaic's Airflow simulator into Factory.

## No mandatory Docker

A simple project using DuckDB/dbt/Polars must run without Docker.

## No MinIO or local object-store infrastructure by default

Prefer:

- local filesystem;
- Parquet;
- DuckDB;
- DuckLake when lakehouse semantics are useful.

Do not add MinIO/S3 emulation merely to make the prototype look cloud-like. Add an object-store profile only if a concrete scenario needs object-store behavior.

---

# 4. Core V1 technology stack

## Base

- Python 3.11+;
- Node/VS Code extension host;
- Parquet;
- JSON/CSV where useful.

## Analytical SQL

- DuckDB.

## Lakehouse-like local storage

- DuckLake, optional.

## Ingestion

- dlt.

## Global orchestration / visible DAG

- Dagster OSS as the V1 reference orchestration adapter;
- Factory semantic DAG remains provider-neutral and serializable;
- Dagster UI is optional deep operational tooling, not the only DAG view.

## SQL transformation

- dbt Core + dbt-duckdb.

## DataFrames

- Polars;
- Pandas.

## ML

- scikit-learn.

## Service boundary

- FastAPI, optional.

## Cache/queue/stream/state

- Redis, optional.

## Service packaging

- Docker Compose, optional.

## Charts

- reuse simple local charts and/or integrate dbt Charts as an optional external surface.

## Sharing

- MotherDuck optional later; not required for V1.

## Local-minimalism rule

Before adding a runtime/service, ask whether an existing local primitive already solves the need.

Preferred order:

1. DuckDB.
2. Local Parquet.
3. DuckLake only when lakehouse table semantics matter.
4. Polars/Pandas.
5. dbt-duckdb.
6. dlt.
7. scikit-learn.
8. Dagster for the global DAG.
9. FastAPI only for a real API/service boundary.
10. Redis only for a real cache/queue/stream/state boundary.
11. Docker Compose only when service processes require it.
12. MotherDuck only as an optional sharing/remote target.

Do not add Postgres, MinIO, Kafka, Kubernetes or another database/service simply to imitate cloud infrastructure.

---

# 5. Why these components

## DuckDB

Factory’s default local analytical engine.

Use for:

- SQL transformations;
- warehouse-like prototype;
- Parquet queries;
- profiling;
- query plans;
- dbt target;
- small/medium local analytical workflows.

## DuckLake

Use only when the prototype needs a lakehouse-style table/storage separation.

Good use cases:

- Bronze/Silver/Gold stored as Parquet;
- snapshots/time-travel demonstrations;
- lakehouse project story;
- multiple local clients against the same logical tables.

Do not force DuckLake on every project.

## Polars

Use for:

- fast in-process dataframe work;
- feature engineering;
- transformation logic that is cleaner in Python/dataframes;
- ML preprocessing.

## Pandas

Keep first-class because:

- client projects often use it;
- familiar prototype path;
- useful comparison option;
- useful for small data and existing code.

Do not force-convert Pandas to Polars automatically.

## scikit-learn

Use for:

- classification;
- regression;
- clustering;
- anomaly detection;
- feature preprocessing;
- model evaluation.

It is sufficient for compelling local ML prototypes.

## dlt

Use for:

- API/JSON/database/file ingestion patterns;
- extract/normalize/load;
- local schema/state/load handling;
- quick pipeline construction into DuckDB-like destinations.

dlt is not the whole orchestrator.

## FastAPI

Use only where an API/service boundary is meaningful.

Examples:

- synthetic business API;
- IoT telemetry API;
- scoring endpoint;
- webhook receiver;
- backend service;
- data generator service.

## Redis

Optional for:

- stream/event buffer;
- queue;
- cache;
- temporary shared state;
- job status.

Do not launch Redis for a project that does not need it.

## Docker Compose

Use to launch service components such as FastAPI + Redis.

It is an optional adapter.

---

# 6. Product shell: VS Code first

Current direction should use a VS Code extension because:

- native source files remain in the editor;
- Git stays visible;
- split panes already exist;
- DataPass ecosystem already uses VS Code;
- Mosaic provides reusable webview/workbench patterns;
- Hub can route between extensions.

Recommended new repository:

**julian-passebecq/datapass-vscode-factory**

Do not create the repo until Claude has checked user approval/current repository conventions.

---

# 7. Extract workbench-core from Mosaic

Mosaic currently has the most mature reusable local-workbench UX.

Relevant existing pieces include:

- src/webview/MosaicSurface.tsx;
- src/platform/runtimeClient.ts;
- mosaic layout/state helpers;
- result-table components;
- local catalog exploration;
- code/data/chart panes;
- query history;
- runtime state;
- DuckDB/DuckLake-backed preview;
- safe loopback runtime protocol.

The reusable extraction target should be a common package, conceptually:

~~~text
@datapass/workbench-core
~~~

It should contain:

- grid/split layout;
- pane registration;
- editor/source references;
- code pane;
- SQL pane;
- notebook/text pane;
- data table preview;
- chart pane;
- graph/DAG pane;
- lineage mini-map pane;
- run status component;
- runtime protocol types;
- layout persistence primitives.

It must not contain:

- Practice;
- Interview;
- learner grading;
- exercise packs;
- SparkLab;
- Airflow simulator;
- simulated Fabric;
- simulated Databricks;
- course progress.

Mosaic remains a consumer after extraction.

---

# 8. Do not block Factory on a total Mosaic refactor

Safe migration sequence:

1. Define workbench-core API.
2. Move one or two truly generic components.
3. Add compatibility wrappers in Mosaic.
4. Build Factory against those extracted components.
5. Continue extraction only when duplication is proven.

Do not attempt a multi-week “clean architecture rewrite” before Factory can show anything.

---

# 9. Local runtime security model

Mosaic already uses a useful pattern:

- Python FastAPI runtime on loopback;
- per-launch random token;
- token passed in environment/header;
- token never persisted/logged;
- bounded API;
- no arbitrary remote listener.

Factory can reuse/adapt this pattern.

Requirements:

- bind loopback only;
- fresh token each launch;
- explicit workspace root;
- all file paths resolved under allowed workspace directories;
- no arbitrary path reads;
- no arbitrary shell command endpoint;
- no arbitrary Python code execution endpoint hidden behind metadata;
- trusted-local-code execution is explicit;
- scrub error output where needed;
- run adapters are typed/allowlisted.

---

# 10. Factory project structure

Illustrative structure:

~~~text
factory-project/
  factory.yml

  orchestration/
    definitions.py

  generators/
    scenario.yml
    generate.py

  ingest/
    customers.py
    telemetry.py

  dbt/
    dbt_project.yml
    profiles.yml
    models/
      staging/
      intermediate/
      marts/

  transforms/
    polars/
    pandas/
    sql/

  ml/
    features.py
    train.py
    score.py

  services/
    api/
      app.py
      Dockerfile

  quality/
    checks.yml

  charts/
    ...

  data/
    raw/
    ducklake/
    outputs/

  .factory/
    local-state/
~~~

Exact paths may evolve. The principle is native source first and generated state separated.

---

# 11. Factory semantic DAG contract

Factory owns a typed local **semantic project graph**. This is the portable source used by the Factory UI, Common Semantic Engine and Prototype Cloud handoff.

Dagster is the first execution/orchestration adapter for this graph; Dagster Python definitions are not the provider-neutral source of truth.

The graph is explicitly above dbt. A dbt node/group can expose its own internal model DAG from dbt artifacts.

Conceptual YAML:

~~~yaml
version: 1

project:
  id: wind-demo
  title: Wind Turbine Local Factory

steps:
  - id: generate_weather
    uses: generator
    config:
      scenario: wind
      entity: weather

  - id: telemetry_api
    uses: service
    depends_on: [generate_weather]
    config:
      adapter: fastapi
      service: services/api

  - id: stream
    uses: service
    depends_on: [telemetry_api]
    config:
      adapter: redis
      mode: stream

  - id: ingest
    uses: dlt
    depends_on: [stream]
    config:
      file: ingest/telemetry.py

  - id: silver
    uses: dbt
    depends_on: [ingest]
    config:
      selector: silver+

  - id: features
    uses: polars
    depends_on: [silver]
    config:
      file: ml/features.py

  - id: anomaly_model
    uses: sklearn
    depends_on: [features]
    config:
      file: ml/train.py
~~~

The schema must be strict.

Unknown step type rejected.

---

# 12. Execution adapters

Factory execution is adapter-based.

Conceptual interface:

~~~ts
interface ExecutionAdapter {
  id: string;
  canRun(step: LocalExecutionStep): boolean;
  prepare(ctx: RunContext): Promise<PreparedStep>;
  run(step: PreparedStep, signal: AbortSignal): Promise<LocalRunReceipt>;
  cleanup?(step: PreparedStep): Promise<void>;
}
~~~

V1 adapter list:

- generator;
- dlt;
- duckdb-sql;
- dbt;
- python;
- polars;
- pandas;
- sklearn;
- quality;
- fastapi-service;
- redis-service;
- docker-compose.

No spark-sim adapter.

---

# 13. Dagster orchestration adapter responsibilities

Do not build a second scheduler if Dagster already provides the runtime semantics Factory needs.

Factory owns:

- strict DAG schema;
- stable semantic IDs;
- dependency validation;
- cycle refusal before execution;
- nested groups/subgraphs;
- source/evidence references;
- mapping to Common Semantic IR;
- normalized run receipts;
- its own DAG visualization.

Dagster owns:

- dependency execution;
- local run state;
- schedules when a scenario actually needs them;
- retries when explicitly configured;
- resource lifecycle;
- operational logs/events.

Factory must normalize Dagster events into its own receipts so the Common Engine does not depend on Dagster types.

The Factory UI must remain useful without opening Dagster's own UI. Provide an optional **Open in Dagster** route for deeper local operational inspection.

Do not implement:

- Airflow scheduler semantics;
- Kubernetes operators;
- a distributed executor abstraction;
- a second home-grown production scheduler.

---

# 14. Run truth model

Each run step returns real local observations.

Example:

~~~json
{
  "stepId": "silver",
  "status": "succeeded",
  "durationMs": 2840,
  "metrics": [
    {
      "name": "output_rows",
      "value": 1180042,
      "basis": "observed",
      "environment": "local"
    }
  ]
}
~~~

Do not transform this into “Fabric would take 2.84 seconds”.

Prototype Cloud/performance model may use local observations as one weak input only when explicitly modeled.

---

# 15. UI target

Factory UI should combine:

- project tree;
- native editor;
- understanding pane;
- mini flow/data map;
- run controls;
- local catalog;
- metrics.

Target arrangement:

~~~text
+---------------------------------------------------------------+
| FACTORY                                  Run project           |
+----------------+--------------------------+-------------------+
| PROJECT        | EDITOR                   | UNDERSTANDING     |
|                |                          |                   |
| ingest         | telemetry.py             | Role: ingestion   |
| bronze         |                          | Inputs/outputs     |
| silver         | code...                  | Operations        |
| gold           |                          | Findings          |
| ml             |                          | Run metrics       |
+----------------+--------------------------+-------------------+
| FLOW / DATA MAP                                             |
| API -> raw -> silver -> gold -> features -> model           |
+-------------------------------------------------------------+
~~~

This uses common semantic-engine output, not a Factory-specific explanation schema.

---

# 16. Pane behavior

When a file is selected:

left/center:

- native VS Code source.

right:

- artifact kind;
- role;
- milestones;
- inputs/outputs;
- operations;
- findings;
- local run state;
- unresolved questions.

bottom:

- mini lineage / data model / DAG.

When a milestone is selected:

- editor reveals/highlights source range;
- right pane shows only that milestone’s operations/data;
- bottom map scopes to its local subgraph.

This reuses the original “code beside explanation” idea from DataPass Hop.

---

# 17. Local catalog explorer

Factory should expose:

- schemas;
- tables/views;
- row counts;
- columns/types;
- local layer;
- sample rows;
- source producer;
- last run;
- local metrics.

DuckDB/DuckLake catalog is the primary V1 implementation.

Useful actions:

- preview rows;
- open generated SQL;
- open producer file;
- show lineage;
- show dbt model/test status.

---

# 18. Generator framework

Contoso’s GeneratorService already demonstrates useful properties:

- deterministic scenario;
- seed;
- scale;
- run id;
- manifest;
- generator version/hash;
- per-file hashes;
- row counts;
- reproducibility;
- comparison between runs;
- bounded workspace paths.

Generalize this into Factory.

Conceptual interface:

~~~python
class ScenarioGenerator:
    id: str
    version: str

    def generate(
        scenario: str,
        scale: int,
        seed: int,
        output: Path,
    ) -> GenerationReceipt:
        ...
~~~

Receipt should include:

- generator id/version/hash;
- scenario;
- seed;
- scale;
- files;
- hashes;
- row counts;
- created time;
- semantic dataset ids.

---

# 19. Scenario packs

Initial packs:

## Retail / Contoso

Extract/generalize existing scenarios:

- retail baseline;
- online migration;
- margin pressure;
- logistics delays;
- currency exposure.

Entities:

- customer;
- product;
- store;
- exchange rates;
- sales.

## Wind

Entities:

- turbine;
- site;
- telemetry;
- weather;
- maintenance;
- alerts;
- predictions.

Scenarios:

- nominal;
- high wind;
- sensor drift;
- bearing vibration anomaly;
- temperature anomaly;
- maintenance backlog;
- power underperformance.

Future:

- finance;
- manufacturing;
- logistics.

---

# 20. Wind demo V1

Recommended demonstration pipeline:

~~~text
deterministic generators
   |
   +-- weather
   +-- turbines
   +-- maintenance
   +-- telemetry
          |
          v
       FastAPI      optional service mode
          |
          v
       Redis        optional streaming mode
          |
          v
         dlt
          |
          v
     DuckLake Bronze
          |
          v
       dbt Silver
          |
     +----+---------+
     |              |
     v              v
  dbt Gold        Polars features
                    |
                    v
                 sklearn
                    |
                    v
               predictions
     |              |
     +------v-------+
          dashboard
~~~

A reduced no-services mode should also work:

~~~text
generator -> Parquet -> dlt/dbt -> DuckDB/DuckLake -> Polars/sklearn
~~~

This keeps Docker optional.

---

# 21. Retail/Contoso demo V1

Factory should reproduce and improve the existing Contoso flow:

~~~text
Generate
 -> Inspect
 -> Bronze
 -> dbt Silver
 -> dbt Gold
 -> SQL/KPI
 -> Charts
~~~

Enhancements:

- run DAG;
- semantic understanding pane;
- data lineage;
- scenario comparison;
- local metrics;
- ML optional scenario;
- exportable run snapshot.

Contoso Data Studio remains the source reference during migration.

---

# 22. dlt integration

Use dlt for ingestion where appropriate.

Factory adapter responsibilities:

- invoke a declared ingestion module;
- pin workspace/destination;
- collect load receipt;
- capture produced datasets;
- expose schema/load state;
- sanitize errors;
- register evidence.

Do not treat dlt’s internal extract/normalize/load as the entire project orchestration.

---

# 23. dbt integration

dbt is important in Factory.

Support:

- dbt-duckdb;
- run/build/test;
- selectors;
- manifest/run_results ingestion;
- model lineage;
- tests;
- materializations;
- source/model status.

Prefer real dbt Core for Factory, not the Mosaic teaching emulation.

Mosaic’s dbt emulation remains a learning feature.

dbt Charts optional:

- detect dct;
- validate project/boards;
- offer open/serve route;
- do not reimplement the whole official UI.

## dbt as a nested DAG

The global Factory/Dagster graph must sit **above** dbt.

At global level:

~~~text
ingest -> dbt group -> ML/features -> publish
~~~

Inside the dbt group, ingest `manifest.json` / `run_results.json` and render:

~~~text
sources -> staging -> intermediate -> marts -> tests
~~~

Clicking a dbt group should expand to models; clicking a model should open its SQL and semantic understanding.

Dagster's dbt integration may execute/materialize the dbt assets, but Factory/Common IR keeps provider-neutral IDs and evidence.

---

# 24. DuckLake integration

Contoso’s current DuckLake service already demonstrates:

- local SQLite-backed metadata;
- managed data path;
- schemas;
- load Parquet;
- snapshots;
- read-only inspection;
- snapshot preview/compare.

Generalize:

- project-specific catalog name;
- arbitrary layers;
- bounded attach paths;
- explicit extension loading;
- no uncontrolled file access;
- receipts linking runs to tables/snapshots.

DuckLake is optional.

DuckDB-only project must stay simpler.

---

# 25. SQL security

Factory SQL editor/query actions should distinguish:

- read-only exploratory query;
- declared transformation run.

Read-only UI query:

- SELECT/EXPLAIN only;
- external file/network functions blocked unless explicitly managed;
- bounded result rows.

Transformation adapter:

- runs known project SQL/dbt assets;
- explicit user run;
- isolated local workspace.

Do not make arbitrary pasted SQL a silent write path.

---

# 26. Python execution policy

Factory differs from DataPass discovery: Factory is explicitly an execution product.

Still:

- user must explicitly run;
- only within selected local project/workspace;
- show which file will run;
- no invisible background execution;
- environment is local/trusted;
- record exact file hash/commit when feasible;
- capture output/logs;
- cancellation support.

Reusable Mosaic trusted-Python UX may be useful.

---

# 27. FastAPI component

Factory FastAPI adapter should support:

- known project service folder;
- loopback bind by default;
- health probe;
- start/stop;
- logs;
- port allocation;
- service URL;
- optional Docker mode.

It should not expose arbitrary public interfaces by default.

---

# 28. Redis component

Factory Redis adapter should support:

- local Docker container by default;
- known port;
- optional persistence;
- health check;
- stream/list/cache examples;
- explicit cleanup.

Do not build a full Redis admin UI in V1.

---

# 29. Docker Compose adapter

Use only for service scenarios.

Responsibilities:

- validate compose file location under project;
- explicit user start/stop;
- show services;
- health/status;
- logs;
- no arbitrary compose path outside project;
- no hidden privileged mode;
- do not auto-pull unknown images without user visibility.

For a generated project, templates should use pinned major/minor images where practical.

---

# 30. No Kubernetes V1

If later requested, V2/V3 may support:

- generate manifests;
- validate manifests;
- optional kind/k3d adapter.

Do not add cluster setup to the first acceptance gate.

---

# 31. Local runtime variants

Factory should allow implementation profiles.

Example logical task:

~~~text
customer enrichment
~~~

Possible local implementations:

- DuckDB SQL;
- Polars;
- Pandas.

A profile can select one.

Factory may compare local observations:

~~~text
DuckDB
  0.31 s
  observed local

Polars
  0.43 s
  observed local

Pandas
  1.82 s
  observed local
~~~

Do not claim the fastest local adapter is automatically the best cloud architecture.

---

# 32. Comparison dimensions

Local variant comparison can include:

- setup steps;
- dependencies;
- number of files;
- service/container count;
- local runtime;
- local memory if safely measurable;
- rows processed;
- output files/tables;
- tests;
- developer complexity;
- operational components;
- reproducibility.

Cloud estimates come from Prototype Cloud/catalog, not directly from these numbers.

---

# 33. ML workflow

Factory ML V1 should be simple and transparent.

Example:

~~~text
Gold customer table
  -> Polars feature transform
  -> train/test split
  -> sklearn model
  -> metrics
  -> prediction table
~~~

Capture:

- algorithm;
- feature list;
- input dataset;
- train/test rows;
- random seed;
- metrics;
- model artifact path/hash;
- prediction output.

No need for full MLOps platform in V1.

Optional later:

- MLflow adapter;
- ONNX export;
- batch scoring service.

---

# 34. Dashboard/visual output

V1 can use:

- simple chart panels;
- dbt Charts integration;
- table/KPI panels.

Do not spend initial effort building a Power BI clone.

The investor/demo value is the full end-to-end pipeline and inspectability.

---

# 35. Run snapshot contract

Factory should be able to emit a sanitized local snapshot for later consumers.

Conceptual:

~~~json
{
  "format": "datapass.factory.snapshot",
  "version": 1,
  "project": {},
  "scenario": {},
  "run": {},
  "steps": [],
  "datasets": [],
  "lineage": [],
  "metrics": [],
  "modelMetrics": [],
  "findings": []
}
~~~

No secrets.

This is not the canonical project source.

It is a run/read-model snapshot.

Future consumers:

- Prototype comparison;
- Cloudiagram publication;
- Mongoku portfolio;
- export/report.

---

# 36. Common semantic engine integration

Factory should not invent separate dataset/operation concepts.

Flow:

~~~text
Factory project files
  -> common semantic analyzers
  -> semantic graph
  -> Factory runner
  -> observed local receipts
  -> merge receipts back into semantic read model
~~~

Example:

Static analysis says:

~~~text
model stg_sales
  reads bronze.sales
  writes silver.stg_sales
~~~

Factory run adds:

~~~text
observed local
  output rows = ...
  duration = ...
~~~

Both coexist.

---

# 37. Prototype Cloud -> Factory mapping

Prototype Cloud scenario can offer:

~~~text
Build local prototype
~~~

The mapping layer chooses local analogues.

Example Fabric-oriented scenario:

~~~text
Fabric Lakehouse
  -> local DuckLake

Fabric Pipeline
  -> Factory DAG

Notebook transform
  -> Polars/Pandas/Python or dbt/SQL according to logic

Semantic model/report
  -> local Gold mart + simple charts
~~~

This is conceptual mapping, not service emulation.

---

# 38. Hub -> Factory integration

Hub can expose:

- “Open local prototype in Factory”;
- “Run current Factory project” if extension installed;
- “Install/show Factory extension”;
- “Open Factory docs”.

Hub does not execute the run itself.

---

# 39. Testing strategy

## Workbench-core

- layout persistence;
- pane registration;
- message validation;
- no learning-module dependency.

## Runtime API

- loopback token required;
- wrong/missing token refused;
- path traversal refused;
- workspace boundary tests;
- adapter schema validation.

## DAG

- cycle refusal;
- dependency ordering;
- failure propagation;
- cancellation;
- invalid adapter;
- duplicate IDs;
- missing dependencies.

## DuckDB/DuckLake

- reproducible project fixture;
- read/write policy;
- path restrictions;
- schema/table lifecycle;
- snapshots if DuckLake enabled.

## dlt/dbt/Polars/Pandas/sklearn

One synthetic fixture each.

## Services

- FastAPI health/start/stop;
- Redis stream happy path;
- Docker Compose optional CI lane where available.

## End-to-end

Contoso:

~~~text
generate -> bronze -> dbt build -> query -> charts
~~~

Wind:

~~~text
generate -> ingest -> transform -> ML -> dashboard
~~~

No cloud account.

---

# 40. Factory V1 acceptance criteria

A fresh machine with prerequisites can:

1. install/open Factory;
2. open the Contoso fixture;
3. generate deterministic data;
4. run the local DAG;
5. inspect Bronze/Silver/Gold tables;
6. open source files beside semantic explanation;
7. view lineage;
8. see real local run durations/row counts;
9. run dbt tests;
10. inspect charts/KPIs;
11. reset/reproduce the scenario.

Wind acceptance:

1. generate turbine/weather/maintenance data;
2. run no-service mode without Docker;
3. optionally enable FastAPI + Redis service mode with Docker Compose;
4. ingest/transform;
5. train/score simple sklearn model;
6. view anomaly/prediction output;
7. show actual local metrics.

At no point is fake Spark involved.

---

# 41. Suggested initial Factory milestones

## F0 — extraction design

- inventory Mosaic reusable components;
- inventory Contoso reusable services;
- define workbench-core API;
- define Factory runtime/DAG contract.

## F1 — minimal shell

- VS Code extension;
- project tree;
- workbench panes;
- loopback runtime;
- DuckDB;
- run one SQL step.

## F2 — local DAG

- strict factory.yml;
- DAG validation;
- run receipts;
- status UI.

## F3 — Contoso path

- generator extraction;
- DuckDB/DuckLake;
- dbt;
- charts/KPIs.

## F4 — dataframe/ML

- Polars;
- Pandas;
- sklearn;
- ML receipt/metrics.

## F5 — ingestion

- dlt adapter.

## F6 — services

- FastAPI;
- Redis;
- Docker Compose.

## F7 — Common Engine integration

- semantic artifact/role/milestone/lineage in panes;
- observed local metrics merged into read model.

## F8 — Wind demo

- scenario pack;
- end-to-end run;
- investor demo script.

---

# 42. What Claude must not do during Factory bootstrap

Do not:

- copy the full Mosaic repo and rename it;
- keep Mosaic learning code hidden inside Factory;
- port SparkLab;
- build Kubernetes support;
- install Airflow because “orchestration” sounds like Airflow;
- require Docker for DuckDB/dbt-only scenarios;
- invent a new semantic schema separate from Common Engine;
- make Contoso-specific names the core Factory model;
- make the runtime listen publicly;
- add arbitrary shell execution endpoints;
- claim local performance predicts Fabric/Databricks;
- rewrite dbt Charts instead of integrating it where practical.

Factory should remain small enough that a local prototype is genuinely faster than using the cloud.
