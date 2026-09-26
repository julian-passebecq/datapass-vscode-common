# What DataPass provides, what you fill

Every DataPass client bridge starts from this template (`templates/bridge/` in
[DataPass common](https://github.com/julian-passebecq/datapass-vscode-common)). DataPass provides the
files, their structure, the rules and one validated example of every field. You (the client, or the
AI that prepares the project for you) replace the placeholders with your project's facts.

## How to read the placeholders

| Marker | Meaning | Before the first merge |
|---|---|---|
| `TODO-CLIENT: …` | A text only you can write | Replace every one: `git grep TODO-CLIENT` must return nothing |
| `example-…` ids, `YOUR-ORG`, `YOUR-…-REPO` | Ids and addresses to rename | Rename to your own |
| `00000000-0000-0000-0000-00000000000N`, `rg-example-*`, `stexample*` | Fake ids of the ID map | Replace with the real ids (never a secret), or keep them fake until you know them |
| Components, repositories, options | A validated shape to adapt | Keep, rename, delete or duplicate; every id stays declared before use |

Every file of the template validates with DataPass ≥ 0.26 as it is. Keep it valid at each step: in
VS Code, DataPass checks the files as you type and lists **Problems in project files**.

## When

| Stage | What you fill | Files |
|---|---|---|
| 1. Onboarding (day 1) | Project identity, repositories, environments, one sub-project, current components | `project.json` (`project`, `repositories`, `environments`, `scopes`), `graph.json`, `README.md`, `AGENTS.md` |
| 2. First architecture pass | The data flow, what runs where, the open decisions with their variants and dated prices | `docs/ARCHITECTURE.md`, `options.json`, `graph.json` relations |
| 3. Before anyone runs anything | Tools and versions, the ID map for `dev`, the sign-ins, the local variable names | `project.json` (`toolchain`, `identifiers`, `connections`, `localEnv`), `.vscode/extensions.json` |
| 4. As facts become known | Volumes, key columns, formulas copied from the code, where code runs | `sheet.json` |
| Every change after that | Keep the bridge in step with the native repositories, in its own pull request | any |

## File by file

### `README.md`, `AGENTS.md`, `docs/ARCHITECTURE.md`

| Part | DataPass provides | You fill |
|---|---|---|
| `README.md` | The first-opening steps (clone next to the bridge, open the folder, the AI loop) | Title, the table of sub-projects and where their code is |
| `AGENTS.md` | The rules every AI follows (no secret, no code in the bridge, no claim of "deployed", one pull request per repository) | The project name; project-specific rules below the generic ones (vocabulary, contracts, what must never be touched) |
| `docs/ARCHITECTURE.md` | The sections | The flow, repositories, environments, open decisions, unknowns |
| `EXPECTATIONS.md` | This file | Nothing; a client-specific `DATAPASS_EXPECTS.md` may list what DataPass still expects from you |

### `.datapass/project.json` (manifest v5)

| Field | DataPass provides | You fill | When |
|---|---|---|---|
| `schemaVersion` | `5` | — (write `4` only for someone still on DataPass 0.14–0.17) | — |
| `project.id` | `example-project` | A lowercase id, `^[a-z][a-z0-9_.-]*$` | 1 |
| `project.title`, `description` | placeholders | Plain words | 1 |
| `project.type` | `dev` (the owning AI merges on green CI; work orders stay off until `modules.workOrders` is true) | Keep `dev`; use `work` only if the client wants a person to approve every merge | 1 |
| `modules` | `azure` on, the others off | Switch on only what the project uses (`databricks`, `fabric`, `databases`, `infrastructure`, `airflow`, `powerbi`, `grafana`, `diagramcloud`); `workOrders` stays `false` unless you allow AI work orders | 1 |
| `repositories.bridge` | `path: "."` and a remote to rename | The bridge's own remote URL | 1 |
| `repositories.<key>` | One native repository, one `planned` repository | One key per native repository: `label`, `remote.url` (https or `git@`, never credentials; Azure DevOps Clone address accepted), `branch`, `description`. `planned: true` for one that does not exist yet | 1 |
| `environments` | `dev`, `prod` (`production: true`) | Your environments; every deploy/run/publish operation will name one | 1 |
| `docs` | Architecture, AGENTS, EXPECTATIONS | More files (`path`, optional `repoRef`) or https pages | any |
| `scopes` | One sub-project | One per sub-project: `id`, `title`, `objective`, `repoRef` (its default repository), `itemRefs` (its components), `checklist` | 1 |
| `toolchain.tools` | git, python, az, three extensions | Only tool ids DataPass knows (list in PREPARING_A_PROJECT §12), with version ranges; `optional`, `where: ci` / `fabric` | 3 |
| `identifiers` | Four fake ids (tenant, subscription, resource group, storage account) | Real ids per environment (`values`) or one `value`. **Ids only**: DataPass refuses a URL, token, key or connection string | 3 |
| `connections` | One Azure CLI sign-in for `dev` | Sign-ins (`cli.az`, `cli.fab`, `cli.databricks` profile name), Git bindings (Fabric, Databricks), cloud connections by display name | 3 |
| `localEnv` | An optional `.env` in the pipeline repository, two variable names | The env files and variable **names** a developer needs; a name with no identifier is treated as a secret, fetched by the person from their vault | 3 |
| `resources`, `bindings` | — (not in the template) | Shared machines (VMs) by SSH **alias** only, if you have any: guide 04 §4.5 | when needed |
| `links` | DataPass common | Portals, boards, handbooks (https) | any |

### `.datapass/graph.json` (graph 0.2)

| Field | DataPass provides | You fill | When |
|---|---|---|---|
| `items[]` | Input storage, a Python script with tests, output storage, planned infrastructure | One item per component of the **current** architecture: `id`, `kind`, `label`, `provider` (a known provider id), `description` | 1 |
| `items[].artifacts` | `profile`, `root`, `entry`, `files` | Where its files are: `repoRef` (if not the scope's), `root`, `profile`, `entry`, extra `files`, `generated` outputs with their producer | 1–2 |
| `items[].operations` | `python.tests.run` | The operations you want shown; deploy/run/publish ones name an `environment` and a `target` of resource **names** | 2–3 |
| `items[].status` | `planned` on the infrastructure | What you say (`planned`, `in-progress`…). DataPass still checks the files; never `prepared` to look done | any |
| `relations[]` | consumes, produces, deployedFrom | How data and control flow between components | 2 |

### `.datapass/options.json` (optional, DataPass ≥ 0.15)

| Field | DataPass provides | You fill | When |
|---|---|---|---|
| `criteria` | cost, setup | Your comparison criteria | 2 |
| `decisions[]` | One decision, current option A and one alternative B | One decision per open question: `current` = what graph.json is today (no `changes`), at most two alternatives, each with complete `changes` | 2 |
| `options[].costs` | Two dated lines, one `shared` storage line | Your lines with `source` (https) and `asOf`; `shared` for a line common to several options; `use: "learning-only"` for a free learning tier. Never invent a price | 2 |
| `scenarios` | A and B | One per coherent combination | 2 |
| `chosen`, `decidedOn`, `rationale` | — | Written when a person records a decision; applying it is a separate pull request | later |

Delete the file if nothing is open.

### `.datapass/sheet.json` (optional, DataPass ≥ 0.15)

| Field | DataPass provides | You fill | When |
|---|---|---|---|
| `summary`, `asOf` | placeholders | The order of magnitude in one sentence, the date | 4 |
| `datasets[]` | One result set with one key column | Tables, collections, file sets: sizes as **text orders of magnitude**, the columns that matter, producers and consumers | 4 |
| `formulas[]` | — | Formulas copied exactly from the code, with the file and function that compute them | 4 |
| `runtimes[]` | One, linked to decision `run` | Where code runs, `access` as an SSH alias or a tool, never an address with credentials | 4 |
| `glossary` | One term | Words of the project | 4 |

### `.vscode/extensions.json`

DataPass provides the recommendations matching the template's toolchain. You keep it in step with
`toolchain` (DataPass compares the two and never installs anything).

### What never goes in the bridge

Code, notebooks, data, PDFs, `.env` files, `local.settings.json`, anything under `.datapass/local/`,
a `"$schema"` line, a `board.json` if your project keeps its tasks elsewhere.

## What DataPass does with it

It reads these files, checks them against the disk and the official tools, shows the architecture,
the variants and the costs, and prepares context for your AI. It never executes your files during
discovery, never provisions cloud resources, never signs in for you and never reads a secret value.
