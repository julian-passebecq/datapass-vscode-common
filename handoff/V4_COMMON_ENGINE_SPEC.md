# DataPass V4 — Common Engine technical specification

This document translates the V4 product direction into a technical contract.

The objective is not to freeze every field before implementation. The objective is to prevent each consumer product from inventing its own incompatible meaning for “file”, “role”, “join”, “lineage”, “finding”, “workload”, “cost” or “evidence”.

---

# 1. Design goals

The common engine must be:

- provider-neutral first;
- pure/headless where possible;
- usable without VS Code;
- local-first;
- deterministic before AI;
- explicit about uncertainty;
- versioned;
- bounded;
- testable with synthetic fixtures;
- compatible with current DataPass contracts during migration;
- usable by DataPass V4, Prototype Cloud, Factory and Hub;
- suitable later for Cloudiagram/publication consumers.

It must not:

- execute arbitrary client code during analysis;
- require a cloud account;
- require a model/API subscription;
- expose secret values;
- treat provider-specific names as universal logical semantics;
- make performance/cost estimates look observed;
- become a second project authority competing with DataPass project/graph/options.

---

# 2. Recommended package decomposition

Proposed top-level packages in datapass-vscode-common once implementation begins:

~~~text
packages/
  semantic-core/
  understanding-compat/
  analyzers/
    sql/
    python/
    notebook/
    dbt/
    airflow/
    fabric/
    adf/
    databricks/
    powerbi-model/
    infra/
  catalog/
  workload/
  findings/
  performance/
  costing/
  scenario/
  runtime-contracts/
  workbench-core/
~~~

Do not create all directories in one empty scaffolding PR unless there is a clear implementation order. Prefer vertical increments.

The most important initial packages are:

1. semantic-core;
2. understanding-compat;
3. one or two analyzers;
4. catalog core;
5. workload/findings primitives.

---

# 3. Core identity model

Every semantic entity needs stable identity.

Recommended basic shape:

~~~ts
type EntityId = string;

interface SourceAnchor {
  repository?: string;
  path?: string;
  sha256?: string;
  lineStart?: number;
  lineEnd?: number;
  jsonPointer?: string;
  symbol?: string;
  nativeId?: string;
}

interface EvidenceRef {
  id: EntityId;
  kind:
    | "source"
    | "native-metadata"
    | "captured-plan"
    | "runtime-receipt"
    | "user-declaration"
    | "ai-proposal"
    | "catalog";
  anchor?: SourceAnchor;
  title?: string;
  asOf?: string;
}
~~~

An entity may have multiple evidence references.

IDs must not be silently regenerated every analysis if a stable deterministic key can be produced from native identity.

---

# 4. Basis / provenance / confidence

Recommended common vocabulary:

~~~ts
type Basis =
  | "declared"
  | "static"
  | "inferred"
  | "estimated"
  | "illustrative"
  | "captured"
  | "observed";

interface KnowledgeMeta {
  basis: Basis;
  confidence?: number; // 0..1, only when meaningful
  evidence?: EvidenceRef[];
  asOf?: string;
  environment?: "local" | "dev" | "test" | "prod" | "unknown" | string;
  note?: string;
}
~~~

Interpretation:

- declared: person/native config/bridge explicitly states it;
- static: directly extracted from code/config by deterministic parsing;
- inferred: deduced from several static/declared facts;
- estimated: quantitative/qualitative approximation using a model;
- illustrative: intentionally synthetic/example-only;
- captured: supplied provider/optimizer/plan metadata;
- observed: runtime evidence/measurement.

Do not attach a confidence number to hard parsed facts merely to look scientific.

---

# 5. Artifact model

Artifact describes a native project thing.

~~~ts
type ArtifactKind =
  | "notebook"
  | "script"
  | "sql-model"
  | "pipeline"
  | "workflow"
  | "dataflow"
  | "semantic-model"
  | "report"
  | "dashboard"
  | "dataset-definition"
  | "infrastructure"
  | "container"
  | "service"
  | "configuration"
  | "test"
  | "package"
  | "bundle"
  | "documentation"
  | "other";

