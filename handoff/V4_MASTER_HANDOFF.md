# DataPass V4 — master handoff for Claude

Date: 2026-09-28  
Architecture analysis author: **GPT-5.6 Sol in ChatGPT — not GPT-6**  
User / product owner: Julian Passebecq

This document is intentionally long. It is not a marketing summary. It records the reasoning path, the existing repositories, what has already been built, what should be reused, what should be separated, the product boundaries, the target architecture and the remaining work. Claude should use it as an orientation map and then verify every current repository before editing.

---

# 1. The problem we were actually trying to solve

The original problem was not “draw prettier cloud architecture”.

Julian repeatedly wanted to be able to start from a project and progressively understand it at every useful level:

~~~text
project
  -> architecture
    -> service / pipeline
      -> task
        -> transformation
          -> table / schema / join
            -> notebook / SQL / code
              -> execution concerns
                -> metrics / evidence
~~~

The recurring concrete request was:

- click a cloud component;
- zoom into the job or pipeline;
- click a task;
- understand the input/output datasets;
- see joins, keys and data model;
- open the notebook/source beside the explanation;
- split the file into a small number of meaningful milestones;
- click one milestone and understand its sub-lineage/operations;
- understand concepts such as joins, aggregations, partitions, shuffles, warehouse usage or capacity implications when evidence supports them;
- identify likely logical/performance issues without pretending a static guess is runtime truth;
- compare architecture alternatives;
- eventually explain/present the project cleanly.

Cloudiagram work initially tried to solve the “architecture -> micro detail -> editable PPTX” side. This led to a useful independent semantic-detail format and drill-down work, but it also exposed a deeper problem: a presentation product cannot explain arbitrary Git projects unless another layer first understands the project.

The central question became:

> How can the system answer “what does this file do?” consistently across SQL, notebooks, pipelines, Spark/PySpark, dbt, Power BI/Fabric metadata, orchestration and infrastructure?

That requires a common semantic model, not more diagrams.

---

# 2. First important realization: a file has a finite set of semantic roles

A Spark notebook, SQL model, pipeline JSON or Python job can be arbitrarily long, but its project role is usually one or more members of a comparatively small taxonomy.

Examples of useful file/project roles:

- ingestion;
- extraction;
- loading;
- cleaning;
- normalization;
- enrichment;
- aggregation;
- curation;
- quality;
- reconciliation;
- serving;
- semantic modeling;
- reporting;
- orchestration;
- infrastructure;
- configuration;
- testing;
- monitoring;
- deployment;
- exploration;
- feature engineering;
- model training;
- scoring;
- API/service;
- synthetic data generation/simulation;
- utility.

A notebook may have one primary role and several secondary roles.

Example:

~~~text
silver_orders.py

artifact kind:
  notebook

technology:
  PySpark

primary role:
  enrichment

secondary roles:
  cleaning
  quality
  curation

layer:
  silver

inputs:
  bronze.orders
  ref.customers

outputs:
  silver.orders
~~~

This is different from merely knowing that the artifact is a “notebook”.

The current DataPass graph already models component kinds such as notebook, workflow, pipeline, dataset, semantic model, infrastructure definition, function, storage and database. That is useful, but it does not fully answer the semantic-role question.

V4 needs both:

- structural artifact kind: “what native thing is this?”;
- semantic role: “what job does it perform in this project?”.

---

# 3. Second realization: transformations also reduce to a finite operation vocabulary

The provider-specific APIs differ, but common data work repeatedly uses the same logical operations.

Core operation taxonomy should include at least:

- read;
- write;
- filter;
- project/select;
- rename;
- cast;
- derive/calculated column;
- join;
- lookup;
- aggregate;
- window;
- sort;
- deduplicate;
- union;
- pivot;
- unpivot;
- explode/unnest;
- merge;
- upsert;
- validate;
- assert;
- branch;
- loop;
- trigger;
- call/invoke;
- API request;
- send/receive event;
- train;
- predict/score.

Physical or engine-specific concerns should be metadata, not primary logical operation kinds.

For example:

~~~text
Spark df.join(...)
SQL JOIN
Fabric/Dataflow merge
Hop database lookup
~~~

can all map to a provider-neutral logical operation:

~~~text
operation.join
~~~

with implementation metadata attached separately.

The common semantic engine should therefore avoid treating “SparkBroadcastHashJoin” as the fundamental operation. The fundamental operation is a join; “broadcast hash” is one possible implementation/captured plan strategy.

---

# 4. DataPass Hop V3 already solved an important part

A major discovery during the review was that the current DataPass V3 “Hop” is not Apache Hop execution.

It already has a strong one-file explanation contract called **datapass.understanding**.

The current format contains, per native file:

- target repository;
- target path;
- normalized SHA-256;
- language;
- title;
- summary;
- steps;
- line ranges;
- inputs;
- outputs;
- columns;
- links;
- joins;
- provenance;
- optional milestone labels.

The current step kinds include:

- source;
- read;
- filter;
- transform;
- join;
- aggregate;
- write;
- task;
- branch;
- config;
- test;
- other.

The V3 UX links visual steps to exact code lines. Cursor movement and visual selection can synchronize.

The SHA allows DataPass to distinguish:

- current explanation;
- stale explanation;
- invalid explanation;
- missing explanation.

This is highly reusable. It should not be thrown away.

However, V3 mostly assumes the client AI prepares the understanding JSON. DataPass validates and renders it; DataPass itself does not deeply parse arbitrary code to generate the explanation.

The V4 direction should be:

~~~text
deterministic static analysis first
          +
AI enrichment only for unresolved semantics
          +
reviewed project/client declarations
          +
optional captured/runtime evidence
~~~

DataPass Hop should become a **lens over a broader semantic engine**, not the whole engine.

Potential UX naming can remain “Hop” if Julian likes it, but the underlying core should have a provider-neutral name such as DataPass Semantic Engine.

---

# 5. Third realization: “lineage” needs several truth levels

The goal is to build our own lineage-like understanding, but we must not collapse different evidence levels.

At least four useful lineage/evidence modes exist:

## Static lineage

