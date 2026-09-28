# DataPass V4 — multi-repository execution plan for Claude

This is the implementation order for the architecture described in the master handoff.

It is intentionally staged. The biggest risk is not lack of code; it is creating incompatible versions of the same “common” concept in DataPass, Prototype Cloud, Factory and Mosaic.

---

# 1. Operating rules

Before work in any repository:

1. fetch the current default branch;
2. read its CLAUDE.md / AGENTS.md / handoff;
3. inspect open PRs;
4. inspect current CI;
5. create a feature branch;
6. preserve current behavior/tests;
7. keep changes scoped to the repo’s product boundary.

Do not merge branches automatically merely because this plan exists.

Do not rewrite default branches.

Do not make one cross-repo change that cannot be rolled back independently.

---

# 2. Repository ownership matrix

| Repository | V4 role | Immediate action |
| --- | --- | --- |
| datapass-vscode-common | Common contracts/catalog/coordination; future shared packages | Start here |
| datapass-vscode | DataPass V4 repo intelligence | Consume common engine incrementally |
| datapass-vscode-cloud | Prototype Cloud | Refocus around scenario/catalog engine |
| datapass-vscode-hub | Capability/tool router | Implement after capability contract |
| datapass-vscode-factory | New local prototype extension | Create after common contracts/workbench boundary |
| datapass-mosaic-vscode | Learning product + donor for workbench-core | Extract generic shell incrementally |
| contoso-data-studio | Donor + first Factory scenario fixture | Generalize generator/runtime concepts |
| cloudiagram GitLab | Later publication consumer | Not critical path |
| Mongoku-datapass | Optional portfolio/read-model | Not critical path |

---

# 3. Current snapshots at plan creation

Orientation only; re-check before edit.

- datapass-vscode-common: 6afdac03b28a42054ba04bdd4579b63ee9ea1ec2
- datapass-vscode: ccb2d8913681729d29e322a2f526b3bcf21fb3b3
- datapass-vscode-cloud: fcedc911719c996bd1885a0d42eb8be8be359e16
- datapass-vscode-hub: 7b716843ef5600c24b85dfe9704e5edeae66ef11
- datapass-mosaic-vscode: 60192bd59c50e6a04b513c5e72d8042ca47917fa
- contoso-data-studio: 5faa9c47c694cea35471d8a7aef991faaac3fb12
- Mongoku-datapass: 53a5a751b5ca2ce4f4ec740d68ce28257eccbb28
- Cloudiagram working branch: 2941d9c3f0c1e21f5bac3a9c297e3f35bdbdd79e

---

# 4. Phase 0 — repository reconciliation and architecture freeze

Goal: establish the real live state before implementation.

## P0.1 datapass-vscode

Read:

- CLAUDE.md;
- handoff/CURRENT.md;
- handoff/PLAN.md;
- IMPLEMENTATION_STATUS.md;
- docs/PREPARING_A_PROJECT.md;
- docs/guide/12_DATAPASS_HOP.md;
- current understanding contract;
- toolkit/catalog code;
- options/cost code;
- graph/project model.

Produce a short live-state note in the working branch:

- current release/source version;
- tests;
- active PRs;
- exact Hop implementation;
- exact toolkit/options ownership;
- exact sync-to-common behavior.

Do not change product behavior in this inventory PR.

## P0.2 datapass-vscode-common

Confirm which directories are release-synced and must not be manually edited.

Create/agree new ownership namespaces for V4 work, e.g.:

~~~text
packages/
handoff/
knowledge/v4/
~~~

Do not move existing synchronized schemas yet.

## P0.3 datapass-vscode-cloud

Inventory current:

- src/catalog.ts;
- src/core.ts;
- src/extension.cjs;
- src/host.cjs;
- docs/ARCHITECTURE.md;
- current UX/features/tests.

Classify:

- keep;
- reuse through common;
- replace;
- remove/defer.

## P0.4 Mosaic

Inventory reusable modules and dependencies:

- MosaicSurface;
- layout;
- runtimeClient;
- ResultTable;
- catalog state;
- graph/DAG components;
- runtime protocol;
- trusted Python control;
- styling.

