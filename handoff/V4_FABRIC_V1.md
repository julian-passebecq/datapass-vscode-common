# DataPass V4 — Fabric V1 catalog vertical

Fabric is the first concrete provider vertical for the new Catalog + Prototype Cloud + DataPass V4 architecture.

The objective is not to encode every Microsoft Fabric feature. The objective is to cover the decisions that repeatedly matter in real analytics/data-engineering projects and that DataPass can explain using repo/workload evidence.

---

# 1. Scope

Fabric V1 should understand/catalog enough to compare and explain projects using combinations of:

- Fabric Lakehouse;
- Fabric Warehouse;
- Data Factory/Pipeline-style orchestration;
- notebooks;
- SQL transformation;
- semantic model;
- Power BI consumption;
- Git/project artifacts where available;
- monitoring/capacity concerns;
- dbt as an optional development method.

Out of first vertical scope:

- every Fabric item type;
- real cloud deployment;
- provider account automation;
- complete API parity;
- automatic capacity-sizing guarantee;
- exact CU prediction;
- Power BI report-layout editing.

---

# 2. Base architecture families

Start with a small set of architecture families.

## fabric.lakehouse.medallion

Typical shape:

~~~text
sources
  -> ingestion/pipeline
  -> Lakehouse Bronze
  -> Silver transforms
  -> Gold tables
  -> Semantic Model
  -> Power BI
~~~

Good when:

- file/table lakehouse is central;
- transformations need dataframe/notebook or SQL flexibility;
- medallion structure is useful;
- Direct Lake may be relevant.

Concerns:

- transform engine choice;
- small-file/table maintenance;
- concurrency;
- notebook duration;
- semantic refresh/consumption mode;
- monitoring.

## fabric.warehouse.elt

Typical shape:

~~~text
sources
  -> ingestion
  -> Warehouse staging
  -> SQL transforms
  -> marts
  -> Semantic Model
  -> Power BI
~~~

Good when:

- SQL-first team;
- relational modeling;
- warehouse semantics;
- less notebook-centric transformation.

Concerns:

- SQL workload concurrency;
- materialization;
- refresh strategy;
- semantic consumption;
- data movement into warehouse.

## fabric.hybrid.lakehouse-warehouse

Typical shape:

~~~text
sources
 -> Lakehouse/raw
 -> transform
 -> curated tables
 -> Warehouse/marts or SQL serving layer
 -> Semantic Model
~~~

Good when:

- raw/semi-structured data benefits from lakehouse;
- final serving is strongly relational/SQL.

Concerns:

- duplicated layers;
- extra data movement;
- operational complexity;
- unclear ownership between layers.

## fabric.batch-bi.simple

A deliberately small pattern for prototypes/smaller teams.

~~~text
pipeline
 -> one curated store
 -> SQL transforms
 -> semantic model
~~~

Good when:

- volumes/complexity do not justify a large medallion hierarchy;
- team values fewer moving parts.

Do not force Bronze/Silver/Gold onto every Fabric project.

---

# 3. Important design decisions / feature options

Each architecture scenario should expose a subset of the following decisions.

---

# 3.1 Storage / serving: Lakehouse vs Warehouse

## Lakehouse-oriented

Potential advantages:

- file/table lakehouse workflow;
- natural notebook/dataframe integration;
- medallion organization;
- Direct Lake opportunities.

Trade-offs:

- data engineering concepts become more prominent;
- table/file maintenance matters;
- SQL-only developers may need more tooling/concepts.

Evidence needed:

- data types;
- volume;
- transformation style;
- notebook requirements;
- semantic consumption.

## Warehouse-oriented

Potential advantages:

- SQL-first workflow;
- relational model;
- familiar ELT/warehouse development.

Trade-offs:

- not every raw/semi-structured flow belongs there;
- heavy non-SQL transformation may be less natural.

Do not mark one as “better”.

---

# 3.2 Transformation engine: Notebook/Spark vs SQL vs mixed

Catalog decision:

~~~text
fabric.transform-engine
values:
  notebook
  sql
  mixed
~~~

For repo analysis, DataPass should infer current usage from:

- notebooks;
- pipeline activities;
- SQL models/scripts;
- dbt;
- Warehouse/Lakehouse artifacts.

Comparison dimensions:

- developer skill fit;
- testability;
- dependency visibility;
- incremental strategy;
- transformation complexity;
- large/wide data operations;
- operational monitoring;
- deployment/versioning.

Do not estimate Spark-specific runtime from source alone.

---

# 3.3 dbt: on/off

Catalog decision:

~~~text
fabric.dbt
values:
  off
  on