Derived from source/configuration analysis.

Example:

~~~text
orders -> join customers -> silver_orders
~~~

Basis: source syntax / AST / metadata. Not runtime proof.

## Declared lineage

Provided by a person, bridge file, AI proposal, architecture graph or native manifest.

Example:

~~~text
Fabric pipeline A -> notebook B -> table C
~~~

It may be accurate, but it is a declaration.

## Captured lineage / plan

Derived from supplied native artifacts such as:

- Spark explain/physical plan;
- dbt manifest/catalog;
- Fabric/ADF JSON;
- TMDL/BIM;
- provider export.

This is stronger than a general inference but still not necessarily one observed run.

## Observed lineage / metrics

Derived from runtime receipts/logs/provider telemetry.

Example:

~~~text
run 493
read table X
wrote table Y
shuffle bytes = ...
duration = ...
~~~

Every edge/fact should be able to carry:

- basis;
- confidence when relevant;
- evidence anchor;
- source artifact;
- line/span or provider object;
- timestamp/as-of where relevant.

Do not automatically “promote” an inference into observed truth.

---

# 6. Provenance model for V4

The prior work used several related vocabularies. V4 should unify them without losing compatibility.

Recommended semantic basis vocabulary:

- declared;
- static;
- inferred;
- estimated;
- illustrative;
- captured;
- observed.

Do not create a special “observed-local” truth level. Instead use:

- basis = observed;
- environment/scope = local.

That lets Factory report real local runtime measurements honestly without implying cloud runtime equivalence.

Examples:

~~~text
join strategy = sort-merge
basis = captured
source = supplied Spark plan
~~~

~~~text
potential shuffle = true
basis = inferred
confidence = 0.82
evidence = SQL/PySpark operation graph
~~~

~~~text
query duration = 340 ms
basis = observed
environment = local
engine = DuckDB
~~~

~~~text
Fabric CU impact = medium
basis = estimated
assumptions = [...]
confidence = medium
~~~

---

# 7. DataPass V4 should become the repo-intelligence product

The emerging product boundary is:

> DataPass V4 answers: “What is actually in this Git project, what does it appear to do, how is it connected, what changed, what is missing, what is risky, and which official tool should I use next?”

DataPass should keep and strengthen:

- Git/repository awareness;
- multiple native repositories;
- bridge/project manifest;
- graph and component ownership;
- file presence/readiness;
- environment IDs/names without secret values;
- architecture view;
- work orders;
- AI bounded context;
- variants/options;
- toolkit/capability knowledge;
- stale-analysis detection;
- source-to-understanding line sync;
- static analysis;
- semantic Git diff;
- findings;
- extension/tool routing;
- explicit uncertainty.

DataPass should not become:

- a cloud deployment engine;
- an arbitrary code execution environment;
- a Fabric/Databricks replacement UI;
- a full Office editor;
- a heavy local data runtime;
- a presentation authoring suite;
- a secret manager;
- a mandatory remote service.

Discovery stays non-executing and local-first.

---

# 8. DataPass V4 needs a headless common engine

The most important architectural change is to move provider-neutral logic below the VS Code extension.

Target conceptual structure:

~~~text
DataPass Common Engine
|
+-- semantic-core
|   +-- artifact
|   +-- classification
|   +-- milestone
|   +-- operation
|   +-- dataset
|   +-- lineage
|   +-- evidence
|   +-- finding
|   +-- workload
|
+-- analyzers
|   +-- sql
|   +-- python / pyspark
|   +-- ipynb
|   +-- dbt
|   +-- airflow
|   +-- adf
|   +-- fabric
|   +-- databricks
|   +-- tmdl / bim
|   +-- terraform / bicep
|   +-- docker / compose
|
+-- catalog
|   +-- tools
|   +-- services
|   +-- architecture patterns
|   +-- operations
|   +-- practices
|   +-- monitoring
|   +-- cost models
|   +-- recipes
|
+-- findings
+-- workload
+-- performance
+-- costing
+-- scenario-engine
+-- runtime-contracts
~~~

Consumer products:

~~~text
DataPass VS Code
Prototype Cloud
DataPass Hub
Factory
later: Cloudiagram
later/optional: Mosaic, Mongoku, other tools
~~~

The common engine should not import vscode APIs.

---

# 9. Why datapass-vscode-common is the coordination home

The existing repository julian-passebecq/datapass-vscode-common already describes itself as the public reference for every DataPass client and for AIs preparing projects.

It already contains release-synced:

- schemas;
- examples;
- toolkit knowledge;
- DataPass version reference.

It also already includes useful examples:

- doc-pipeline with A/B/C variants;
- ETL demo;
- datapass.understanding examples for Python/PySpark/SQL/Airflow-style files;
- options and project bridge examples.

For this reason this repository is the best coordination point for the V4 handoff.

Important migration warning:

Today some folders are copied **from datapass-vscode into common** by npm run sync:common. Claude must not create a circular ownership model.

Recommended migration strategy:

1. Existing V3 synced folders continue to be owned by datapass-vscode until explicitly migrated.
2. New V4 common contracts/packages can begin under new top-level ownership in datapass-vscode-common, for example packages/ and knowledge/v4/.
3. Consumer repositories pin explicit common-engine versions/commits.
4. Only after the shared ownership model is proven should existing shared schemas move from extension-owned to common-owned.
5. Never maintain two hand-edited copies of the same contract.

Do not blindly rewrite the current sync system in the first PR.

---

# 10. The final central product galaxy

The core product set should be intentionally small.

~~~text
                        DATAPASS COMMON ENGINE
                                  |
                  +---------------+----------------+
                  |               |                |
                  v               v                v
             DataPass V4    Prototype Cloud     Factory
             repo truth      architecture       local prototype
                                  |
                                  |
                                  v
                             DataPass Hub
                          routing / tools UX
~~~

This diagram is conceptual. Hub can consume context from all three, but should not become their runtime owner.

## Central products

### DataPass V4

Purpose: understand the real Git project.

### Prototype Cloud

Purpose: propose and compare possible architecture designs/variants using the common catalog and project/workload facts.

