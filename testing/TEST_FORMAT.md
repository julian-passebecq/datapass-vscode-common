# Test settings: `datapass.codex-tests` and `datapass.test-journey`

Two kinds of test runs use the same two formats:

- **App tests** (`purpose: "app"`): the tester tests the DataPass VSIX itself: installing, opening,
  closing and switching several projects, themes, reloads, error logs. It also answers DataPass's
  open questions. These settings live in DataPass's own test repository.
- **Client journeys** (`purpose: "client"`): the tester uses DataPass as one client would, on
  that client's project, and walks the client's journeys. These settings live in the client's
  **auto repository**.

Both repositories hold settings only: a config file, journey files, optional fixtures and a
three-line `AGENTS.md`. Deleting them changes nothing in any project's code or bridge.

```
datapass-tests.json     the config (format datapass.codex-tests)
journeys/*.json         journeys (format datapass.test-journey)
fixtures/               optional: files a journey needs (for example a broken bridge copy)
AGENTS.md               three lines: read common/testing, then create or update these settings
```

> The validators in DataPass (`npm run qa:prepare`) are the reference. If this page and a
> validator disagree, the validator wins and this page is corrected.

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

The same fields, except that `workspaces[]` lists several projects to open, close and switch
between. Each one gets its own `.code-workspace` file.

```json
{
  "format": "datapass.codex-tests", "version": 1, "purpose": "app",
  "client": { "id": "datapass-app", "title": "DataPass app tests" },
  "datapass": { "version": "0.26.0", "vsix": "vsix/datapass-vscode-0.26.0.vsix" },
  "workspaces": [
    { "id": "doc-pipeline", "title": "DataPass example: doc pipeline",
      "bridge": { "remote": "https://github.com/julian-passebecq/datapass-vscode", "folder": "datapass-vscode", "path": "examples/v3/doc-pipeline" },
      "repositories": [] },
    { "id": "codex-wind-lab", "title": "Codex Wind Lab (fictional client)",
      "bridge": { "remote": "https://github.com/julian-passebecq/codex-datapass-bridge", "folder": "codex-datapass-bridge" },
      "repositories": [
        { "remote": "https://github.com/julian-passebecq/datapass-codex-fakeclient", "folder": "wind-study-2d" }
      ] }
  ],
  "journeys": ["journeys/A01-switch-projects.json"],
  "questions": "QUESTIONS.md",
  "report": { "remote": "https://github.com/julian-passebecq/datapass-codex-test", "folder": "reports/app" },
  "limits": { "runMinutes": 120, "journeyMinutes": 20 }
}
```

| Field | Meaning |
|---|---|
| `purpose` | `app` (test the VSIX itself, several workspaces) or `client` (one client's journeys) |
| `client` | `id` (lowercase, used in run ids) and a display `title` |
| `datapass.version` | The DataPass release under test |
| `datapass.vsix` | The VSIX file, **a local path** relative to the run root. See [CODEX_PROCEDURE.md](CODEX_PROCEDURE.md) for how to build it from the released commit |
| `workspace` | `client` only: the `bridge` and the native `repositories[]`, each with an https `remote` and the `folder` it is cloned into |
| `workspaces[]` | `app` only: several projects, each with an `id`, a `title`, a `bridge` and `repositories[]`. `bridge.path` is a sub-folder inside the clone (an example inside a repository) |
| `journeys[]` | Journey files, relative to the settings repository |
| `questions` | `app` only: the file of open questions the run answers |
| `report` | The audit repository and its folder: `reports/app` or `reports/client` |
| `limits` | Minutes for the whole run and for one journey |

Rules:

- Every `folder` and `path` is relative to one **run root**. `..` and absolute paths are refused.
- Remotes are `https://` only.
- No secret, token or personal path anywhere.

## Journeys (`datapass.test-journey`, version 1)

A journey is a goal, not a script. The tester reaches it through DataPass's UI and says whether it
got there.

```json
{
  "format": "datapass.test-journey", "version": 1, "kind": "client",
  "id": "J03", "title": "Prepare the dev deployment of the publish function (variant C)",
  "as": "client engineer, cloud beginner",
  "goal": "Know exactly what is needed to deploy the publish function to dev under variant C, and hand the preparation to an AI.",
  "setup": { "mode": "Standard", "variant": "C", "environment": "dev" },
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
| `kind` | `client` (a client's goal on its project) or `app` (a check of DataPass itself across projects) |
| `id` | `J` + digits for client journeys, `A` + digits for app journeys; unique in the repository (`V` + digits is kept for the open questions of app tests) |
| `title` | One line |
| `as` | Who is using DataPass in this journey (role, experience) |
| `goal` | What they want to achieve, in their own words |
| `setup` | Optional: DataPass `mode` (Vanilla, Standard, DataPass, Advanced), `variant`, `environment` to start from |
| `hints` | Optional: where a person would start; never a list of commands |
| `expected` | What should be visible when the goal is reached; each item is judged met / not met / unclear |
| `features` | Tags from [FEATURES.md](FEATURES.md) |
| `outOfScope` | What the tester must not do in this journey |

A journey is data. DataPass never executes anything from it; the tester follows it in the UI.

Examples: [JOURNEYS.md](JOURNEYS.md).