Separate them from teaching-specific modules.

## P0.5 Contoso

Inventory:

- GeneratorService;
- scenario schema;
- run manifests;
- DuckLakeService;
- explorer;
- dbt runner;
- charts integration;
- API shell;
- tests.

## Exit gate Phase 0

A single checked repo map with:

- exact owners;
- no duplicate intended contract;
- no unresolved “which repo owns this?” for semantic core, catalog, workbench core, Factory DAG.

---

# 5. Phase 1 — Common Semantic Core alpha

Repository: datapass-vscode-common.

Create a focused package/workspace structure.

Possible first files:

~~~text
packages/semantic-core/
  package.json
  src/
    ids.ts
    knowledge.ts
    artifact.ts
    role.ts
    milestone.ts
    operation.ts
    dataset.ts
    lineage.ts
    finding.ts
    metric.ts
    analysis.ts
  tests/
~~~

Do not add provider analyzers yet.

## Required alpha features

- strict IDs;
- Basis/KnowledgeMeta;
- Artifact;
- RoleAssignment;
- Milestone;
- Operation;
- Dataset/Field;
- LineageEdge;
- ExecutionHint;
- Metric;
- Finding;
- Uncertainty;
- AnalysisResult.

## Tests

- serialization;
- invalid references;
- duplicate IDs;
- confidence bounds;
- source anchors;
- basis enum;
- round-trip fixtures.

## Packaging decision

Claude must choose a reproducible cross-repo consumption method and document it.

Allowed directions:

- published package;
- GitHub package;
- pinned source dependency;
- pinned vendor/prepare script.

Not acceptable:

- copy/paste source independently into consumers.

Do not publish externally without user approval.

## Exit gate

Common core can be imported by a tiny standalone test consumer without VS Code.

---

# 6. Phase 2 — datapass.understanding compatibility

Repositories:

- datapass-vscode-common;
- datapass-vscode.

Goal: prove V4 can consume V3 without breaking V3.

## Common

Add:

~~~text
packages/understanding-compat/
~~~

Implement V1 -> semantic IR.

Use current examples from common:

- PySpark;
- SQL;
- Airflow;
- Dockerfile/other existing fixture.

Preserve:

- target path/repo;
- SHA;
- line ranges;
- joins;
- links;
- provenance;
- title/summary.

## DataPass

Add a read-only experimental path/lens that converts current understanding into Common Engine IR.

Do not remove old parser/UI.

Feature flag if needed.

## Acceptance

Given every existing V3 understanding fixture:

- old Hop behavior unchanged;
- V4 semantic projection validates;
- line anchors agree;
- stale hash state agrees.

---

# 7. Phase 3 — first deterministic analyzer: SQL

Repository: datapass-vscode-common.

Create:

~~~text
packages/analyzers/sql/
~~~

Use an AST/parser suitable for supported dialects.

Do not use regex as the primary semantic parser.

V1 supported extraction:

- source tables;
- CTEs;
- output/write when present;
- filters;
- joins/type/keys when resolvable;
- group by;
- aggregate;
- window;
- union;
- merge if parser supports it;
- columns for supported forms.

Output:

- artifact;
- role hints;
- operations;
- datasets;
- lineage;
- uncertainties.

## Findings V1

- possible cross join;
- LEFT JOIN + right-side non-null WHERE behavior;
- SELECT * with join note;
- unknown/dynamic reference.

## Evidence

Every operation/edge anchors to source position/span if parser provides it.

## Acceptance

Fixtures:

- simple select;
- two joins;
- CTE;
- nested query;
- window;
- union;
- malformed;
- unsupported/dynamic;
- dialect variants.

No execution.

---

# 8. Phase 4 — DataPass V4 first visible feature

Repository: datapass-vscode.

Goal: answer “what does this file do?” from deterministic analysis even when no AI understanding file exists.

## UX

For supported SQL file:

~~~text
Understanding
  Artifact
  Role
  Milestones/operations
  Inputs/outputs
  Lineage
  Findings
  Unknowns