Current repository to evolve: **julian-passebecq/datapass-vscode-cloud**.

### Factory

Purpose: turn a selected architecture idea into a fast, low-cost, executable local prototype that can be inspected and demonstrated.

There is no existing Factory repository yet. Recommended new repository name: **julian-passebecq/datapass-vscode-factory** if the product remains a VS Code extension, or **datapass-factory** if the team later decides the UI should not be VS Code-hosted. Current direction strongly favors a VS Code extension using extracted Mosaic workbench components.

### DataPass Hub

Purpose: route the user to the right extension/tool/capability for the current artifact/action.

Current repository: **julian-passebecq/datapass-vscode-hub**. It is currently tiny/documentation-oriented and is suitable for focused implementation.

## Peripheral products

### Mosaic

Repository: **julian-passebecq/datapass-mosaic-vscode**.

Mosaic stays a broad learning/training product. It already has many labs and simulators. V4 should extract reusable workbench infrastructure from it, but should not make the full Mosaic product part of the main developer flow.

### Cloudiagram

Repository: GitLab **julianpassebecq/cloudiagram**.

Current V2 branch/MR at handoff time:

- branch: feat/cloudiagram-v2-desktop-ai-documents;
- SHA: 2941d9c3f0c1e21f5bac3a9c297e3f35bdbdd79e;
- MR !1 open/draft;
- pipeline green.

Cloudiagram is no longer central to the immediate V4/Factory program. It remains useful later as a publication/visual explanation consumer of common-engine/Factory snapshots. Do not delete it. Do not block V4 on it.

### Mongoku

Repository: julian-passebecq/Mongoku-datapass.

Mongoku remains optional portfolio/oversight/read-model territory. It must not become a required authority for V4.

### Contoso Data Studio

Repository: **julian-passebecq/contoso-data-studio**.

This is a critical donor/reference for Factory, not something to discard. It already has:

- deterministic retail synthetic generator;
- Parquet staging;
- local DuckLake;
- DuckDB inspection/query;
- dbt-duckdb;
- dbt tests;
- dbt Charts;
- Gold KPI preview;
- React/Fluent UI shell.

Factory should generalize the reusable concepts. Contoso becomes the first strong Factory scenario pack/reference implementation rather than remaining the only product.

---

# 11. Exact repository snapshots used for this analysis

These SHA values are orientation snapshots from 2026-09-28. Claude must fetch each repo before work.

- datapass-vscode-common main: 6afdac03b28a42054ba04bdd4579b63ee9ea1ec2
- datapass-vscode main: ccb2d8913681729d29e322a2f526b3bcf21fb3b3
- datapass-vscode-cloud main: fcedc911719c996bd1885a0d42eb8be8be359e16
- datapass-vscode-hub main: 7b716843ef5600c24b85dfe9704e5edeae66ef11
- datapass-mosaic-vscode main: 60192bd59c50e6a04b513c5e72d8042ca47917fa
- contoso-data-studio main: 5faa9c47c694cea35471d8a7aef991faaac3fb12
- Mongoku-datapass master: 53a5a751b5ca2ce4f4ec740d68ce28257eccbb28
- Cloudiagram GitLab V2 working branch: 2941d9c3f0c1e21f5bac3a9c297e3f35bdbdd79e

Do not assume these remain current.

---

# 12. The three knowledge catalogs we need

The existing DataPass Toolkit is useful and should be expanded rather than replaced.

However one catalog type is not enough.

V4 needs three conceptual knowledge layers.

## 12.1 Tool / service / extension catalog

Question answered:

> With what tool should I work?

Examples:

- VS Code extensions;
- CLIs;
- MCP servers;
- SDKs/libraries;
- cloud services;
- local tools.

Current Toolkit already stores useful fields such as:

- publisher;
- install instructions;
- pricing/free-tier info;
- source/date verification;
- useWhen;
- avoidWhen;
- side effects;
- recipes;
- transport/host for MCP.

Keep this.

## 12.2 Architecture pattern catalog

Question answered:

> How can this system be structured?

Examples:

- lakehouse medallion;
- warehouse ELT;
- batch ingestion;
- CDC;
- streaming;
- semantic BI;
- document pipeline;
- ML feature/scoring pipeline;
- API ingestion.

Each pattern should describe:

- required capabilities;
- optional capabilities;
- typical component roles;
- data-flow shape;
- volume/latency suitability;
- operational complexity;
- cost drivers;
- performance drivers;
- monitoring requirements;
- provider mappings;
- useful feature toggles;
- goodWhen / avoidWhen;
- required developer tasks.

## 12.3 Operation knowledge catalog

Question answered:

> What does this transformation mean and what concerns does it create?

Examples:

- join;
- window;
- merge;
- repartition;
- cache;
- aggregate;
- deduplicate.

For a join, knowledge might include:

- input/output expectations;
- keys;
- cardinality concerns;
- null semantics;
- skew sensitivity;
- possible data movement;
- broadcast eligibility concepts;
- common logical bugs;
- common performance checks;
- provider/engine mappings.

This operation catalog powers:

- DataPass explanations/findings;
- Prototype Cloud trade-offs;
- Factory UI explanations;
- later Cloudiagram detail rendering.

---

# 13. Toolkit should become DataPass Catalog, not a new separate “Toolkit extension”

Julian suggested a possible extra “DataPass Toolkit” extension. The conclusion was: do not create another product unless needed.

The existing Toolkit/catalog capability is already the right home for the knowledge.

Expand the knowledge model so DataPass Catalog contains:

~~~text
catalog
|
+-- tools
+-- extensions
+-- MCP
+-- services
+-- architecture patterns
+-- operations
+-- provider mappings
+-- practices
+-- monitoring patterns
+-- findings/rules
+-- cost models
+-- recipes
~~~

Hub is the UX/router over this knowledge.

Prototype Cloud uses the architecture side.

DataPass V4 uses the artifact/operation/findings/tooling side.

Factory uses the local-component/recipe side.

Do not duplicate catalog records independently in each extension.

---

# 14. Hub’s exact role

The Hub should not be a fourth analysis engine.

It answers:

