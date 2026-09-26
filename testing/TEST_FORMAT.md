# Test settings: `datapass.codex-tests` and `datapass.test-journey`

Two kinds of test runs use the same two formats:

- **App tests** (`purpose: "app"`): the tester tests the DataPass VSIX itself: installing, opening,
  closing and switching several projects, themes, reloads, error logs. It also answers DataPass's
  open questions.
- **Client journeys** (`purpose: "client"`): the tester uses DataPass as one client would, on
  that client's project, and walks the client's journeys.

The settings live in a **settings repository** written by the client's side (the client's AI or
the tester), not by DataPass. It holds settings only: the config, journey files, optional fixtures
and a three-line `AGENTS.md`. Deleting it changes nothing in any project's code or bridge.

```
datapass-codex-tests.json   the config (format datapass.codex-tests)
journeys/*.json             journeys (format datapass.test-journey)
fixtures/                   optional: files a journey needs (for example a broken bridge copy)
AGENTS.md                   three lines: read common/testing, then create or update these settings
```

DataPass also reads `datapass-auto.json` as an older name for the config. Complete, valid
examples are in [examples/](examples/): [examples/client/](examples/client/) (the fictional
client Codex Wind Lab, journeys J01–J10) and [examples/app/](examples/app/) (DataPass's examples
plus Codex Wind Lab, journey A01).

**Check your files** before a run, from a `datapass-vscode` clone (no VS Code and no network
needed):

```
npm run qa:prepare -- --auto <settings repository> --check [--report <report.json>]
```

Exit code 0 means valid. Exit code 2 means invalid, and each reason is printed. That validator is
the reference: when this page and the validator disagree, the validator wins.

## The config (`datapass.codex-tests`, version 1)

### Client journeys (`purpose: "client"`)

```json
{
  "format": "datapass.codex-tests", "version": 1, "purpose": "client",
  "client": { "id": "codex-wind-lab", "title": "Codex Wind Lab (fictional)" },
  "datapass": { "version": "0.26.0", "vsix": "vsix/datapass-vscode-0.26.0.vsix" },
  "workspace": {
    "bridge": { "remote": "https://github.com/julian-passebecq/codex-datapass-bridge", "folder": "codex-datapass-bridge" },
    "repositories": [
      { "remote": "https://github.com/julian-passebecq/datapass-codex-fakeclient",  "folder": "wind-study-2d" },
      { "remote": "https://github.com/julian-passebecq/datapass-codex-fakeclient2", "folder": "wind-blade-3d" },
      { "remote": "https://github.com/julian-passebecq/datapass-codex-fakeclient3", "folder": "wind-platform" }
    ]
  },
  "journeys": ["journeys/J01-open-my-project.json", "journeys/J02-choose-a-variant.json"],
  "report": { "remote": "https://github.com/julian-passebecq/datapass-codex-test", "folder": "reports/client" },
  "limits": { "runMinutes": 120, "journeyMinutes": 20 }
}
```

### App tests (`purpose: "app"`)

These configs have no `client` and no `workspace`. Instead, `workspaces[]` lists 1 to 10 projects
to open, close and switch between. Each one has its own `id` and `title`, and its own
`.code-workspace` file. Several workspaces may share one clone, each with a different `path`.

```json
{
  "format": "datapass.codex-tests", "version": 1, "purpose": "app",
  "datapass": { "version": "0.26.0", "vsix": "vsix/datapass-vscode-0.26.0.vsix" },
  "workspaces": [
    { "id": "doc-pipeline", "title": "DataPass example: Document pipeline",
      "bridge": { "remote": "https://github.com/julian-passebecq/datapass-vscode", "folder": "datapass-vscode", "path": "examples/v3/doc-pipeline" },
      "repositories": [] },
    { "id": "codex-wind-lab", "title": "Codex Wind Lab (fictional client)",
      "bridge": { "remote": "https://github.com/julian-passebecq/codex-datapass-bridge", "folder": "codex-datapass-bridge" },
      "repositories": [{ "remote": "https://github.com/julian-passebecq/datapass-codex-fakeclient", "folder": "wind-study-2d" }] }
  ],
  "journeys": ["journeys/A01-switch-projects.json"],
  "report": { "remote": "https://github.com/julian-passebecq/datapass-codex-test", "folder": "reports/app" },
  "limits": { "runMinutes": 120, "journeyMinutes": 20 }
}
```