~~~

dbt OFF:

Possible advantages:

- fewer tools;
- less setup;
- simpler onboarding;
- fewer project files.

Possible trade-offs:

- more custom SQL/project conventions;
- tests/dependencies may be less standardized;
- more manual documentation discipline.

dbt ON:

Possible advantages:

- explicit model DAG;
- reusable tests;
- consistent SQL model structure;
- manifest/run-results evidence;
- easier static lineage for DataPass.

Possible trade-offs:

- extra toolchain;
- project/configuration;
- CI step;
- developer learning overhead.

Important:

Do not claim dbt automatically reduces CU or cloud cost.

Any cost/performance change must come from changed materialization, incremental strategy, avoided recompute, etc.

---

# 3.4 Full refresh vs incremental

Decision:

~~~text
fabric.refresh-strategy
values:
  full
  incremental
~~~

Inputs:

- total dataset bytes/rows;
- changed bytes/rows per period;
- refresh frequency;
- SLA;
- key/watermark support;
- late-arriving data;
- update/delete behavior.

Full refresh:

- simpler implementation;
- often easier recovery;
- potentially more compute/data scan.

Incremental:

- potentially less work per run;
- more state/watermark/change logic;
- harder recovery/backfill;
- correctness constraints.

Prototype Cloud should show:

- assumptions;
- relative processed-data range;
- developer/ops complexity;
- manual tasks;
- quality checks required.

---

# 3.5 Semantic consumption: Direct Lake vs Import

Decision:

~~~text
fabric.semantic-mode
values:
  direct-lake
  import
~~~

Catalog must describe:

- data location requirements;
- refresh behavior;
- model implications;
- performance/capacity considerations;
- compatibility constraints.

DataPass should detect current/declared semantic-model metadata when possible.

Do not claim Direct Lake is always cheaper/faster.

---

# 3.6 Data movement: Copy vs Shortcut

Decision:

~~~text
fabric.data-access
values:
  copy
  shortcut
  mixed
~~~

Copy:

- materializes data;
- more storage/data movement;
- stronger local ownership/isolation in some designs;
- scheduled movement/refresh.

Shortcut:

- avoids some copies;
- depends on external data/source arrangement;
- operational/governance behavior differs.

Catalog should include:

- provider prerequisites;
- freshness model;
- ownership;
- failure/dependency implications;
- data movement/cost driver.

---

# 3.7 Orchestration style

Decision:

~~~text
fabric.orchestration
values:
  pipeline
  notebook-chain
  mixed
~~~

Pipeline:

- visible control flow;
- retries/activities;
- operational routing;
- explicit dependencies.

Notebook chaining:

- fewer orchestration objects for simple flows;
- control logic may become code-centric;
- observability/maintenance implications.

Do not turn this into a stylistic preference.

Use project complexity and operational requirements.

---

# 3.8 Medallion depth

Decision:

~~~text
fabric.layering
values:
  simple
  bronze-silver
  bronze-silver-gold
  custom
~~~

Use evidence:

- number of source systems;
- transformation complexity;
- reuse;
- quality boundaries;
- serving requirements.

Avoid “medallion because cloud project”.

---

# 3.9 Materialized Gold vs views

Decision:

~~~text
fabric.gold-materialization
values:
  views
  tables
  mixed
~~~

Compare:

- recompute/query cost;
- storage;
- freshness;
- model consumption;
- complexity;
- data volume.

---

# 3.10 Capacity scheduling / refresh staggering

Decision:

~~~text
fabric.schedule-profile
values:
  concurrent
  staggered
  workload-aware
~~~

The catalog should explain why overlapping:

- pipelines;
- notebooks;
- Warehouse queries;
- semantic refreshes

can create capacity pressure.

Inputs:

- schedules;
- durations if observed/captured;
- concurrency;
- SLA.

Prototype Cloud can propose staggering as an option, but should not claim a precise CU saving without evidence/model.

---

# 3.11 Monitoring profile

Decision:

~~~text
fabric.monitoring-profile
values:
  minimum
  standard
  enhanced
~~~

Minimum:

- failed pipeline;
- failed notebook/job;
- semantic refresh failure;
- capacity/throttling signal where available.

Standard:

- above plus durations;
- data volumes;
- freshness;
- critical table tests;
- refresh overlap.

Enhanced:

- above plus detailed performance/capacity monitoring;
- critical join/transform metrics;
- anomaly thresholds;
- trend history.

This must be an explicit operational trade-off.

---

# 3.12 Data quality profile

Decision:

~~~text
fabric.quality-profile
values:
  embedded
  explicit-stage
  contract-driven
~~~