> For the selected project/artifact/task, what capability do I need, which installed/available tool provides it, and where should I go?

Example for a Power BI semantic model:

~~~text
Understand project/model
  -> DataPass

Edit semantic model
  -> Power BI Authoring MCP / official supported tooling

Inspect Fabric workspace
  -> Fabric extension / official tool

Compare architecture alternatives
  -> Prototype Cloud

Build local proof-of-concept
  -> Factory

Prepare later presentation
  -> Cloudiagram
~~~

Example for a notebook:

~~~text
Understand notebook
  -> DataPass Hop/Semantic lens

Run against real provider
  -> official Fabric/Databricks tooling

Build a local prototype version
  -> Factory

Compare possible cloud architecture
  -> Prototype Cloud
~~~

Hub should understand:

- installed extensions/tools;
- catalog capabilities;
- supported actions;
- current artifact type/role;
- recommended tool routes;
- limitations;
- useful companion actions.

Hub may open/call known extension commands where safe, or copy commands/instructions. It should not silently install or execute arbitrary catalog commands.

---

# 15. Extension capability broker

We need a provider-neutral capability registry.

Example:

~~~text
capability.semantic-model.inspect
  providers:
    - Fabric IQ MCP
    - Power BI authoring tooling
    - TMDL static analyzer

capability.fabric.workspace.browse
  providers:
    - official Fabric extension
    - Fabric MCP where appropriate

capability.data.local.sql
  providers:
    - Factory DuckDB
~~~

For each tool/extension:

- capabilities it offers;
- required installation;
- connection/auth state if observable;
- side effects;
- whether action is local/read-only/remote/writing;
- useWhen;
- avoidWhen;
- provider scope;
- version qualification;
- open/launch route.

This strengthens DataPass’s existing toolkit/toolchain logic rather than replacing it.

---

# 16. Prototype Cloud’s role after the redesign

Prototype Cloud should become an architecture-composition product, not another generic cloud assistant.

Input:

- DataPass project analysis;
- workload profile;
- user constraints;
- DataPass Catalog;
- optional client-AI answers to unresolved business/context questions.

Output:

- 2–3 candidate architecture scenarios;
- feature toggles/subvariants inside each scenario;
- assumptions;
- missing information;
- trade-offs;
- expected developer/operational effort;
- monitoring plan;
- performance/cost drivers;
- required tools/extensions;
- validated typed JSON.

Prototype Cloud does not provision cloud resources.

It can produce proposals for DataPass options/scenarios.

---

# 17. Architecture alternatives must go beyond simple A/B/C

DataPass V3 already supports architecture decisions/options/scenarios. Keep that model.

The missing concept is **feature-level subvariants** within an architecture.

Example base scenario:

~~~text
Fabric Pipeline
 -> Lakehouse Bronze
 -> Spark or SQL Silver
 -> Gold
 -> Semantic Model
~~~

Feature decisions may include:

- dbt on/off;
- Direct Lake vs Import;
- incremental Gold vs full rebuild;
- shortcuts on/off;
- separate Gold Lakehouse vs same Lakehouse;
- materialized Gold vs views;
- Spark vs SQL vs mixed transformation;
- monitoring profile;
- quality-stage profile;
- refresh staggering;
- Git/deployment complexity level.

Each toggle needs structured consequences, not a “best” label.

Example dbt off:

- fewer tools;
- faster onboarding;
- simpler local/toolchain setup;
- more custom conventions;
- less standardized dependency/test documentation;
- more manual discipline.

Example dbt on:

- explicit model dependencies;
- reusable tests;
- clear SQL model structure;
- extra tool/config/CI;
- developer learning overhead;
- not automatically lower cloud compute.

Do not claim dbt “saves CU” unless a particular design changes actual work and the assumptions support it.

---

# 18. Cost/performance comparison principles

The system should compare architecture options using **drivers**, not magic precise numbers.

Examples of workload drivers:

- dataset rows/bytes;
- daily growth;
- refresh frequency;
- latency target;
- concurrency;
- number/size of joins;
- materialization strategy;
- retention;
- number of consumers;
- semantic refresh frequency;
- ML training/scoring frequency.

Examples of cost/capacity units:

- Fabric CU-related capacity drivers;
- Databricks DBU;
- vCPU-hour;
- warehouse compute-hour;
- storage GB-month;
- requests;
- egress GB.

Never create a fake universal conversion such as “1 CU = N DBU”.

Scenario comparison should expose:

- assumptions;
- known inputs;
- missing inputs;
- estimated range;
- confidence;
- source/date for price data;
- separate native unit drivers.

DataPass V3’s existing honest-cost behavior should remain:

- unknown is not zero;
- currencies are not silently combined;
- no-cloud-cost must be explicit;
- dated/source-backed prices;
- shared resources counted carefully.

---

# 19. Workload Profile should become first-class

The existing project sheet already stores useful dataset size/growth/refresh information. V4 should formalize a reusable WorkloadProfile projection rather than create uncontrolled duplicate files.

Useful fields:

- datasets;
- rows;
- bytes;
- file counts;
- growth;
- refresh frequency;
- partitioning if declared/captured;
- latency target;
- concurrency;
- transform complexity;
- joins;
- retention;
- consumers;
- semantic refresh;
- batch/event profile.

Every quantitative value should carry basis/as-of where relevant.

Prototype Cloud uses WorkloadProfile for scenarios.

Factory can generate local datasets from a workload profile.

DataPass derives parts from repo/bridge/captured evidence.

---

# 20. Static findings and logic checks are valuable

The common engine should support deterministic or confidence-labelled findings.

Examples:

- LEFT JOIN followed by a WHERE condition that requires a right-side non-null key: likely behaves as an inner join;
- join with no obvious predicate: possible cross join;
- SELECT * through wide joins: potentially excessive column movement;
- repeated aggregation/repartition patterns: potential repeated shuffle in distributed engines;
- collect/toPandas-like operation on a potentially large dataset: driver-memory risk if the engine/context makes that relevant;
- repartition(1) on large data: bottleneck risk;
- fact-to-fact or ambiguous bidirectional semantic-model relationships;
- pipeline with no failure path;
- missing data quality checks on declared key/grain.