~~~

Source selection sync:

- operation -> source range;
- cursor -> closest operation where possible.

## Status

Show:

- deterministic/static;
- AI not used;
- source SHA;
- analyzer version.

If an existing reviewed datapass.understanding exists:

- show/merge with clear basis;
- do not silently overwrite it.

## Acceptance

Opening SQL file in synthetic ETL fixture gives useful explanation with no AI and no execution.

---

# 9. Phase 5 — Python/PySpark static analysis

Repository: common.

Important distinction:

DataPass may analyze PySpark source.

Factory will not fake/execute Spark.

Analyzer goals:

- imports;
- Spark session/table/read/write calls;
- select/filter;
- withColumn/derive;
- joins;
- groupBy/agg;
- windows;
- merge;
- repartition/coalesce as execution hints;
- calls/chaining;
- dynamic unresolved references.

Do not execute Python.

Use Python AST.

AI enrichment can later name business milestones.

## Findings examples

- collect/toPandas risk only when size context warrants;
- repartition(1) concern;
- possible missing join predicate;
- repeated wide operations as heuristic with confidence.

Do not pretend to know runtime Spark strategy.

---

# 10. Phase 6 — Notebook analyzer

Common.

For IPYNB:

- cells;
- markdown headings;
- source;
- language;
- tags;
- saved outputs with explicit basis;
- delegate cells to analyzers.

Milestone grouping:

- deterministic headings/cell blocks first;
- optional AI titles later.

DataPass UI:

- notebook source left;
- milestone understanding right;
- mini lineage bottom.

This is a core user-requested experience.

---

# 11. Phase 7 — Catalog V2 foundation

Common.

Do not replace existing toolkit files immediately.

Introduce new schemas for:

- architecture pattern;
- operation knowledge;
- capability mapping;
- feature option;
- monitoring profile;
- cost/performance driver.

Keep current Tool Toolkit compatible.

## Requirements

- strict schema;
- stable IDs;
- source/date rules;
- no arbitrary executable content;
- provider-neutral core + provider mappings.

## First catalog IDs

Architecture:

- architecture.lakehouse.medallion
- architecture.warehouse.elt
- architecture.hybrid.lakehouse-warehouse
- architecture.batch-bi.simple

Operations:

- operation.join
- operation.aggregate
- operation.window
- operation.merge
- operation.deduplicate
- operation.read
- operation.write

Capabilities:

- semantic-model.inspect
- semantic-model.edit
- workspace.browse
- local.sql.run
- local.prototype.run
- architecture.compare

---

# 12. Phase 8 — Fabric V1 catalog

Common.

Implement the separate Fabric V1 document.

Start with:

- patterns;
- feature toggles;
- consequences;
- monitoring;
- required workload fields;
- capability mappings.

Do not add volatile prices until source/date model is validated.

Then add price/capacity data.

---

# 13. Phase 9 — WorkloadProfile

Common + DataPass.

DataPass derives WorkloadProfile from:

- project sheet;
- datasets;
- static analysis;
- user declarations;
- optional captured evidence.

UI should show:

- value;
- basis;
- source;
- unknown.

No invented defaults hidden from user.

---

# 14. Phase 10 — Prototype Cloud refactor

Repository: datapass-vscode-cloud.

Current product is a Cloud Studio prototype.

Refactor incrementally.

## Keep

- independent extension shell;
- useful navigation/goal UX;
- safe native tool routing;
- existing tests/security.

## Replace/centralize

- local hard-coded catalog data -> Common Catalog;
- ad hoc scenario meaning -> Common Scenario contract.

## New V1 workflow

~~~text
Select/open DataPass project or import analysis snapshot
  -> show current architecture/workload
  -> choose goal
  -> generate/accept 2–3 catalog-valid scenarios
  -> edit feature toggles
  -> compare
  -> export proposal to DataPass options/bridge
  -> optional "Prototype locally" handoff to Factory
~~~

AI:

- optional;
- only proposes catalog IDs;
- assumptions/questions explicit;
- validated before display.

No cloud provision/deploy.