| Field | Meaning |
|---|---|
| `purpose` | `app` (test the VSIX itself, several workspaces) or `client` (one client's journeys) |
| `client` | `client` only: `id` (lowercase, used in run ids) and a display `title` |
| `datapass.version` | The DataPass release under test |
| `datapass.vsix` | Optional: the VSIX file, **a local path** relative to the run root, ending in `.vsix`. See [CODEX_PROCEDURE.md](CODEX_PROCEDURE.md) for how to build it from the released commit |
| `workspace` | `client` only: the `bridge` and the native `repositories[]` |
| `workspaces[]` | `app` only: 1–10 projects, each with `id`, `title`, `bridge` and `repositories[]` |
| `bridge`, `repositories[]` entries | `remote` (https), `folder` (the clone under the run root) and optional `path` (a sub-folder inside that clone) |
| `journeys[]` | 1–50 journey files, relative to the settings repository |
| `report` | The audit repository (`remote`) and its folder: `reports/app` or `reports/client` |
| `limits` | Optional: `runMinutes` (whole run) and `journeyMinutes` (one journey) |

Rules:

- Every `folder`, `path` and file name is a relative path without `..`, and no segment starts with
  a dot. Absolute paths and drives are refused.
- Remotes are `https://` only.
- Objects are closed: an unknown field makes the file invalid.
- No secret, token or personal path anywhere.

## Journeys (`datapass.test-journey`, version 1)

A journey is a goal, not a script. The tester reaches it through DataPass's UI and says whether it
got there.

```json
{
  "format": "datapass.test-journey", "version": 1,
  "id": "J03", "kind": "client", "title": "Prepare the dev deployment of the publish function (variant C)",
  "as": "client engineer, cloud beginner",
  "goal": "Know exactly what is needed to deploy the publish function to dev under variant C, and hand the preparation to an AI.",
  "setup": { "client": "codex-wind-lab", "mode": "Standard", "variant": "C", "environment": "dev" },
  "hints": ["Start from the Architecture panel", "Readiness shows what is missing"],
  "expected": [
    "The function component shows its repository, folder and the dev target",
    "Readiness lists the tools needed (Python, Azure Functions extension) and marks cloud evidence as not observed",
    "A work order or Copy Context pack names variant C, dev and the bridge revision",
    "Nothing is deployed and no sign-in is asked for"
  ],
  "features": ["architecture", "variants", "readiness", "evidence", "work-orders", "stamps"],
  "outOfScope": ["real Azure deployment"]
}
```

| Field | Meaning |
|---|---|
| `id` | Capital letter + letters, digits or `-` (2–20 characters). By convention `J01…` for client journeys and `A01…` for app journeys |
| `kind` | `client` or `app`; it must equal the config's `purpose` |
| `title` | One line |
| `as` | Who is using DataPass in this journey (role, experience) |
| `goal` | What they want to achieve, in their own words |
| `setup` | Optional: `client` (a client or workspace `id` of the config), `mode` (Vanilla, Standard, DataPass, Advanced), `variant` and `environment` to start from |
| `hints` | Optional: where a person would start; never a list of commands |
| `expected` | 1–20 things visible when the goal is reached; each one is judged met / not met / unclear |
| `questions` | Optional, mostly for app journeys: open questions this journey must answer (one `answers[]` entry each in the report) |
| `features` | 1–20 tags from [FEATURES.md](FEATURES.md) |
| `outOfScope` | What the tester must not do in this journey |

A journey is data. DataPass never executes anything from it; the tester follows it in the UI.

More examples: [JOURNEYS.md](JOURNEYS.md).