Every finding should include:

- rule id;
- severity;
- confidence;
- evidence;
- affected artifact/milestone/operation;
- explanation;
- missing evidence;
- suggested verification steps.

Do not assert a defect when evidence only supports a suspicion.

---

# 21. Semantic Git diff is an important V4 opportunity

Once files are analyzed into the same semantic IR, DataPass Git can compare meaning, not only lines.

Example:

~~~text
commit A -> commit B

join type:
  inner -> left

dataset:
  + customer_segment

partition setting:
  200 -> 64

new quality check:
  customer_id not null

architecture:
  new Gold model
~~~

This directly supports the earlier objective of a “GitLens supercharged for data/cloud”.

It belongs in DataPass V4, not Factory.

---

# 22. AI should become an uncertainty resolver, not the parser of everything

Recommended analysis pipeline:

~~~text
native file
  -> deterministic detector/parser
  -> structure
  -> operations/datasets/dependencies
  -> static lineage
  -> static findings
  -> confidence / unresolved questions
  -> optional AI enrichment
  -> reviewed merge into semantic result
~~~

Example unresolved output:

~~~text
artifact:
  pipelines/silver_orders.py

known:
  reads dynamic table expression
  joins customer dimension
  writes silver table

unresolved:
  business meaning of lines 82–139
  dynamic dataset resolves by environment

questions for client AI:
  1. What logical business milestone is implemented by lines 82–139?
  2. Which logical dataset family does the parameterized table name represent?
~~~

This is cheaper, safer and more trustworthy than always giving the entire repository to an AI and accepting its description.

---

# 23. Factory emerged because design/explanation alone is insufficient

A key pivot was recognizing a separate need:

> We need a fast way to actually materialize a data/cloud idea locally, execute it cheaply, inspect it, edit it and show it to an investor/client without requiring Fabric/Databricks/Airflow/Spark/Kubernetes.

This is Factory.

Factory is not DataPass V4.

DataPass understands the real repo.

Prototype Cloud proposes architecture options.

Factory makes a local executable proof-of-concept.

---

# 24. Factory V1 stack decision

Factory V1 should optimize time-to-demo and low local complexity.

Core stack:

- Python;
- DuckDB;
- Parquet;
- optional DuckLake;
- dlt for ingestion;
- **Dagster OSS as the reference global orchestrator / visible project DAG;**
- dbt + dbt-duckdb for SQL modeling;
- Polars;
- Pandas;
- scikit-learn;
- FastAPI where a service boundary is useful;
- Redis where cache/queue/stream/state is useful;
- Docker Compose for optional local services;
- dbt Charts as an optional official/existing surface;
- MotherDuck optional for sharing/publishing beyond the laptop.

Explicitly excluded from Factory V1:

- **fake Spark;**
- **Spark simulation;**
- real Spark runtime by default;
- Airflow as the Factory V1 orchestrator;
- Meltano as a core Factory dependency;
- Kubernetes;
- Kafka;
- MinIO/local S3 emulation by default;
- a mandatory Docker dependency for simple projects.

The existing Mosaic SparkLab stays in Mosaic as a learning feature. Do not port it into Factory.

---

# 25. Why no fake Spark in Factory

Earlier discussion considered using Polars/DuckDB for real local work and separately showing “Spark-equivalent” concepts.

Julian explicitly removed that direction.

Final decision:

> Factory V1 does not have fake Spark, simulated Spark stages, fake shuffles or a pseudo-Spark execution model.

If future Factory versions support Spark, it should be through:

- a real Spark adapter;
- or imported/captured Spark plans/metrics used for analysis;
- or a link to Mosaic if the user wants educational Spark simulation.

Factory’s credibility should come from **real local execution on local engines**, not emulated Spark.

---

# 26. Why DuckDB / DuckLake / Polars / Pandas / sklearn

## DuckDB

Use for:

- analytical SQL;
- local warehouse-style prototype;
- Parquet/CSV/JSON querying;
- dbt-duckdb target;
- observable query execution;
- lightweight local catalog.

## DuckLake

Use when the prototype benefits from lakehouse-like table/file separation.

Do not force DuckLake onto every project.

Simple local prototype:

~~~text
DuckDB
~~~

Lakehouse-oriented prototype:

~~~text
DuckLake catalog
 + managed Parquet files
~~~

## Polars

Use for:

- fast DataFrame transformations;
- feature preparation;
- local processing where SQL is not the best fit;
- ML preprocessing.

## Pandas

Keep available because:

- many client notebooks already use it;
- it is familiar;
- Factory may compare alternative local implementations;
- converting every prototype to Polars is unnecessary.

## scikit-learn

Use for low-friction local ML:

- classification;
- regression;
- clustering;
- anomaly detection;
- preprocessing;
- feature selection;
- metrics.

No need for distributed ML in Factory V1.

---

# 27. FastAPI’s real role

FastAPI does not exist “to support data types”.

It exists when the architecture needs a service/API boundary.

Use cases:

- synthetic IoT/business API;
- scoring endpoint;
- webhook;
- backend service;
- data generator API;
- local service boundary between components.

Example wind prototype:

~~~text
wind simulator
  -> FastAPI
      /telemetry
      /turbines
      /weather
      /maintenance
~~~

A simple CSV -> dbt -> DuckDB project should not launch FastAPI unnecessarily.

---

# 28. Redis’s real role

Redis is optional.

Useful roles:

- cache;
- temporary shared state;
- queue;
- stream/event buffer;
- job status/progress.

Example event-style local architecture:

~~~text
turbine generator
  -> FastAPI
  -> Redis Stream
  -> consumer / dlt
  -> DuckLake
~~~

This can represent the concept of event ingestion without forcing Kafka/Event Hubs into a local V1.

Again: projects that do not need this do not launch Redis.

---

# 29. Docker Compose: yes; Kubernetes: no for V1

Factory components fall into two categories.

In-process/local libraries:

- DuckDB;
- DuckLake;
- Polars;
- Pandas;
- sklearn;
- dbt;
- dlt.

Service components:

- FastAPI when an API/service boundary is part of the prototype;
- Redis when queue/cache/stream/state semantics are actually required.

Do not add Postgres, MinIO, Grafana, MLflow or another service by default. Add a component only when the scenario needs the capability and DuckDB/DuckLake/local files cannot provide the simpler path.

Docker Compose is an appropriate optional service boundary.

Factory should clearly indicate whether a selected local scenario needs containers.

Example:

~~~text
Scenario B
DuckDB + dbt + Polars
Docker required: no

Scenario D
FastAPI + Redis + DuckLake
Docker required: yes
services: 2
~~~

Do not require Minikube/kind/k3d/Kubernetes for Factory V1.

Kubernetes can be a later specialized profile if a prototype explicitly needs container orchestration semantics.

---

# 30. Factory orchestration: semantic DAG + Dagster reference runtime

This decision changed during the design discussion and is now explicit.

Factory needs a **global DAG above dbt**.

The hierarchy is:

~~~text
Factory semantic DAG
  generator
      ↓
  dlt ingestion
      ↓
  dbt transformation group
      ├─ staging
      ├─ intermediate
      └─ marts/tests
      ↓
  Polars/Pandas feature work
      ↓
  sklearn train/score
      ↓
  publish/demo
~~~

The global DAG is provider-neutral and belongs to Factory/Common Engine semantics.

**Dagster OSS is the Factory V1 reference execution/orchestration adapter.**

Why:

- local/open-source;
- clear asset/job graph;
- good fit for data assets;
- can orchestrate Python work;
- can sit above dbt instead of replacing dbt;
- dlt has a Dagster integration path;
- no Kubernetes requirement;
- local UI is available for deeper operations.

Factory should still render its own DAG in the Workbench so the product is not dependent on embedding Dagster's UI.

The dbt DAG is nested, not flattened away:

~~~text
global Factory DAG
  -> dbt group
       -> dbt internal model DAG
~~~

Clicking the dbt group should expose models/tests/sources from dbt artifacts.

dlt remains ingestion, not the project orchestrator.

Airflow remains useful for real client projects and can later be analyzed/imported/routed, but is intentionally not the Factory V1 runtime because the setup/operational surface is larger than needed.

Meltano is not core V1 because it overlaps with dlt/dbt and its documented orchestration path introduces another abstraction and commonly Airflow; keep it as a cataloged alternative/importer for projects that already use it.

Do not build a second production scheduler. Factory owns:

- strict DAG schema;
- stable IDs;
- dependency validation;
- nested groups;
- source/evidence mapping;
- normalized run receipts;
- UI visualization.

Dagster owns execution/runtime orchestration.

Full rationale: [V4_ORCHESTRATION_DECISION.md](V4_ORCHESTRATION_DECISION.md).

---

# 31. Factory runtime profiles / variants

Factory should not impose one stack.

The same logical project can have alternative local implementations.

Examples:

Minimal:

~~~text
CSV -> Pandas -> Parquet
~~~

Analytical:

~~~text
Parquet -> DuckDB -> dbt -> Gold
~~~

High-performance local:

~~~text
Parquet -> Polars -> DuckLake
~~~

Service/event:

~~~text
FastAPI -> Redis -> dlt -> DuckLake
~~~

ML:

~~~text
DuckDB/Polars features -> sklearn -> predictions
~~~

These are real local runtime variants, not fake cloud execution.

Factory can compare actual local metrics:

- runtime;
- memory where measurable;
- rows processed;
- files produced;
- number of steps;
- service/container count;
- project complexity.

It must clearly label these as **local observations**, not Fabric/Databricks runtime predictions.

---

# 32. Extract Mosaic’s reusable workbench base

Mosaic already contains a mature flexible VS Code workbench.

Existing useful pieces include:

- flexible panes/grid;
- native file editing;
- SQL/Python/Markdown surfaces;
- DuckDB/DuckLake runtime;
- import CSV/Parquet/JSON;
- query history;
- Polars runtime;
- chart/data views;
- DAG/pipeline canvases;
- local runtime client/protocol;
- run status;
- catalog view;
- layout persistence.

Mosaic also contains many teaching-specific modules:

- Practice;
- Interview;
- SparkLab;
- Airflow simulator;
- Fabric/Cloud Lab;
- SQL pool simulation;
- Databricks Lab;
- BI Lab;
- grading;
- missions;
- exercise packs.

Do **not** make Factory import the whole Mosaic extension.

Extract a provider-neutral reusable base such as:

~~~text
@datapass/workbench-core
  workspace grid
  split panes
  file/editor bridge
  code viewer
  SQL pane
  notebook pane
  table/data preview
  chart pane
  DAG/graph pane
  lineage mini-map
  runtime protocol
  run status
  layout persistence
~~~

The core should not contain:

- quiz;
- learner;
- exercise;
- solution;
- grading;
- SparkLab;
- simulated Fabric;
- simulated Airflow.

Mosaic then remains a consumer of the extracted workbench core.

Factory becomes another consumer.

Do not force the complete Mosaic refactor to finish before any Factory work; define the boundary first and migrate incrementally.

---

# 33. Contoso Data Studio becomes a source + scenario pack

Contoso Data Studio should not simply be renamed Factory.

It is a valuable working vertical.

Reusable assets to extract/generalize:

- deterministic generator contracts;
- scenario/seed control;
- output to Parquet/JSON/CSV as appropriate;
- DuckLake workspace/catalog handling;
- DuckDB read-only inspection;
- dbt-duckdb invocation/result model;
- dbt Charts integration;
- table/schema/statistics exploration;
- Gold KPI preview patterns;
- web/UI concepts if they fit the VS Code workbench.

The retail data generator should become a generic Factory generator/scenario interface.

Example conceptual API:

~~~python
generate_dataset(
    scenario="retail",
    entity="orders",
    rows=100000,
    seed=42
)
~~~

and optional stream generation:

~~~python
generate_stream(
    scenario="wind",
    entity="telemetry",
    rate_per_second=10
)
~~~

Scenario packs envisioned:

- retail/Contoso;
- wind turbines;
- finance;
- manufacturing;
- logistics.