interface Artifact {
  id: EntityId;
  kind: ArtifactKind;
  title: string;
  repository?: string;
  path?: string;
  language?: string;
  technology?: string;
  provider?: string;
  nativeType?: string;
  roles?: RoleAssignment[];
  layer?: DataLayer;
  evidence?: EvidenceRef[];
}
~~~

ArtifactKind answers “what native thing is it?”.

Roles answer “what job does it perform?”.

Do not collapse those two questions.

---

# 6. File/project role taxonomy

Initial role taxonomy:

~~~ts
type SemanticRole =
  | "ingestion"
  | "extraction"
  | "loading"
  | "cleaning"
  | "normalization"
  | "enrichment"
  | "aggregation"
  | "curation"
  | "quality"
  | "reconciliation"
  | "serving"
  | "semantic-modeling"
  | "reporting"
  | "orchestration"
  | "infrastructure"
  | "configuration"
  | "testing"
  | "monitoring"
  | "deployment"
  | "exploration"
  | "feature-engineering"
  | "training"
  | "scoring"
  | "api-service"
  | "generation"
  | "utility"
  | "other";

interface RoleAssignment extends KnowledgeMeta {
  role: SemanticRole;
  primary?: boolean;
}
~~~

A file may have multiple roles.

Rules:

- at most one primary role unless there is a deliberate multi-role case;
- role assignment can be static/inferred/declared;
- no role should be inferred purely from file name if stronger evidence contradicts it;
- “silver/gold/bronze” is a data layer, not a semantic role.

---

# 7. Data layer

~~~ts
type DataLayer =
  | "source"
  | "landing"
  | "raw"
  | "bronze"
  | "silver"
  | "gold"
  | "warehouse"
  | "semantic"
  | "feature"
  | "serving"
  | "unknown";
~~~

Layer can be:

- declared by project conventions;
- inferred from schema/path/table naming with lower confidence;
- extracted from native provider metadata.

Do not assume every project is medallion.

---

# 8. Milestone model

Milestones are human-comprehensible segments of one artifact.

~~~ts
interface Milestone {
  id: EntityId;
  artifactId: EntityId;
  title: string;
  summary?: string;
  role?: SemanticRole;
  source?: SourceAnchor;
  operationIds?: EntityId[];
  inputDatasetIds?: EntityId[];
  outputDatasetIds?: EntityId[];
  children?: EntityId[];
  meta: KnowledgeMeta;
}
~~~

Example notebook segmentation:

~~~text
1. Setup/config
2. Load Bronze orders
3. Normalize schema
4. Join customer reference
5. Calculate KPIs
6. Validate output
7. Write Silver
~~~

Milestones should be generated by:

1. deterministic structure segmentation;
2. semantic operation grouping;
3. optional AI title/intent refinement.

AI must not rewrite the source line ranges without validation against the current file.

---

# 9. Logical operation model

Initial operation vocabulary:

~~~ts
type OperationKind =
  | "source"
  | "read"
  | "write"
  | "filter"
  | "project"
  | "rename"
  | "cast"
  | "derive"
  | "join"
  | "lookup"
  | "aggregate"
  | "window"
  | "sort"
  | "deduplicate"
  | "union"
  | "pivot"
  | "unpivot"
  | "explode"
  | "merge"
  | "upsert"
  | "validate"
  | "assert"
  | "branch"
  | "loop"
  | "trigger"
  | "call"
  | "http-request"
  | "send"
  | "receive"
  | "train"
  | "predict"
  | "persist"
  | "cache"
  | "partition"
  | "other";

interface Operation {
  id: EntityId;
  artifactId: EntityId;
  milestoneId?: EntityId;
  kind: OperationKind;
  title?: string;
  source?: SourceAnchor;
  inputDatasetIds?: EntityId[];
  outputDatasetIds?: EntityId[];
  properties?: Record<string, unknown>;
  implementation?: {
    technology?: string;
    provider?: string;
    nativeOperation?: string;
  };
  meta: KnowledgeMeta;
}
~~~

Physical concerns such as “BroadcastHashJoin”, “Exchange”, “warehouse scale”, etc. belong in implementation/execution evidence, not the core logical kind.

---

# 10. Dataset and field model

