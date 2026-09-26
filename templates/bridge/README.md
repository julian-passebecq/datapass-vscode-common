# TODO-CLIENT: project title — DataPass bridge

This repository describes TODO-CLIENT project for **DataPass VS Code** (manifest v5, graph 0.2;
DataPass ≥ 0.26). It lists the project's repositories, sub-projects, environments and the files each
component expects. The code lives in the native repositories; removing this bridge breaks nothing.

| Sub-project | Where the code is | Expected state when first opened |
|---|---|---|
| TODO-CLIENT: sub-project | `TODO-CLIENT: native repository` | clone it next to this bridge |

## First opening

1. Clone this repository, then each native repository **next to it** (same parent folder). DataPass
   finds a sibling clone by its Git origin; otherwise Project → Repositories → **Clone** or **Locate**.
2. Open **this folder** in VS Code. DataPass shows the architecture, the variants
   (`.datapass/options.json`) and the project sheet (`.datapass/sheet.json`).
3. Select a component → **Prepare AI context** → paste it into your AI → pull request → merge →
   **Check for updates** → **Get updates**.

Nothing here is deployed and nothing here is secret. See `AGENTS.md`, `EXPECTATIONS.md` and
`docs/ARCHITECTURE.md`.

<!-- Template: https://github.com/julian-passebecq/datapass-vscode-common/tree/main/templates/bridge -->