Each scenario pack can declare:

- entities;
- schemas;
- relationships;
- generation rules;
- anomalies;
- volume profile;
- event profile;
- expected demo story.

---

# 34. Wind-turbine Factory example

Julian explicitly wants the ability to prototype/simulate a cloud-like wind project locally.

A good real-local example:

~~~text
Weather generator ---------+
                            |
Turbine telemetry -> FastAPI +-> Redis Stream
                            |
Maintenance generator -----+
                                  |
                                  v
                                 dlt
                                  |
                                  v
                           DuckLake Bronze
                                  |
                             dbt / DuckDB
                                  |
                         Silver telemetry
                           /             \
                          v               v
                      Gold KPIs         Polars
                                          |
                                          v
                                       sklearn
                                          |
                                          v
                                   anomaly score
                           \             /
                            v           v
                              Dashboard
~~~

Nothing here requires fake Spark.

Possible ML example:

- wind speed;
- temperature;
- vibration;
- rotor speed;
- power output;
- maintenance history;
- anomaly detector or failure-risk classifier.

The demo can show:

- actual generated data;
- actual ingestion;
- actual tables;
- actual dbt transformations;
- actual ML training/scoring;
- actual local run metrics;
- actual lineage from the common semantic model.

This is far stronger for an investor demo than a static diagram.

---

# 35. MotherDuck is optional sharing, not a core dependency

Factory should default to local execution.

MotherDuck may later be an optional target for sharing/querying outside the laptop.

Conceptually:

~~~text
local DuckDB/DuckLake
        |
        +-- optional publish/share -> MotherDuck
~~~

Do not make network/cloud access mandatory.

---

# 36. Cloud analogues in Factory

Factory can map local components to conceptual cloud capabilities, but must avoid claiming technical identity.

Example:

~~~text
Local Factory component      Conceptual cloud analogue

DuckLake                     Lakehouse/Delta-style table layer
DuckDB                       SQL analytics/warehouse engine
FastAPI                      API/app/function-style service boundary
Redis Stream                 event/queue/stream buffer
Polars                       high-performance dataframe transform
dbt                          modeled SQL transformation/test layer
sklearn                      ML training/scoring workload
Docker Compose               local multi-service packaging
~~~

The mapping is educational/planning metadata, not proof that performance/semantics are identical.

Prototype Cloud uses these mappings to produce a local-prototype plan.

---

# 37. Cloudiagram’s later role

Cloudiagram is intentionally removed from the immediate central galaxy.

It still has value later as a publication consumer.

Factory/DataPass could eventually emit a sanitized snapshot containing:

- architecture;
- semantic graph;
- runs;
- datasets/tables;
- schemas;
- lineage;
- observed local metrics;
- model results;
- findings;
- selected scenario.

Cloudiagram can then generate/edit:

- architecture pages;
- drill-down views;
- PPTX/PDF.

This prevents Cloudiagram from needing to become the source-code analyzer.

Do not spend V4/Factory critical-path time expanding Cloudiagram now.

---

# 38. Mosaic’s later role

Mosaic stays outside the central developer product set.

It is valuable for:

- learning;
- practice;
- simulation;
- labs;
- interviews;
- concept exercises.

The V4 program should extract reusable workbench infrastructure but must not merge all Mosaic modules into DataPass/Factory.

Specific final decision:

> **No fake Spark is ported from Mosaic to Factory.**

---

# 39. What not to merge into one product

Do not build a “mega extension” containing all responsibilities.

Bad direction:

~~~text
one extension:
  repo intelligence
  cloud design
  local execution
  learning labs
  PPTX
  portfolio dashboard
  cloud deployment
~~~

Desired direction:

~~~text
common engine/contracts/catalog
       |
       +-> DataPass V4
       +-> Prototype Cloud
       +-> Factory
       +-> Hub
       +-> later consumers
~~~

Shared logic is centralized; product responsibilities stay narrow.

---

# 40. First provider vertical: Fabric

The first architecture-catalog vertical should focus on Fabric because Julian wants a useful V1 rather than a huge multi-cloud encyclopedia.

Fabric V1 should cover real recurring design questions, not decorative alternatives.

At minimum:

- Lakehouse vs Warehouse;
- Spark/Notebook vs SQL vs mixed transform;
- Bronze/Silver/Gold organization;
- full refresh vs incremental;
- Direct Lake vs Import;
- Pipeline orchestration vs notebook chaining where applicable;
- dbt on/off;
- Copy vs Shortcut where applicable;
- capacity scheduling/concurrency;
- refresh staggering;
- monitoring profile;
- quality profile;
- Git/deployment complexity.

Each decision/toggle should expose:

- when useful;
- when avoid;
- developer effort;
- operational effort;
- manual work;
- performance drivers;
- cost/capacity drivers;
- monitoring implications;
- required capabilities/tools;
- compatibility/conflicts;
- assumptions/evidence.

A separate Fabric V1 document in this handoff defines the first concrete catalog.

---

# 41. Monitoring must be catalog knowledge

Monitoring recommendations should be associated with architecture patterns/components.

Example categories for Fabric data projects:

Minimum:

- pipeline failure;
- notebook/job failure;
- semantic refresh failure;
- capacity throttling.

Performance:

- step/notebook duration;
- query duration;
- data volume;
- concurrency;
- capacity usage;
- refresh overlap.

Data quality:

- row volume;
- freshness;
- key/null checks;
- unexpected growth;
- schema drift where relevant.

The exact provider capability/source should be catalog metadata, not hardcoded UI text everywhere.

---

# 42. The bridge/client AI role

The AI client should be able to read:

- common catalog subset;
- DataPass repo analysis;
- workload profile;
- unresolved questions.

It then proposes typed JSON.

Example conceptual output:

~~~json
{
  "architectureFamily": "fabric.medallion-bi",
  "assumptions": [
    "hourly ingestion",
    "large curated dataset"
  ],
  "scenarios": [
    {
      "id": "lakehouse-mixed",
      "pattern": "fabric.lakehouse.medallion",
      "features": {
        "dbt": false,
        "directLake": true,
        "incrementalGold": true,
        "shortcuts": false
      }
    }
  ],
  "questions": [
    "Is hourly latency mandatory?",
    "Can Gold be incrementally maintained?"
  ]
}
~~~

