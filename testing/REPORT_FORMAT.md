# The run report

Each run writes one folder in the audit repository:

```
reports/app/<run-id>/        App tests
reports/client/<run-id>/     Client journeys
  report.json                the report (format datapass.qa-report)
  summary.md                 one page for a person
  screens/                   screenshots named <journey>-<nn>.png
```

`<run-id>` is `<yyyymmdd-hhmm>-<client id>`, for example `20260927-0930-codex-wind-lab`.

## `report.json` (`datapass.qa-report`, version 1)

```json
{
  "format": "datapass.qa-report", "version": 1, "purpose": "client",
  "runId": "20260927-0930-codex-wind-lab",
  "datapass": { "version": "0.26.0", "sha256": "…" },
  "vscode": "1.105.0", "os": "Windows 11",
  "client": { "id": "codex-wind-lab", "bridge": { "folder": "codex-datapass-bridge", "commit": "…" },
              "repositories": [{ "folder": "wind-study-2d", "commit": "…" }] },
  "agent": { "tool": "codex", "model": "…", "host": "app" },
  "journeys": [
    { "id": "J01", "outcome": "reached", "minutes": 6,
      "path": ["Ran Open a Client Project with the bridge URL", "Opened the Architecture panel"],
      "expected": [{ "text": "The workspace opens with the bridge first…", "met": true }],
      "screens": ["screens/J01-01.png"] }
  ],
  "findings": [
    { "id": "F01", "journey": "J02", "severity": "major", "area": "costs",
      "title": "The learning-only line is added to the production total",
      "steps": ["…"], "expected": "…", "actual": "…", "screens": ["screens/J02-03.png"], "suggestion": "…" }
  ],
  "answers": [
    { "question": "V04", "answer": "The selected variant survives a reload, per project.", "evidence": "screens/V04-02.png", "confidence": "high" }
  ],
  "clientFeedback": ["QUESTIONS.md: the difference between shared and learning-only was unclear"],
  "coverage": { "listed": ["onboarding", "variants"], "reached": ["onboarding"] }
}
```

| Field | Meaning |
|---|---|
| `purpose` | `app` or `client`, as in the config |
| `runId`, `datapass`, `vscode`, `os`, `client` | The run context, copied from `run.json` (written by `qa:prepare`): DataPass version and the VSIX's sha256, each repository's commit |
| `agent` | `tool` (codex), `model`, `host` (`app` or `terminal`) |
| `journeys[]` | Per journey: `outcome` = `reached` · `partly` · `not-reached` · `blocked`; `minutes`; `path` (what the tester did, in short steps); `expected[]` with `met` = `true` · `false` · `"unclear"`; `screens[]` |
| `findings[]` | What went wrong or could be better: `severity` = `blocker` · `major` · `minor` · `idea`; `area` = a tag from [FEATURES.md](FEATURES.md); `steps`, `expected`, `actual`, `screens`, `suggestion` |
| `answers[]` | App tests only: one per open question: `question` (its id), `answer`, `evidence` (a screen, a log line, a file), `confidence` = `high` · `medium` · `low` |
| `clientFeedback[]` | Client journeys: the doc gaps the client's AI wrote in the bridge's `QUESTIONS.md` |
| `coverage` | Feature tags listed by the journeys, and those actually reached |

Findings and answers are **claims** by the tester. DataPass's team verifies each one before acting
on it.

## `summary.md`

One page: the run context in two lines, a table of journeys (id, outcome, minutes), the blocker and
major findings with their first screen, the answers (app tests), and the coverage line.