~~~ts
type DatasetKind =
  | "table"
  | "view"
  | "file-set"
  | "stream"
  | "collection"
  | "index"
  | "api-payload"
  | "dataframe"
  | "feature-set"
  | "unknown";

interface Dataset {
  id: EntityId;
  name: string;
  kind: DatasetKind;
  provider?: string;
  location?: string;
  layer?: DataLayer;
  grain?: string;
  fields?: Field[];
  metrics?: DatasetMetric[];
  producedBy?: EntityId[];
  consumedBy?: EntityId[];
  meta: KnowledgeMeta;
}

interface Field {
  id?: EntityId;
  name: string;
  dataType?: string;
  nullable?: boolean;
  semanticRole?:
    | "key"
    | "foreign-key"
    | "partition"
    | "time"
    | "measure"
    | "dimension"
    | "text"
    | "vector"
    | "important";
  meaning?: string;
  sources?: FieldSource[];
  meta?: KnowledgeMeta;
}
~~~

Dataset identity resolution is hard. Do not merge two names just because their final token matches.

Use explicit namespace/provider/repository information where available.

---

# 11. Lineage model

Lineage must distinguish several edge families.

~~~ts
type LineageKind =
  | "data"
  | "column"
  | "control"
  | "dependency"
  | "infrastructure"
  | "deployment";

interface LineageEdge {
  id: EntityId;
  from: EntityId;
  to: EntityId;
  kind: LineageKind;
  operationId?: EntityId;
  fields?: Array<{
    from?: string;
    to?: string;
    expression?: string;
  }>;
  meta: KnowledgeMeta;
}
~~~

Do not call source-order links “data lineage”.

Compatibility note:

Current DataPass Hop links can map to data/control/dependency edges when their declared kind says so.

For conservative lexical source hints with no dependency proof, use a separate sequence relation or a note, not a lineage edge.

---

# 12. Execution evidence

Execution knowledge must remain separate from logical operations.

~~~ts
interface ExecutionHint {
  id: EntityId;
  operationId?: EntityId;
  artifactId?: EntityId;
  engine?: string;
  kind:
    | "strategy"
    | "partitioning"
    | "shuffle"
    | "materialization"
    | "parallelism"
    | "cache"
    | "resource"
    | "other";
  value: unknown;
  meta: KnowledgeMeta;
}
~~~

Examples:

- static inference: join may require data movement;
- captured Spark plan: Exchange hashpartitioning(..., 48);
- observed local DuckDB plan/duration;
- observed provider telemetry.

Do not manufacture distributed execution details from local DuckDB/Polars runs.

---

# 13. Metric model

~~~ts
interface MetricValue {
  value: number | string;
  unit?: string;
}

interface Metric {
  id: EntityId;
  subjectId: EntityId;
  name: string;
  value: MetricValue;
  meta: KnowledgeMeta;
}
~~~

Examples:

- rows;
- bytes;
- files;
- duration_ms;
- memory_bytes;
- refresh_per_day;
- capacity_unit;
- dbu;
- compute_hours;
- model_accuracy.

A metric with basis=observed and environment=local must not be used as a cloud runtime fact without an explicit separate estimate model.

---

# 14. WorkloadProfile

WorkloadProfile is a reusable derived projection.

~~~ts
interface WorkloadProfile {
  projectId: string;
  datasets: WorkloadDataset[];
  schedules?: WorkloadSchedule[];
  consumers?: WorkloadConsumer[];
  concurrency?: KnowledgeValue<number>;
  latencyTargets?: LatencyTarget[];
  assumptions?: Assumption[];
}

interface WorkloadDataset {
  datasetId: EntityId;
  rows?: KnowledgeValue<number>;
  bytes?: KnowledgeValue<number>;
  files?: KnowledgeValue<number>;
  dailyGrowthRows?: KnowledgeValue<number>;
  dailyGrowthBytes?: KnowledgeValue<number>;
  refreshEveryMinutes?: KnowledgeValue<number>;
  retentionDays?: KnowledgeValue<number>;
}
~~~

KnowledgeValue wraps value + KnowledgeMeta.

WorkloadProfile should combine:

- project sheet declarations;
- static repo facts;
- captured provider metadata;
- Factory local observations when relevant;
- user/AI proposals as declared/inferred.

It should preserve source per value.

---

# 15. Finding model

~~~ts
type FindingSeverity = "info" | "low" | "medium" | "high";

interface Finding {
  id: EntityId;
  ruleId: string;
  title: string;
  severity: FindingSeverity;
  subjectIds: EntityId[];
  explanation: string;
  whyItMatters?: string;
  verification?: string[];
  missingEvidence?: string[];
  meta: KnowledgeMeta;
}
~~~

Finding rule examples:

- sql.left-join-right-filter;
- sql.possible-cross-join;
- sql.wide-select-star;
- pyspark.collect-large-input;
- pyspark.repartition-one;
- distributed.repeated-wide-operation;
- model.fact-to-fact;
- model.bidirectional-ambiguity;
- pipeline.no-failure-path;
- quality.missing-key-check.

Rules must be conservative.

If a rule depends on dataset size and size is unknown, the finding may be “check needed” rather than “problem”.

---

# 16. Uncertainty and AI questions

~~~ts
interface Uncertainty {
  id: EntityId;
  subjectId?: EntityId;
  question: string;
  reason: string;
  importance: "optional" | "useful" | "required";
  evidence?: EvidenceRef[];
}

interface AnalysisResult {
  artifacts: Artifact[];
  milestones: Milestone[];
  operations: Operation[];
  datasets: Dataset[];
  lineage: LineageEdge[];
  executionHints: ExecutionHint[];
  metrics: Metric[];
  findings: Finding[];
  uncertainties: Uncertainty[];
}
~~~

The analyzer should be useful without AI.

AI receives bounded uncertainties + relevant evidence, not necessarily the entire repository.

---

# 17. Analyzer interface

Recommended conceptual interface:

~~~ts
interface ArtifactAnalyzer {
  id: string;
  version: string;

  detect(input: AnalysisInput): DetectionResult | null;

  analyze(
    input: AnalysisInput,
    level: AnalysisLevel
  ): Promise<Partial<AnalysisResult>>;
}
~~~

Analysis levels:

- L0 Identify;
- L1 Structure;
- L2 Semantics;
- L3 Static lineage;
- L4 Findings/performance drivers;
- L5 AI enrichment;
- L6 Captured/observed evidence merge.

L5 is not necessarily inside the deterministic analyzer package. It can be a separate enrichment stage.

---

# 18. Analyzer registry

Initial registry targets:

## SQL

Detect/parse:

- CTEs;
- reads;
- writes;
- joins;
- join type/keys when determinable;
- filters;
- group by;
- windows;
- unions;
- merge/upsert;
- output columns where possible;
- source-to-target column lineage where parser support is sufficient.

Use a real parser/AST where practical, not regex for the primary lineage engine.

## Python / PySpark source

Detect:

- imports;
- notebook cell markers where present;
- spark.read/table;
- select/filter/withColumn;
- joins;
- groupBy/agg;
- windows;
- write/saveAsTable;
- merge patterns;
- repartition/coalesce as engine-specific hints;
- calls to other notebooks/jobs where explicit.

Do not execute code.

AST analysis must handle unknown/dynamic expressions by generating uncertainty instead of guessing.

## IPYNB

Extract:

- ordered cells;
- language/source;
- markdown headings;
- code boundaries;
- optional saved outputs only with explicit handling;
- parameter cells/tags.

Then delegate code cells to language analyzers.

## dbt

Prefer native artifacts when available:

- manifest;
- catalog;
- run results.

Otherwise parse project models conservatively.

Capture:

- refs;
- sources;
- tests;
- materialization;
- model role;
- column metadata;
- lineage.

## Airflow

Static DAG analysis:

- tasks;
- dependencies;
- schedule;
- retries/timeouts;
- operators;
- external assets where explicit.

Do not import/execute arbitrary DAG modules during discovery.

## Fabric/ADF pipeline JSON

Capture:

- activities;
- dependencies;
- datasets/connections by identifier only;
- notebook/script/copy references;
- parameters;
- schedule/trigger when supplied;
- control-flow containers.

## Databricks bundles/jobs

