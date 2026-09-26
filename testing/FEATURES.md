# Feature tags

A journey lists the DataPass features it exercises (`features`), and a finding names the one it
concerns (`area`). Both use this fixed list, so each run shows which features it covered. A tag
not in this list makes the file invalid.

| Tag | What it covers |
|---|---|
| `onboarding` | Opening a client project for the first time (from the bridge URL), first-run guidance |
| `workspace` | The multi-folder workspace: bridge first, then the native repositories |
| `architecture` | The Architecture panel, the Project tree, Details: components, repositories, folders |
| `variants` | The selected variant (status bar, preview, switching A / B / C) |
| `options` | Comparing architecture options and recording a decision |
| `costs` | Cost lines, `shared` lines counted once, `learning-only` lines labelled |
| `readiness` | Env files, variable names, tools and versions, connections |
| `evidence` | What DataPass observed versus what is declared or unknown (never inferred) |
| `work-orders` | Writing, launching and following work orders for an AI agent |
| `stamps` | The variant, environment and bridge revision recorded on packs and orders |
| `file-context` | Copy Context for My AI on a file |
| `ai-exchange` | Importing an AI's proposal through the reviewed exchange |
| `resources` | Declared resources (storage, VMs, services) and their state |
| `git` | The Git view: changes, branches, pull requests, CI across repositories |
| `format-checks` | Problems reported for the bridge's `.datapass` files |
| `modes` | Vanilla, Standard, DataPass and Advanced presets |
| `file-versions` | Keeping and comparing older versions of a file |
| `toolkit` | The Toolkit: tools, prices, recipes |
| `mcp` | MCP servers: what they are, what they reach, what they send to a model |
