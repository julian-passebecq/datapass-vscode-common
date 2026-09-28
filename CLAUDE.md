# Claude entry point — DataPass V4 / Common Engine / Factory

This branch is the planning and coordination entry point for the next DataPass architecture.

**Authorship note:** the architecture analysis recorded here was produced in ChatGPT by **GPT-5.6 Sol** on 2026-09-28. It was **not** produced by GPT-6. Do not rewrite the attribution.

## Read in this order

1. [handoff/V4_MASTER_HANDOFF.md](handoff/V4_MASTER_HANDOFF.md) — full decision history, product boundaries, repository map and the target galaxy.
2. [handoff/V4_COMMON_ENGINE_SPEC.md](handoff/V4_COMMON_ENGINE_SPEC.md) — semantic IR, analyzers, catalog, provenance, lineage, findings, workload, performance/cost and AI bridge.
3. [handoff/V4_FACTORY_SPEC.md](handoff/V4_FACTORY_SPEC.md) — the new local-first Factory product, extracted Mosaic workbench base, Contoso generator extraction and runtime stack.
4. [handoff/V4_ORCHESTRATION_DECISION.md](handoff/V4_ORCHESTRATION_DECISION.md) — final orchestration decision: provider-neutral Factory DAG, Dagster OSS runtime, dbt nested DAG, dlt ingestion, no Airflow/Meltano core.
5. [handoff/V4_FABRIC_V1.md](handoff/V4_FABRIC_V1.md) — first provider vertical: Fabric patterns, feature toggles, developer trade-offs, monitoring and capacity/cost drivers.
6. [handoff/V4_EXECUTION_PLAN.md](handoff/V4_EXECUTION_PLAN.md) — phased multi-repository implementation plan, gates and PR order.
7. Current V3 source of truth in [julian-passebecq/datapass-vscode](https://github.com/julian-passebecq/datapass-vscode), especially its own `CLAUDE.md`, `handoff/CURRENT.md`, `handoff/PLAN.md` and `IMPLEMENTATION_STATUS.md`.

## Critical instruction

Do **not** begin by coding the entire plan in this repository.

This repository is the common reference and proposed future home of shared, provider-neutral contracts/packages. Work is intentionally spread across several existing repositories. Before editing any repository:

- fetch its current default branch;
- inspect open PRs and its own agent instructions;
- establish which repo owns the behavior;
- create a feature branch;
- keep existing tests green;
- do not merge merely because this handoff says a feature is desired.

## Product boundaries that must survive V4

- DataPass remains local-first and Git-aware.
- Discovery never executes arbitrary client code.
- Native cloud/provider files remain authoritative native files.
- Official/specialist extensions and CLIs remain the place to perform real provider work.
- AI is optional. Deterministic analysis comes first; AI resolves uncertainty and proposes typed data.
- Catalog knowledge is versioned, sourced and dated. Unknown is not zero, free, false or observed.
- Static/inferred/captured/observed evidence must never be conflated.
- No secret or credential value goes into the bridge, semantic IR, catalog, Factory project metadata or AI packs.
- Do not create a second competing DataPass project authority.
- Do not make Cloudiagram, Mosaic or Mongoku mandatory dependencies of DataPass V4.
- **Factory V1 has no fake Spark and no Spark simulator.** Existing SparkLab remains only in the separate Mosaic learning product. Factory may support real Spark in the future through an explicit adapter, but V1 uses local engines such as DuckDB, DuckLake, Polars, Pandas and scikit-learn.
- **Kubernetes is not a Factory V1 dependency.** Docker Compose is the service boundary for V1.
- **Dagster OSS is the Factory V1 reference orchestrator**, above dlt/dbt/Polars/sklearn. The Factory semantic DAG remains engine-neutral; dbt is an expandable nested transformation DAG.
- **Local minimalism is mandatory:** DuckDB/local Parquet first, DuckLake when lakehouse semantics matter; no MinIO/S3 emulator, Postgres, Kafka or extra service merely to imitate cloud infrastructure.

## Current coordination branch

- Repository: `julian-passebecq/datapass-vscode-common`
- Branch: `plan/v4-semantic-engine-factory`
- Base main snapshot when this handoff was created: `6afdac03b28a42054ba04bdd4579b63ee9ea1ec2`

The SHA values in the handoff are snapshots for orientation, not a license to ignore newer source.