Capture:

- jobs/tasks;
- notebook/script references;
- dependencies;
- compute declarations;
- schedules;
- parameters;
- outputs if declared.

## TMDL/BIM

Capture:

- tables;
- columns;
- measures;
- relationships;
- cardinality;
- filter direction;
- model expressions where safe;
- partitions/source expressions only under explicit policy.

Do not infer fact/dimension role from names alone.

## Infra

Terraform/Bicep/Docker/Compose:

- resources/services;
- dependencies;
- ports;
- volumes;
- environments by name only;
- deployment relationships.

No secret values.

---

# 19. Static analysis pipeline

Recommended stages:

~~~text
file inventory
  -> analyzer detection
  -> syntax/structure parse
  -> semantic operations
  -> dataset identity resolution
  -> static lineage
  -> role classification
  -> milestone grouping
  -> findings
  -> unresolved questions
  -> optional AI enrichment
  -> merge captured evidence
~~~

Milestone grouping should be a deterministic heuristic where possible.

AI can improve human-facing titles/summaries after the structure is fixed.

---

# 20. Compatibility with datapass.understanding V1

Do not break current V3 Hop.

Create a compatibility layer.

V1 -> V4 mapping:

- target -> Artifact source identity;
- title/summary -> Artifact/milestone presentation;
- step -> Milestone and/or Operation;
- step.lines -> SourceAnchor;
- inputs/outputs -> Dataset references;
- columns -> Field lineage hints;
- links -> LineageEdge when semantics permit;
- joins -> Operation(kind=join) + dataset/key metadata;
- provenance declared/inferred/estimated/illustrative -> closest new Basis;
- target sha256 -> staleness binding.

A V1 file should remain renderable in Hop without conversion committed to disk.

V4 may expose a derived semantic representation in memory/cache.

If a V2/new serialized format is introduced, it needs explicit migration/versioning; do not silently reinterpret V1.

---

# 21. Caching and staleness

Analysis is tied to exact source identity.

Cache key should include at least:

- repository identity;
- relative path;
- content digest;
- analyzer id/version;
- relevant catalog/rule version.

If any changes:

- source SHA changed;
- analyzer version changed;
- relevant catalog semantic rule changed;

the previous analysis becomes stale or requires recomputation.

Do not key solely by path.

---

# 22. Serialization

Not every derived semantic object must be committed to Git.

Three storage modes are useful:

## Native/committed bridge declarations

For stable reviewed project facts such as existing project/graph/options/sheet.

## Derived local cache

For deterministic analyzer output.

Should be safely deletable/rebuildable.

## Reviewed AI/project enrichment

May be committed if the project wants durable human/AI explanations, similar to current understanding files.

Avoid making enormous machine-generated semantic snapshots mandatory Git files.

---

# 23. Catalog architecture

Catalog entries need stable IDs and versioned schemas.

Conceptual entry families:

~~~text
tool.*
service.*
architecture.*
operation.*
practice.*
monitoring.*
finding-rule.*
cost-model.*
recipe.*
capability.*
~~~

Every externally sourced mutable claim needs:

- source URL/reference;
- checkedAt/asOf;
- provider;
- region if price-specific;
- qualification/status.

Do not hardcode volatile price tables into application code.

---

# 24. ArchitecturePattern

Conceptual shape:

~~~ts
interface ArchitecturePattern {
  id: string;
  title: string;
  summary: string;
  capabilities: {
    required: string[];
    optional?: string[];
  };
  typicalRoles: SemanticRole[];
  dataShape?: string[];
  goodWhen?: string[];
  avoidWhen?: string[];
  performanceDrivers?: DriverRef[];
  costDrivers?: DriverRef[];
  monitoringProfiles?: string[];
  featureOptions?: string[];
  providerMappings?: ProviderPatternMapping[];
  sources?: CatalogSource[];
}
~~~

Provider-neutral pattern:

~~~text
architecture.lakehouse.medallion
~~~

Provider mapping:

~~~text
fabric.lakehouse
databricks.delta
local.ducklake
~~~

Do not encode “local.ducklake == Fabric Lakehouse” as equivalence; it is a mapping/analogue for prototype purposes.