---

# 15. Phase 11 — Scenario engine and options compatibility

Common + DataPass.

Need to decide how feature toggles map to current options.json.

Do not break options version 1 silently.

Possible strategies:

A. extend with a new version;
B. keep options V1 and store feature selections in a new proposal/read-model format;
C. compile feature scenarios down to existing decision picks.

Choose based on current parser/migration burden.

Required behavior:

- A/B/C still works;
- feature toggles work;
- requires/conflicts validated;
- active variant can still drive DataPass views.

---

# 16. Phase 12 — Workbench-core extraction

Common + Mosaic.

This is the start of Factory enablement.

## Extract first

- layout/grid;
- pane interfaces;
- runtime protocol types;
- generic result table;
- generic run status;
- graph/DAG surface if separable.

## Leave in Mosaic

- labs;
- grading;
- missions;
- SparkLab;
- Airflow simulation;
- teaching content.

Mosaic must remain functional.

Use compatibility adapters if needed.

---

# 17. Phase 13 — create Factory repository

Recommended name:

- datapass-vscode-factory.

Before creation:

- confirm name with user if uncertain;
- inspect naming/CI conventions.

Initial structure:

~~~text
src/
  extension/
  webview/
  platform/

runtime/
  factory_runtime/

schemas/
  factory-project.schema.json

fixtures/
  hello-duckdb/

docs/
~~~

Dependencies:

- common semantic-core;
- common runtime-contracts;
- common workbench-core.

First milestone:

- run one DuckDB SQL step;
- show receipt;
- show table result.

---

# 18. Phase 14 — Factory semantic DAG + local orchestrator

Implement strict `factory.yml` as a provider-neutral semantic DAG above dbt.

Before writing a new scheduler, extract/reuse the proven control-flow concepts from Mosaic:

- `runtime/factorylab/engine.py`;
- `runtime/datapass_runtime/factory_workspace.py`;
- `SharedGraphCanvas`;
- `FactoryPipelines.tsx`;
- `PipelineSurface.tsx`.

Replace simulation-only work execution with real local adapters.

Initial node kinds:

- generator;
- sql / DuckDB;
- python;
- polars;
- pandas;
- quality;
- dbt;
- dlt;
- sklearn;
- export.

Required V1 scheduler behavior:

- graph/cycle validation;
- dependency ordering;
- parallel independent ready nodes;
- success/failure/always conditions;
- retries;
- timeout;
- cancellation;
- subprocess isolation;
- local logs/events;
- deterministic normalized run receipts.

dbt is a nested subgraph:

~~~text
Factory DAG
  -> dbt group
       -> dbt internal DAG
~~~

Do **not** require Dagster, Airflow, Meltano, Kubernetes or Docker for this phase.

Dagster can remain a later optional adapter if a concrete post-V1 requirement justifies it.

## Tests

- cycles;
- unknown node/adapter;
- missing dependency;
- independent-node parallelism;
- success/failure edges;
- retry;
- timeout;
- cancellation;
- subprocess failure isolation;
- nested dbt group integrity;
- deterministic receipts;
- observed/local truth labels.

Full rationale: `handoff/V4_ORCHESTRATION_DECISION.md`.

---

# 19. Phase 15 — extract Contoso generator

Repositories:

- contoso-data-studio;
- Factory;
- possibly common runtime contract.

Do not break Contoso first.

Create generic generator interface in Factory/common.

Adapt existing Contoso GeneratorService concepts:

- scenario;
- seed;
- scale;
- manifest;
- generator version/hash;
- file hashes;
- row counts;
- reproduction;
- compare runs.

Port retail scenario data generation through an adapter.

Once Factory retail tests match expected outputs, Contoso may gradually consume the shared generator or remain a verified donor.

---

# 20. Phase 16 — DuckDB/DuckLake

Factory.

DuckDB default.

DuckLake optional.

Prefer local filesystem + Parquet + DuckLake. **Do not introduce MinIO/local S3 emulation in V1** unless a specific scenario explicitly needs object-store semantics.

Generalize Contoso DuckLake patterns:

- bounded data path;
- local catalog;
- schemas/layers;
- snapshots;
- read-only preview;
- no arbitrary external file access.

Add catalog pane.

---

# 21. Phase 17 — dbt

Factory.

Use real:

- dbt Core;
- dbt-duckdb.

Capture:

- manifest;
- run_results;
- tests;
- model lineage.

Expose dbt as a nested subgraph of the global Factory DAG. Clicking the dbt group should expose sources/models/tests and link back to native SQL files.

Do not use Mosaic teaching emulation for Factory.

Optional dbt Charts integration remains external/official.

---

# 22. Phase 18 — Polars/Pandas/sklearn

Factory.

Add execution adapters.

Requirements:

- explicit run;
- source hash;
- environment;
- metrics;
- bounded output preview;
- model receipt.

Create simple ML fixture.

No MLOps platform.

---

# 23. Phase 19 — dlt

Factory.

Add dlt adapter.

Use for quick ingestion.

Integrate dlt as an ingestion activity/group under the global Factory DAG; do not treat dlt as the whole orchestrator.

Record:

- source;
- load;
- produced tables;
- status;
- schema evidence.

---

# 24. Phase 20 — FastAPI / Redis / Docker Compose

Factory.

Only after non-service Factory is usable.

FastAPI:

- loopback;
- health;
- logs;
- start/stop.

Redis:

- Docker by default;
- health;
- stream/cache primitive.

Docker Compose:

- explicit user action;
- known project file;
- status/logs;
- no Kubernetes.

Do not add MinIO, Postgres, Kafka, Grafana or MLflow merely to imitate cloud infrastructure. Add a service only when the demo capability genuinely requires it and a simpler DuckDB/DuckLake/local-file path cannot serve the need.

---

# 25. Phase 21 — Wind scenario

Factory.

Build a compelling local demonstration.

Modes:

A. pure local/no Docker;
B. service/event mode with FastAPI + Redis + Compose.

Add:

- telemetry;
- weather;
- maintenance;
- anomalies;
- Polars features;
- sklearn anomaly/failure model;
- dashboard/KPI.

Investor walkthrough should be deterministic/reproducible.

---

# 26. Phase 22 — Common Engine inside Factory

Factory source is analyzed by same semantic engine.

For selected file show:

- role;
- milestones;
- operations;
- datasets;
- lineage;
- findings.

Merge run receipts as basis=observed, environment=local.

This closes the architecture loop.

---

# 27. Phase 23 — Hub implementation

Repository: datapass-vscode-hub.

Current repo is mostly docs.

Build minimal extension only after capability contract is stable.

V1:

- detect companion DataPass extensions;
- show current project/artifact if available;
- capability cards;
- route to:
  - DataPass;
  - Prototype Cloud;
  - Factory;
  - official tools;
  - later Cloudiagram.

No duplicate analyzer.

No duplicate catalog.

No arbitrary extension command execution; use allowlisted integrations.

---

# 28. Phase 24 — Semantic Git diff

DataPass V4.

Once analysis results are stable:

- compare file analysis between Git revisions;
- operation changes;
- dataset changes;
- join type/key changes;
- lineage changes;
- role/layer changes;
- findings changes.

Integrate with existing Git lens.

This can become one of V4’s most valuable differentiators.

---

# 29. Phase 25 — richer provider analyzers

After the core path is proven:

- Fabric pipeline;
- ADF pipeline;
- dbt native artifacts;
- Databricks bundle/job;
- TMDL/BIM;
- Airflow static;
- Terraform/Bicep/Docker.

Do not implement all at once.

Select based on actual fixture/user need.

---

# 30. Phase 26 — cost/performance engine

Common + Prototype.

Only after WorkloadProfile and catalog are stable.

V1:

- driver-based estimates;
- ranges;
- assumptions;
- confidence;
- missing evidence.

No universal CU/DBU conversion.

Fabric-specific capacity driver model first.

---

# 31. Phase 27 — AI enrichment

DataPass/Prototype.

AI should not be first.

Implement only after deterministic requests/outputs are structured.

