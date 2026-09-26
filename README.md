# DataPass common

The public reference for every DataPass client and for the AIs that prepare client projects.

DataPass VS Code shows a company's cloud project as one architecture. The client keeps its code in
its own repositories, which never contain DataPass files, and describes the project in a separate
**bridge repository** (`.datapass/*.json`). DataPass reads that bridge; removing it breaks nothing.

## Current version

- **Released:** 0.25.0 (2026-09-26).
- **Next:** 0.26.0 (MCP servers in the toolkit, cost lines with `shared` and `learning-only`,
  read-only pilot), then 1.0.0.
- A file that uses a newer field states it; DataPass ≥ that version is needed to read it.

## Where the formats are (authoritative until this repository is synced)

These links point at the source repository. A release-synced copy will be added here under
`formats/`, `schemas/` and `examples/`, with a version stamp.

| What | Link |
|---|---|
| How to prepare a client project (the contract) | https://github.com/julian-passebecq/datapass-vscode/blob/main/docs/PREPARING_A_PROJECT.md |
| Step-by-step guide (what the AI prepares, known limits, variants, toolkit) | https://github.com/julian-passebecq/datapass-vscode/tree/main/docs/guide |
| Ready-to-paste prompt for a client's AI | https://github.com/julian-passebecq/datapass-vscode/blob/main/docs/guide/06_PROMPT_FOR_THE_CLIENT_AI.md |
| JSON Schemas (manifest, graph, options, sheet, board, toolkit, work orders) | https://github.com/julian-passebecq/datapass-vscode/tree/main/schemas |
| A complete public example (bridge + two native folders, variants A/B/C) | https://github.com/julian-passebecq/datapass-vscode/tree/main/examples/v3/doc-pipeline |

## What a client prepares

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

## Planned sections

`knowledge/` (cloud features and what each MCP server does), `testing/` (the test batteries a client
can ask for), `formats/` (release-synced copies).