---

# 25. FeatureOption

Feature options enable subvariants beyond A/B/C.

~~~ts
interface FeatureOption {
  id: string;
  title: string;
  values: FeatureValue[];
  affectsCapabilities?: string[];
  conflictsWith?: string[];
  requires?: string[];
  developerEffort?: ConsequenceModel;
  operationsEffort?: ConsequenceModel;
  performanceDrivers?: DriverRef[];
  costDrivers?: DriverRef[];
  monitoringImplications?: string[];
  manualTasks?: string[];
  sources?: CatalogSource[];
}
~~~

Examples:

- fabric.dbt;
- fabric.direct-lake;
- fabric.incremental-gold;
- fabric.shortcut;
- fabric.refresh-staggering;
- fabric.monitoring-profile;
- fabric.quality-profile.

Consequences must be descriptive and evidence-backed, not a hidden score that decides for the user.

---

# 26. CostDriver

~~~ts
interface CostDriver {
  id: string;
  unit:
    | "cu"
    | "dbu"
    | "vcpu-hour"
    | "compute-hour"
    | "gb-month"
    | "request"
    | "egress-gb"
    | "refresh"
    | "custom";
  expression?: string;
  inputs: string[];
  provider?: string;
  source?: CatalogSource;
}
~~~

Avoid direct cross-unit normalization.

Scenario UI may compare:

- expected monthly price range;
- compute pressure;
- latency;
- operational complexity;
- developer effort;

without pretending every dimension reduces to one numeric score.

---

# 27. PerformanceDriver

Examples:

- input_bytes;
- row_count;
- join_cardinality;
- skew;
- partition_count;
- materialization;
- cache;
- concurrency;
- refresh_overlap;
- model_size;
- query_complexity.

Performance models should return:

- qualitative or bounded estimate;
- assumptions;
- missing data;
- confidence;
- verification suggestions.

Do not claim exact cloud runtime from local data unless actually measured in that environment.

---

# 28. Factory runtime contracts in common

Factory needs common runtime-neutral contracts, even though actual execution lives in the Factory repo.

Conceptual types:

~~~ts
interface LocalExecutionStep {
  id: string;
  uses:
    | "generator"
    | "dlt"
    | "sql"
    | "dbt"
    | "python"
    | "polars"
    | "pandas"
    | "sklearn"
    | "quality"
    | "service"
    | "docker-compose";
  dependsOn?: string[];
  config: Record<string, unknown>;
}

interface LocalRunReceipt {
  stepId: string;
  status: "queued" | "running" | "succeeded" | "failed" | "cancelled";
  startedAt?: string;
  finishedAt?: string;
  durationMs?: number;
  metrics?: Metric[];
  outputs?: EvidenceRef[];
  error?: SanitizedError;
}
~~~

No “spark-sim” or “fake-spark” adapter exists in this V1 contract.

---

# 29. Hub capability contract

Hub needs a small shared contract:

~~~ts
interface CapabilityProvider {
  capabilityId: string;
  toolId: string;
  action:
    | "open"
    | "inspect"
    | "edit"
    | "run"
    | "validate"
    | "deploy"
    | "publish"
    | "learn";
  sideEffects: string[];
  qualification?: string;
  launch?: {
    kind: "vscode-command" | "url" | "copy-command" | "product-route";
    target: string;
  };
}
~~~

Catalog data cannot be allowed to execute arbitrary commands.

Only known launch kinds/allowlisted command IDs.

---

# 30. AI enrichment contract

AI should receive a bounded structured request.

~~~ts
interface SemanticEnrichmentRequest {
  artifact: Artifact;
  milestones?: Milestone[];
  operations?: Operation[];
  datasets?: Dataset[];
  uncertainties: Uncertainty[];
  sourceExcerpts: Array<{
    anchor: SourceAnchor;
    text: string;
  }>;
  allowedCatalogIds?: string[];
}
~~~

AI output:

~~~ts
interface SemanticEnrichmentProposal {
  artifactId: string;
  roleProposals?: RoleAssignment[];
  milestoneTitles?: Array<{
    milestoneId: string;
    title: string;
    summary?: string;
  }>;
  datasetAliasProposals?: unknown[];
  answers?: Array<{
    uncertaintyId: string;
    answer: string;
    confidence?: number;
  }>;
}
~~~