Embedded:

- checks inside transform models/notebooks.

Explicit stage:

- dedicated quality step before publish.

Contract-driven:

- stronger dataset rules/contracts/tests.

Compare:

- visibility;
- failure isolation;
- implementation effort;
- reuse;
- auditability.

---

# 3.13 Git/deployment complexity

Decision:

~~~text
fabric.delivery-profile
values:
  simple
  environment-structured
  governed
~~~

Simple:

- small team/prototype;
- minimal branches/environments.

Environment-structured:

- dev/test/prod or equivalent;
- explicit promotion.

Governed:

- approvals;
- stronger validation;
- controlled releases.

DataPass already has Git/work-order/readiness concepts and should surface them rather than Prototype inventing another lifecycle.

---

# 4. Workload inputs required for useful comparisons

Fabric scenario generation should consume a WorkloadProfile.

Important fields:

- dataset size;
- growth/day;
- refresh frequency;
- number of source systems;
- update/delete behavior;
- late-arriving data;
- number/size of joins;
- concurrency;
- latency target;
- semantic model size;
- report/consumer count;
- refresh schedules;
- training/scoring frequency if ML.

If values are missing:

- ask;
- infer with lower confidence only when evidence exists;
- do not invent.

---

# 5. Capacity / CU modeling

V1 should use a **driver model**, not an exact predictive model.

Potential drivers:

- data processed per run;
- run frequency;
- notebook duration;
- SQL query duration;
- concurrency;
- semantic refresh duration;
- overlapping workloads;
- model consumption/query activity;
- copy/movement work.

Output can be:

- low / medium / high pressure;
- estimated range;
- bottleneck windows;
- missing measurements;
- confidence.

Example:

~~~text
Capacity pressure: medium-high
Basis: estimated

Drivers:
  hourly Silver notebook
  semantic refresh starts at same minute
  3 concurrent pipelines

Missing:
  actual notebook duration
  actual semantic refresh duration

Candidate option:
  stagger refresh by 15 minutes

Expected effect:
  lower peak overlap

Not claimed:
  exact CU saved
~~~

---

# 6. Cost modeling

Keep costs separate from capacity.

Potential line items:

- Fabric capacity/subscription;
- storage;
- data movement;
- additional tooling where relevant;
- other provider services.

Each price entry needs:

- source;
- checked date;
- currency;
- region/tier if relevant.

Options can reference a catalog price model.

Do not:

- mix currencies;
- use missing price as zero;
- invent prices;
- use local Factory runtime as cloud billing.

---

# 7. Performance findings for Fabric-oriented code

DataPass static findings can include provider-neutral rules with Fabric mapping.

Examples:

- repeated full scans of large tables;
- wide SELECT * through join-heavy SQL;
- full refresh on very large dataset despite small declared daily change;
- quality checks after publish instead of before;
- refresh schedules overlapping;
- notebook with many independent responsibilities;
- ambiguous semantic-model relationships;
- excessive bidirectional filtering;
- missing key/grain checks.

Provider-specific rules should be sourced/tested.

---

# 8. File classification for Fabric projects

Useful artifact classes:

- Fabric pipeline JSON;
- Fabric notebook;
- SQL script/model;
- dbt model/test;
- TMDL semantic model;
- PBIP/PBIR-related source artifacts when supported;
- deployment/configuration;
- CI;
- documentation.

Semantic roles remain provider-neutral.

Example:

~~~text
artifact:
  notebook

provider:
  fabric

role:
  enrichment

layer:
  silver
~~~

---

# 9. Extension/tool capability mapping

Catalog should guide the developer to the best tool.

Examples of capability categories:

- browse Fabric workspace;
- inspect/edit notebook source;
- inspect OneLake/lakehouse;
- inspect semantic model;
- edit semantic model;
- run/validate dbt;
- inspect lineage;
- monitor capacity;
- query SQL;
- manage Git/deployment.

Each tool record should say:

- capability;
- installed/available;
- provider;
- side effects;
- auth/connectivity expectations;
- useWhen;
- avoidWhen;
- open/launch route.

Hub consumes this.

---

# 10. Developer recipe examples

Architecture pattern should have practical recipes.

Example: “Develop a Silver SQL model”

~~~text
1. Open model in VS Code.
2. DataPass explains sources, joins and output.
3. Run dbt locally/in selected supported environment.
4. Inspect dbt tests/lineage.
5. Check affected downstream Gold models.
6. Review scenario/performance finding if large full scans are detected.
~~~

Example: “Investigate a capacity peak”

