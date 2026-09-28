# DataPass common

The public reference for every DataPass client and for the AIs that prepare client projects.

DataPass VS Code shows a company's cloud project as one architecture. The client keeps its code in
its own repositories, which never contain DataPass files, and describes the project in a separate
**bridge repository** (`.datapass/*.json`). DataPass reads that bridge; removing it breaks nothing.

**Start here:** [How to prepare a client for DataPass](HOW_TO_PREPARE_A_CLIENT.md) (roles, the cycle, the hand-off).

## Current version

The delivered version is in [`VERSION`](VERSION), stamped by each sync. A file that uses a newer
field states it; DataPass ≥ that version is needed to read it.

## Where the formats are

| What | Where |
|---|---|
| JSON Schemas (manifest, graph, options, sheet, board, toolkit, work orders, tests) | [`schemas/`](schemas/) (release-synced) |
| Examples (bridge + native folders; `doc-pipeline` has variants A/B/C) | [`examples/`](examples/) (release-synced) |
| Tools, CLIs and MCP servers, dated | [`knowledge/`](knowledge/README.md) (release-synced) |
| Field reference (the contract) | https://github.com/julian-passebecq/datapass-vscode/blob/main/docs/PREPARING_A_PROJECT.md |
| Step-by-step guide | https://github.com/julian-passebecq/datapass-vscode/tree/main/docs/guide |
| Ready-to-paste prompt for a client's AI | https://github.com/julian-passebecq/datapass-vscode/blob/main/docs/guide/06_PROMPT_FOR_THE_CLIENT_AI.md |
| Testing procedure and journeys | [`testing/`](testing/README.md) |
| Native contracts, native CLIs, metadata-only CI | [`NATIVE_CONTRACTS_AND_CI.md`](NATIVE_CONTRACTS_AND_CI.md) |

`schemas/`, `examples/`, `knowledge/toolkit/` and `VERSION` are copied from the extension
repository by `npm run sync:common`; do not edit them here.

## What a client prepares

Start from the common bridge template, [`templates/bridge/`](templates/bridge/): every file with marked placeholders, and [`EXPECTATIONS.md`](templates/bridge/EXPECTATIONS.md) saying, per file and field, what DataPass provides and what you fill, and when.

1. **Code repositories:** native, with no DataPass file, each runnable and tested on its own.
2. **A bridge repository:**
   - `AGENTS.md`, `README.md` and `docs/ARCHITECTURE.md`;
   - `.datapass/project.json` (manifest v5);
   - `.datapass/graph.json` (graph 0.2);
   - optional `options.json`, `sheet.json` and `board.json`;
   - `.vscode/extensions.json`.
3. **Optionally, an auto repository:** the tests DataPass should launch for the client
   (`datapass-auto.json`, `batteries/`). It is used by the Codex test mode (DataPass ≥ 0.27).

## Rules that never change

- No secret anywhere, only where each secret belongs.
- Every id is declared before it is referenced.
- DataPass never executes client files during discovery.
- DataPass never provisions cloud resources.

## V4 planning / Claude start here

A new multi-repository V4 architecture is being designed on branch `plan/v4-semantic-engine-factory`. It introduces a shared semantic engine/catalog used by DataPass V4, Prototype Cloud, Hub and a new local-first Factory product.

Implementation agents should start with [CLAUDE.md](CLAUDE.md) and [handoff/V4_MASTER_HANDOFF.md](handoff/V4_MASTER_HANDOFF.md). The handoff records the full decision path, technical contracts, Fabric V1 vertical, Factory orchestration decision and phased execution plan. Factory V1 explicitly excludes fake Spark and Kubernetes as required dependencies, uses a **bounded local Factory orchestrator** derived from the existing Mosaic/Factory Lab engine above dlt/dbt; Dagster is optional later. Factory prefers the simplest DuckDB/DuckLake/local-files solution before adding services such as Redis or Docker.
