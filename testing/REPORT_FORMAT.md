# The run report

Each run writes one folder in the audit repository:

```
reports/app/<run-id>/        App tests
reports/client/<run-id>/     Client journeys
  report.json                the report (format datapass.qa-report)
  summary.md                 one page for a person
  screens/                   screenshots: <journey id>-<what>.png, e.g. J01-architecture.png
```

`<run-id>` is `<yyyymmdd>-<hhmm>-<client id>`, for example `20260927-0930-codex-wind-lab`.

Screenshots are saved by a shell capture (Computer Use saves no file). In the report each one is
written `screens/<journey id>-<what>.png`: lowercase letters, digits and `-` after the journey id.

## `report.json` (`datapass.qa-report`, version 1)

A complete, valid example: [examples/client/report.example.json](examples/client/report.example.json).
Check a report with
`npm run qa:prepare -- --auto <settings repository> --check --report <report.json>`.

```json
{
  "format": "datapass.qa-report", "version": 1, "purpose": "client",
  "runId": "20260927-0930-codex-wind-lab",
  "datapass": { "version": "0.26.0", "sha256": "<sha256 of the VSIX>", "commit": "c78f01f" },
  "vscode": { "version": "1.139.1" },
  "os": { "platform": "win32", "release": "10.0.26200", "arch": "x64" },
  "clients": [{ "id": "codex-wind-lab", "title": "Codex Wind Lab (fictional)",
                "bridge": { "folder": "codex-datapass-bridge", "remote": "https://github.com/…", "commit": "<40 hex>" },
                "repositories": [{ "folder": "wind-study-2d", "remote": "https://github.com/…", "commit": "<40 hex>" }] }],
  "agent": { "tool": "codex", "model": "…", "host": "app" },
  "startedAt": "2026-09-27T09:30:00Z", "finishedAt": "2026-09-27T09:52:00Z",
  "journeys": [
    { "id": "J01", "outcome": "reached", "minutes": 6,
      "path": ["Ran DataPass: Open a Client Project… with the bridge URL", "Opened the Architecture panel"],
      "expected": [{ "text": "The workspace opens with the bridge first…", "met": true }],
      "screens": ["screens/J01-architecture.png"] }
  ],
  "findings": [
    { "id": "F1", "journey": "J02", "severity": "major", "area": "costs",
      "title": "The learning-only line is added to the production total",
      "steps": ["…"], "expected": "…", "actual": "…", "screens": ["screens/J02-costs.png"], "suggestion": "…" }
  ],
  "answers": [],
  "clientFeedback": ["QUESTIONS.md: the difference between shared and learning-only was unclear"],
  "coverage": { "listed": ["onboarding", "costs"], "reached": ["onboarding"] }
}
```

| Field | Meaning |
|---|---|
| `purpose` | `app` or `client`, as in the config |
| `runId` | As above |
| `datapass` | The version, the VSIX's `sha256` (recorded by `qa:prepare` in `run.json`) and optionally the released `commit` |
| `vscode`, `os` | VS Code `version` (and `commit`); OS `platform`, `release`, `arch` |
| `clients[]` | Every project of the run: `id`, `title`, and each clone's `folder`, `remote` and 40-hex `commit` (optional `path`) |
| `agent` | `tool` = `codex`, `model`, `host` = `app` (only the Codex desktop app can drive VS Code) |
| `startedAt`, `finishedAt` | Optional times |
| `journeys[]` | Per journey: `outcome` = `reached` · `partly` · `not-reached` · `blocked`; `minutes`; `path` (what the tester did, in short steps); `expected[]` with `met` = `true` · `false` · `"unclear"`; `screens[]` |
| `findings[]` | `id` (`F` + digits), `journey`, `severity` = `blocker` · `major` · `minor` · `idea`, `area` = a tag from [FEATURES.md](FEATURES.md), `title`, `steps`, `expected`, `actual`, optional `screens` and `suggestion` |
| `answers[]` | One per question of the journeys' `questions[]`: `question`, `answer`, `evidence` (a log line, a file, a description), optional `screens`, `confidence` = `high` · `medium` · `low`. Empty when no journey asks a question |
| `clientFeedback[]` | Client journeys: the doc gaps the client's AI wrote in the bridge's `QUESTIONS.md` |
| `coverage` | Feature tags listed by the journeys, and those actually reached |

Findings and answers are **claims** by the tester. DataPass's team verifies each one before acting
on it.

## `summary.md`

One page:

- the run context, in two lines;
- a table of journeys (id, outcome, minutes);
- the blocker and major findings, each with its first screen;
- the answers (app tests);
- the coverage line.