~~~text
1. DataPass shows declared schedules.
2. Import/observe duration evidence if available.
3. Prototype Cloud visualizes overlapping workloads.
4. Compare staggered schedule subvariant.
5. Use provider monitoring tool for real capacity evidence.
~~~

---

# 11. Monitoring recipes

Pattern records should include recommended signals.

## Pipeline

- run status;
- duration;
- retries/failures;
- rows/files moved if available;
- dependency failures.

## Notebook

- duration;
- input/output size if available;
- failure;
- key transform steps;
- observed provider performance metrics only when supplied.

## Warehouse/SQL

- query duration;
- scan/workload/concurrency indicators where available;
- failed queries.

## Semantic model

- refresh status;
- refresh duration;
- model freshness;
- query/user performance where available.

## Data quality

- row counts;
- null/key checks;
- freshness;
- unexpected volume changes.

---

# 12. AI proposal contract example

The client AI should not free-write an architecture only in prose.

Conceptual proposal:

~~~json
{
  "format": "datapass.scenario-proposal",
  "version": 1,
  "catalogVersion": "...",
  "projectRevision": "...",
  "scenarios": [
    {
      "id": "fabric-lakehouse-a",
      "pattern": "fabric.lakehouse.medallion",
      "features": {
        "fabric.transform-engine": "mixed",
        "fabric.dbt": "off",
        "fabric.refresh-strategy": "incremental",
        "fabric.semantic-mode": "direct-lake",
        "fabric.data-access": "copy",
        "fabric.monitoring-profile": "standard"
      },
      "assumptions": [
        {
          "text": "Hourly ingestion is required",
          "basis": "declared"
        }
      ]
    }
  ],
  "questions": []
}
~~~

Engine validates all IDs/values.

---

# 13. Scenario comparison UX

Do not reduce to one overall score.

Show dimensions separately.

Example:

~~~text
Scenario A
Lakehouse + mixed transforms

Developer setup:
  medium

Operational moving parts:
  medium

Incremental complexity:
  medium

Capacity pressure:
  estimated medium

Monitoring:
  standard

Cloud cost:
  partial / assumptions shown

Manual tasks:
  watermark/backfill handling
~~~

Scenario B:

~~~text
Warehouse + SQL

Developer setup:
  low-medium for SQL team

Operational moving parts:
  lower

Transformation flexibility:
  different trade-off

Capacity pressure:
  estimated ...

Monitoring:
  ...
~~~

The user decides.

---

# 14. Factory local prototype mapping for Fabric scenarios

Prototype Cloud can offer “Prototype locally”.

Possible mapping:

~~~text
Fabric Lakehouse
 -> DuckLake

Fabric Warehouse
 -> DuckDB

Fabric SQL transforms
 -> dbt-duckdb / DuckDB SQL

Fabric notebook-style Python transform
 -> Polars/Pandas/Python

Fabric Pipeline
 -> Factory DAG

Fabric API/service integration
 -> FastAPI

Event-like buffer
 -> Redis if scenario requires it

ML workload
 -> sklearn
~~~

Again: conceptual local mapping, not Fabric emulation.

No fake Spark mapping.

---

# 15. Fabric V1 fixture

Build one synthetic but realistic fixture.

Suggested Contoso Fabric project:

Inputs:

- customers;
- stores;
- products;
- orders/sales;
- exchange rates.

Project:

- ingestion;
- Bronze;
- Silver;
- Gold;
- semantic model;
- reports/consumer metadata.

Variants:

A. Lakehouse/mixed transform.
B. Warehouse/SQL-first.
C. Lakehouse + dbt Gold.

Feature toggles:

- incremental;
- Direct Lake/Import;
- dbt;
- monitoring profile;
- quality profile;
- staggered refresh.

Use this fixture across:

- Common Engine;
- DataPass V4;
- Prototype Cloud;
- Factory local mapping;
- Hub capability routing.

One shared fixture exposes integration problems early.

---

# 16. Acceptance criteria for Fabric V1 catalog

Given a synthetic Fabric project + workload:

1. DataPass identifies artifacts/roles/datasets.
2. Common Engine builds static lineage where justified.
3. Workload profile carries sizes/refresh/concurrency with evidence.
4. Prototype Cloud can propose 2–3 catalog-valid scenarios.
5. Each scenario has feature toggles.
6. Invalid feature conflicts are refused.
7. Comparison shows developer/ops/perf/cost drivers separately.
8. Missing capacity evidence is explicit.
9. Monitoring recommendations are attached.
10. Hub can list relevant tools/capabilities.
11. Factory can build one local analogue of a selected scenario.
12. Nothing provisions Fabric.

That is enough for a meaningful Fabric V1.