Use:

- uncertainties;
- bounded source excerpts;
- catalog IDs.

AI output remains proposal.

---

# 32. Phase 28 — Cloudiagram later

Do not block previous phases.

When useful:

- define sanitized snapshot adapter;
- feed architecture/semantic graph/run receipts;
- generate editable publication.

Keep its GitLab project independent.

---

# 33. CI expectations by repository

## common

Must run:

- TypeScript/build as applicable;
- package unit tests;
- schemas;
- analyzer fixtures;
- compatibility fixtures.

## datapass-vscode

Must preserve existing large unit/desktop suites.

New V4 behavior needs:

- pure unit tests;
- VS Code desktop acceptance for actual UI;
- no regression in Hop/Git/options/toolkit.

## cloud prototype

- unit;
- webview/browser;
- scenario validation;
- no deployment.

## Factory

- runtime unit;
- security;
- DAG;
- VS Code host;
- end-to-end local fixture;
- optional Docker lane.

## Mosaic

- existing tests remain green after workbench extraction.

## Contoso

- existing scenario/ducklake/dbt tests remain green while extraction occurs.

---

# 34. Cross-repo versioning rule

Every consumer must pin a known Common Engine version/commit.

A consumer build should report the common version.

Do not silently use “latest main”.

Scenario/analysis snapshots should record:

- semantic-core version;
- analyzer version;
- catalog version.

This is required for reproducibility.

---

# 35. PR strategy

Do not open one “V4” PR with everything.

Suggested PR sequence in common:

1. docs/handoff;
2. semantic-core alpha;
3. understanding compat;
4. SQL analyzer;
5. catalog core;
6. Fabric catalog;
7. runtime contracts;
8. workbench-core extraction support.

DataPass:

1. consume semantic core;
2. Hop compatibility lens;
3. SQL static analysis;
4. file roles/milestones;
5. Python/notebook;
6. semantic Git diff.

Prototype:

1. common catalog;
2. scenario types;
3. Fabric scenario UI;
4. AI proposal import.

Factory:

1. shell/runtime;
2. DAG;
3. Contoso;
4. dbt/dataframe/ML;
5. dlt;
6. services;
7. wind.

Hub:

1. extension skeleton;
2. capability cards;
3. routes.

---

# 36. Stop conditions

Pause and reassess if:

- common model needs provider-specific hacks for every analyzer;
- DataPass V3 behavior requires breaking migration;
- Factory requires Docker/Kubernetes for basic demos;
- Prototype Cloud starts provisioning;
- AI becomes required for basic file understanding;
- workbench-core extraction destabilizes Mosaic significantly;
- local metrics are being presented as cloud estimates without explicit model;
- two repos start owning the same catalog fact.

---

# 37. First concrete work order for Claude

After reading all handoff docs, Claude should perform **only Phase 0 + Phase 1 design/reconciliation first**, unless Julian explicitly says “code everything now”.

Deliverables:

1. live repo-state matrix;
2. proposed Common Engine package layout reconciled with actual source;
3. exact compatibility mapping for datapass.understanding V1;
4. first semantic-core schema/types;
5. unit tests;
6. PR on datapass-vscode-common;
7. no consumer repo behavior changed yet.

Then proceed vertically.

---

# 38. Final acceptance target for the whole program

A synthetic Fabric-style project can be opened in DataPass.

DataPass understands:

- repositories;
- files;
- roles;
- milestones;
- operations;
- datasets;
- static lineage;
- findings;
- workload;
- uncertainties.

Prototype Cloud uses that plus Catalog to propose 2–3 Fabric scenarios with useful feature-level choices.

The user selects one and chooses “Prototype locally”.

Factory builds/runs a real local analogue using:

- DuckDB/DuckLake;
- dlt/dbt;
- Polars/Pandas;
- sklearn;
- optional FastAPI/Redis/Docker Compose.

Factory reports real local results and metrics.

Hub tells the user which product/tool to use at each step.

No fake Spark is involved.

Cloudiagram may later consume the resulting snapshot to create a polished deck.

That is the integrated V4 target.