The AI does not define arbitrary new schema fields.

The common engine validates IDs, feature compatibility and missing required information.

AI output remains proposal/interpretation.

---

# 43. DataPass options.json remains useful

Do not replace the existing V3 options/scenario mechanism.

Use it as the project-level materialization of architecture decisions.

Prototype Cloud can generate/propose options data.

DataPass shows/compares it.

Feature toggles may require an extension to the current options contract or a derived scenario contract. Do not break V1 options compatibility without migration support.

A possible model:

~~~text
scenario
  pattern
  decision picks
  feature options
  workload assumptions
  catalog references
~~~

But exact schema design belongs in the Common Engine spec and should be versioned.

---

# 44. Security / honesty rules that must remain

These are non-negotiable:

- no secrets in common/project/catalog/AI files;
- never treat installation as proof of connectivity/readiness;
- never treat static inference as runtime observation;
- never treat local Factory timing as cloud runtime;
- never convert capacity units without an explicit documented model;
- discovery does not execute arbitrary client code;
- imported AI text cannot approve itself;
- version/hash/staleness remains explicit;
- native provider files stay authoritative;
- official/provider tools remain the action surface for real cloud work;
- dangerous commands are not executed from catalog data;
- Factory project execution is explicit user action and limited to declared local adapters.

---

# 45. Repository-specific current direction

## datapass-vscode

Evolve V3 into V4 repo intelligence.

Do not restart.

Reuse:

- graph;
- project;
- options;
- sheet;
- toolkit;
- work orders;
- Git;
- Hop;
- file contexts;
- toolchain/readiness.

Add:

- semantic engine consumption;
- file classification;
- analyzer registry;
- auto-generated static understanding;
- findings;
- uncertainty/AI questions;
- semantic Git diff;
- stronger capability routing.

## datapass-vscode-common

Coordination and future common ownership.

Immediate:

- handoff;
- V4 contracts;
- catalog evolution;
- common-package design.

Do not corrupt current release-sync ownership.

## datapass-vscode-cloud

Refocus toward Prototype Cloud:

- architecture pattern exploration;
- 2–3 scenario generation;
- feature toggles;
- workload-aware comparisons;
- catalog-backed trade-offs;
- monitoring/tooling guidance;
- cost/performance driver comparison.

Do not turn it into a deployment engine.

## datapass-vscode-hub

Implement capability/tool routing.

Keep lightweight.

## datapass-vscode-factory (new)

Build local executable prototype workbench.

Use common engine + extracted workbench core.

## datapass-mosaic-vscode

Stay learning product.

Extract workbench core incrementally.

Do not port fake Spark to Factory.

## contoso-data-studio

Donor + first Factory scenario/reference.

## Cloudiagram

Pause from central roadmap; preserve repo and use later for publication.

## Mongoku

Optional portfolio/read-model; not a V4 dependency.

---

# 46. Desired user experience across products

Example: user opens a Fabric-oriented Git project.

DataPass V4:

~~~text
repo tree
  pipelines/
  notebooks/
  dbt/
  model/

selected file:
  silver_orders.py

classification:
  PySpark notebook
  primary role: enrichment
  layer: silver

milestones:
  1 load orders
  2 clean
  3 customer join
  4 derive revenue
  5 quality
  6 write silver

data map:
  bronze.orders ----+
                    +-> silver.orders
  dim.customer -----+

findings:
  join size unknown
  output grain declared
  no observed runtime evidence

AI check:
  1 unresolved business-intent question
~~~

Prototype Cloud:

~~~text
Based on:
  current project
  workload
  Fabric catalog

Scenario A:
  Lakehouse + SQL/Notebook mixed
  Direct Lake
  incremental Gold
  no dbt

Scenario B:
  Lakehouse + dbt SQL Gold
  Direct Lake
  incremental

Scenario C:
  Warehouse-first
  import semantic model

Each:
  required files/tools
  developer effort
  monitoring
  capacity/cost drivers
  assumptions
~~~

Factory:

~~~text
Prototype Scenario A locally

generator -> dlt -> DuckLake -> dbt/Polars -> sklearn -> dashboard

Run:
  actual local rows
  actual local duration
  actual output tables
  actual tests
  actual model metrics
~~~

Hub:

~~~text
For current action:
  understand repo -> DataPass
  compare cloud architecture -> Prototype
  local proof -> Factory
  edit/run real Fabric -> official Fabric tooling
~~~

This is the end-to-end vision.

---

# 47. What Claude should do first

Do not implement “everything” in one branch.

First produce a verified inventory and dependency graph across the repos.

Then establish the common contracts before UI.

The execution plan document gives the intended ordering.

A high-level safe order is:

1. Common semantic core contract + tests.
2. Compatibility adapter from datapass.understanding v1 into the new IR.
3. DataPass V4 read-only file classification + one deterministic analyzer.
4. Catalog V2 architecture/operation structure, starting Fabric.
5. Prototype Cloud consumes catalog/scenario types.
6. Workbench-core extraction boundary from Mosaic.
7. Factory skeleton + DuckDB/local DAG.
8. Extract Contoso generator/scenario.
9. Add dlt/dbt/Polars/Pandas/sklearn adapters.
10. Add optional FastAPI/Redis/Docker Compose components.
11. Hub capability routing.
12. Semantic Git diff, richer analyzers, workload/perf/cost.
13. Cloudiagram later as publication consumer.

At every step preserve current product usability.

---

# 48. Final product principle

The ecosystem should be able to answer four different questions without conflating them:

**DataPass V4**
> What did we actually build in Git, and what does it mean?

**Prototype Cloud**
> What could we build instead, and what would the trade-offs be?

**Factory**
> Can we materialize a useful version locally, run it, measure it and show it quickly?

**Hub**
> Which tool/product/extension should I use for the next action?

The shared Common Engine ensures those answers use the same vocabulary.

That is the architecture to implement.
