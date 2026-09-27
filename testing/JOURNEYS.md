# Example client journeys

These ten journeys are **examples** for a fictional client, Codex Wind Lab: a small wind-turbine
study with a 2D study repository, a 3D blade repository and a platform repository (an ADF-like
pipeline, a Functions-style handler, a Docker Compose "VM"), all run locally. A client's AI writes
its own journeys in its auto repository; these show the level of detail expected.

Each one is a valid `datapass.test-journey` file (`kind: "client"`) in [journeys/](examples/client/journeys/).

| Id | Client goal | Features |
|---|---|---|
| [J01](examples/client/journeys/J01-open-my-project.json) | Open my project from its bridge URL and see my three repositories in one architecture | onboarding, workspace, architecture |
| [J02](examples/client/journeys/J02-choose-a-variant.json) | Compare variants A, B and C, including monthly cost, and understand which cost line is only for learning | variants, options, costs |
| [J03](examples/client/journeys/J03-prepare-dev-deployment.json) | Prepare the dev deployment of the publish function under variant C and hand it to an AI | architecture, variants, readiness, evidence, work-orders, stamps |
| [J04](examples/client/journeys/J04-ai-on-a-file.json) | Ask my AI about the pipeline file with the right context, then apply its proposal safely | file-context, ai-exchange |
| [J05](examples/client/journeys/J05-vm-needs.json) | See what my VM (Docker "VM" of variant B) needs and how DataPass shows it is not observed | resources, readiness, evidence |
| [J06](examples/client/journeys/J06-repository-status.json) | Find which of my repositories have changes, open PRs or failing CI | git |
| [J07](examples/client/journeys/J07-check-bridge-files.json) | See whether my bridge files are correct, and understand each error | format-checks |
| [J08](examples/client/journeys/J08-modes.json) | Use DataPass with the fewest panels, then switch to the full view | modes |
| [J09](examples/client/journeys/J09-file-versions.json) | Keep an older version of a file and compare it | file-versions |
| [J10](examples/client/journeys/J10-tools-and-mcp.json) | Check which tools and MCP servers my project needs and what each one would send to a model | toolkit, mcp |

A full run is J01–J10 in at most two hours. A rerun covers only the journeys that were not
reached.

## DataPass release journeys

DataPass's own journeys for a release candidate live in datapass-vscode's `qa/rc/journeys/`
(`kind: "app"`, ids `R01…`). `qa:prepare --rc` adds them to a client run, before the client's.

| Id | What it checks | Launch |
|---|---|---|
| R01 | The pinned VSIX is the DataPass that runs; nothing signs in on its own | client workspace |
| R02 | Restricted Mode: DataPass shows the project but runs neither Git nor tests, until trusted | fresh copy of `doc-pipeline`, not trusted |
| R03 | A project over several repositories appears in about 2 s; variant switches do not mix states | client workspace |
| R04 | Open a Client Project from the bridge's address; no FOIL or Mongoku command or setting | empty window |
| R05 | A broken project file is an error in Problems, not a missing file | fresh copy of `doc-pipeline` |

## Writing a good journey

- Write the goal as the client would say it, not with DataPass's feature names.
- Put in `expected` what a person would check on screen, one item per line.
- Keep `hints` to where to start: the run should show whether a client finds the way alone.
- Name in `outOfScope` anything that would touch the cloud, sign in, or change the client's code.