AI cannot:

- change source hash;
- invent new source spans outside supplied excerpts;
- mark runtime observed;
- approve itself;
- add secret values.

---

# 31. Scenario proposal contract

Prototype Cloud AI can propose only from catalog primitives.

Conceptual:

~~~ts
interface ScenarioProposal {
  id: string;
  architecturePatternId: string;
  featureSelections: Record<string, string | boolean | number>;
  assumptions: Assumption[];
  questions: Uncertainty[];
  mapping?: Array<{
    projectRole: string;
    catalogComponentId: string;
  }>;
}
~~~

The engine validates:

- pattern exists;
- features exist;
- values allowed;
- conflicts/requires satisfied;
- required workload inputs present or flagged missing.

---

# 32. Security model

Pure analyzers:

- read bounded source text/metadata only;
- no eval;
- no exec;
- no import of arbitrary Python;
- no shell;
- no network unless explicitly catalog-refresh process outside analysis;
- scrub/reject credential-shaped values when serialized/exported.

Factory execution is a separate explicit user action with explicit adapters.

Do not reuse Factory execution APIs inside DataPass discovery.

---

# 33. Test strategy

## semantic-core

Property/unit tests for:

- IDs;
- provenance;
- reference integrity;
- serialization;
- bounds;
- invalid cross-references;
- no secret-shaped output if that contract owns scrubbing.

## understanding-compat

Fixtures from existing datapass.understanding examples.

Must preserve:

- line anchors;
- joins;
- links;
- staleness;
- provenance.

## analyzers

Golden fixtures:

- simple;
- nested;
- dynamic/unknown;
- malicious/untrusted;
- incomplete;
- dialect/provider-specific.

The expected result should include explicit uncertainties for unsupported/dynamic cases.

## findings

Each rule needs:

- positive fixture;
- negative fixture;
- unknown/missing-evidence fixture.

## catalog

Strict schema.

Unknown fields rejected unless format explicitly supports extensions.

Prices/volatile claims require date/source rules.

## scenario

Conflict/requires tests.

No invalid feature combination accepted.

## integration

One end-to-end Fabric-style synthetic project:

~~~text
repo files
 -> analysis
 -> workload
 -> scenario proposal
 -> local Factory mapping
~~~

No cloud account needed.

---

# 34. Migration principle

V4 is an evolution, not a rewrite.

Preserve current DataPass V3 behavior while incrementally routing new features through the common engine.

Good migration:

~~~text
existing V3 Hop file
 -> compatibility adapter
 -> common semantic IR
 -> existing Hop UI still works
~~~

Bad migration:

~~~text
delete V3 contracts
replace entire project model
force all bridges to migrate at once
~~~

---

# 35. First vertical slice to implement

A practical first common-engine slice:

1. Artifact/Role/Milestone/Operation/Dataset/Lineage/Evidence types.
2. datapass.understanding V1 compatibility adapter.
3. SQL analyzer using a proper parser for a supported subset.
4. Role classifier for SQL models.
5. static read/join/aggregate/write operations.
6. lineage edges with source anchors.
7. findings: possible cross join + left-join/right-filter.
8. AnalysisResult + uncertainties.
9. DataPass V4 read-only lens consumes it.
10. No AI required.

Second slice:

- Python/PySpark static analyzer;
- notebook integration;
- role/milestone grouping;
- AI enrichment for unresolved intent only.

This proves the architecture before building huge provider catalogs.

---

# 36. Definition of success for Common Engine V4 alpha

Given a supported native file, without executing it, the engine can answer:

- what kind of artifact is this?;
- what is its likely role?;
- what are its major milestones?;
- what logical operations occur?;
- which datasets are read/written?;
- what static lineage can be justified?;
- what possible issues are detected?;
- what is unknown?;
- what source evidence supports every answer?;
- is the analysis current for this exact source hash?;
- which parts came from declaration/static parse/inference/captured evidence?

If it cannot answer one of those, it should say unknown and optionally produce an AI/user question.

That is the contract all products should share.
