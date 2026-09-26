# Knowledge: cloud tools and MCP servers

Generated from the toolkit baseline of the delivered DataPass version ([toolkit/baseline.json](toolkit/baseline.json), synced by `npm run sync:common` in datapass-vscode; baseline updated 2026-09-26). Each entry is dated in the JSON (`verified.on`); prices and free tiers are there too. Installed extensions are not proof that an operation works on your target.

## MCP servers and agent plugins

An MCP server gives an AI agent tools. DataPass never installs or starts one: it lists them, shows their side effects, and edits `.vscode/mcp.json` only on request, keeping every other entry.

| Name | Id | Use when | Side effects | Docs |
|---|---|---|---|---|
| Fabric Core MCP Server (remote) | `mcp.fabric-core` | An agent must find, list or manage Fabric workspaces, items, folders and workspace roles with the signed-in user's permissions. | reads-remote, writes-remote, credential-prompt, sends-to-model | [docs](https://learn.microsoft.com/en-us/rest/api/fabric/articles/mcp-servers/core-remote/overview-core-mcp-server) |
| Fabric MCP Server (local) | `mcp.fabric-local` | Development: offline Fabric API specs, item schemas and best practices; OneLake files; item creation; Fabric Data Factory pipelines and dataflows. | reads-remote, writes-remote, credential-prompt, sends-to-model | [docs](https://learn.microsoft.com/en-us/rest/api/fabric/articles/mcp-servers/pro-dev-local/overview-local-mcp-server) |
| Fabric IQ MCP (read-only Power BI exploration) | `mcp.fabric-iq` | An agent must find Power BI reports and semantic models, read their metadata and run DAX queries, without changing anything. | reads-remote, credential-prompt, sends-to-model | [docs](https://learn.microsoft.com/en-us/fabric/iq/connectors/fabric-iq-mcp) |
| Power BI Authoring MCP (hosted) | `mcp.powerbi-authoring-hosted` | An agent must change a semantic model that lives in a Fabric workspace, with nothing to install. | reads-remote, writes-remote, credential-prompt, sends-to-model | [docs](https://learn.microsoft.com/en-us/power-bi/developer/mcp/power-bi-authoring-mcp) |
| Power BI Agentic: powerbi-authoring plugin (Skills for Fabric) | `plugin.powerbi-authoring` | An agent must build or refine a semantic model and a PBIR report: semantic-model-authoring and report authoring, design and planning skills, plus the local Power BI Authoring MCP server and the Desktop bridge. | installs-software, reads-remote, writes-remote, credential-prompt, sends-to-model | [docs](https://learn.microsoft.com/en-us/power-bi/developer/agentic/power-bi-agentic-overview) |
| Azure MCP Server | `mcp.azure` | Azure resources: storage accounts and blobs, Functions, resource groups, Log Analytics KQL, azd deployments. | reads-remote, writes-remote, credential-prompt, sends-to-model | [docs](https://learn.microsoft.com/en-us/azure/developer/azure-mcp-server/overview) |

## Extensions, CLIs and apps by platform

| Name | Id | Use when | Side effects | Docs |
|---|---|---|---|---|
| undefined | `ext.fabric` |  |  |  |
| undefined | `ext.fabric-data-engineering` |  |  |  |
| undefined | `ext.jupyter` |  |  |  |
| undefined | `ext.python` |  |  |  |
| undefined | `cli.java` |  |  |  |
| undefined | `ext.fabric-studio` | Edit an item definition without Git, run REST calls, manage deployment pipelines from VS Code. |  |  |
| undefined | `ext.onelake` |  |  |  |
| undefined | `cli.fab` |  |  |  |
| undefined | `cli.python` |  |  |  |
| undefined | `cli.az` |  |  |  |
| undefined | `ext.databricks` |  |  |  |
| undefined | `cli.databricks` |  |  |  |
| undefined | `ext.tmdl` |  |  |  |
| undefined | `app.pbi-desktop` |  |  |  |
| undefined | `app.tabular-editor` |  |  |  |
| undefined | `cli.copilot` |  |  |  |
| undefined | `ws.mcp` |  |  |  |
| undefined | `cli.gcx` |  |  |  |
| undefined | `cli.tofu` |  |  |  |
| undefined | `cli.terraform` |  |  |  |
| undefined | `ext.containers` |  |  |  |
| undefined | `ext.remote-ssh` |  |  |  |
| undefined | `cli.ssh` |  |  |  |
| undefined | `ext.mongodb` |  |  |  |
| undefined | `cli.mongosh` |  |  |  |
| undefined | `ext.drawio` |  |  |  |
| undefined | `cli.git` |  |  |  |
| undefined | `ext.azure-functions` |  |  |  |
| undefined | `cli.func` |  |  |  |
| undefined | `ext.azure-storage` |  |  |  |
| undefined | `ext.cosmosdb` |  |  |  |
| undefined | `ext.azure-resources` |  |  |  |
| undefined | `ext.pgsql` |  |  |  |
| undefined | `ext.neon` |  |  |  |
| undefined | `cli.psql` |  |  |  |
| undefined | `cli.docker` |  |  |  |
| undefined | `ext.gcloud-data` |  |  |  |
| undefined | `cli.gcloud` |  |  |  |
| undefined | `ext.github-actions` |  |  |  |
| undefined | `ext.github-prs` |  |  |  |
| undefined | `ext.azure-pipelines` |  |  |  |
| undefined | `ext.gitlab` |  |  |  |
| undefined | `ext.grafana` |  |  |  |
| undefined | `ext.powerbi-studio` |  |  |  |
| undefined | `pack.powerbi-gbrueckl` |  |  |  |
| undefined | `ext.powerbi-modeling-mcp` | An agent must change a semantic model open in Power BI Desktop or stored as PBIP/TMDL files, with the change reviewed as a Git diff. | reads-remote, writes-remote, credential-prompt, sends-to-model | [docs](https://learn.microsoft.com/en-us/power-bi/developer/mcp/power-bi-authoring-mcp) |
| undefined | `py.fabric-cicd` | Deploy the items of a Fabric Git folder to dev, test and prod, with parameter.yml replacing ids per environment. |  | [docs](https://microsoft.github.io/fabric-cicd/) |
| undefined | `py.semantic-link-labs` |  |  |  |
| undefined | `plugin.power-bi-agentic-development` |  |  |  |
