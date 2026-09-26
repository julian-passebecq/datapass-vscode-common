# Tester procedure (Codex)

> **Status: design procedure, not yet verified on the real Codex.** The commands below are the
> intended ones. They will be replaced by the commands verified on Windows. Until then, when a
> command fails, write a `blocked` finding with the exact error and stop that step.

There are two kinds of run. Do the **app tests** first, then the **client journeys**.

| | App tests (vsixtest) | Client journeys (auto) |
|---|---|---|
| What is tested | the DataPass VSIX itself, across several projects | DataPass as one client uses it on its project |
| Settings repository | `codex-datapass-vsixtest` (DataPass's own) | the client's auto repository |
| Config | `purpose: "app"`, several `workspaces[]`, `QUESTIONS.md` | `purpose: "client"`, one `workspace` |
| Report folder | `reports/app/<run-id>/` in the audit repository | `reports/client/<run-id>/` |
| Formats | [TEST_FORMAT.md](TEST_FORMAT.md), [REPORT_FORMAT.md](REPORT_FORMAT.md) | same |

## Your role

- You are a **user** of DataPass: an engineer at a client company, or a careful first-time user.
  You test through VS Code's UI, the way a person would.
- You never change DataPass's source code, never sign in to a cloud service, never run a cloud CLI
  (`az`, `func`, `databricks`, `fab`) and never deploy anything. Docker is optional.
- You never use the machine's normal VS Code profile, only the isolated one below.
- Findings and answers are claims: write what you saw, with a screenshot or a log line.

## Common setup (both kinds)

1. **Run root.** Create `%TEMP%\datapass-qa\<run-id>`, where `<run-id>` is
   `<yyyymmdd-hhmm>-<client id>`. Never use a drive root. Everything below happens inside it.
2. **Clone** the settings repository, and every repository its config lists into the `folder`
   each one names. Also clone `https://github.com/julian-passebecq/datapass-vscode` into
   `datapass-vscode`: the preparation helper (and, if needed, the VSIX) come from it.
3. **Get the VSIX** named by `datapass.vsix` (a local path under the run root). There is no
   download and no GitHub release. If the file is not there, build it locally from the
   **released commit** of `datapass.version`. The repository has no tags, so each release's commit
   is listed below.

   ```
   cd <run root>\datapass-vscode
   git checkout <released commit>
   npm ci
   npx vsce package --out ..\vsix\
   ```

   | DataPass version | Released commit |
   |---|---|
   | 0.26.0 | `c78f01f` |

   For a later release, DataPass's team adds its commit to this table. `qa:prepare` records the
   VSIX's sha256.

4. **Prepare the run.** This validates the settings and checks each folder's remote. It installs
   the VSIX into an isolated profile and records its sha256. It writes the `.code-workspace`
   file(s) and `run.json`:

   ```
   cd <run root>\datapass-vscode
   npm run qa:prepare -- --auto <run root>\<settings repository folder> --root <run root>
   ```

   - Exit code 0 means ready.
   - Exit code 2 means it cannot prepare, and the reason is printed. Fix your clone (not
     DataPass) or write a `blocked` finding.

5. **Launch** the isolated VS Code with the command `qa:prepare` printed. It has this shape:

   ```
   code --user-data-dir <run root>\.vscode-user --extensions-dir <run root>\.vscode-ext <run root>\<workspace>.code-workspace
   ```

   Trust the workspace when VS Code asks. If the first-run walkthrough opens, keep it: it is part
   of the test.

## App tests (vsixtest)

Goal: find what breaks in DataPass itself, and answer the open questions in the settings
repository's `QUESTIONS.md`.

1. Do the common setup with the `codex-datapass-vsixtest` settings. `qa:prepare` writes one
   `.code-workspace` per entry of `workspaces[]`.
2. Walk each app journey (`kind: "app"`) of the settings repository.
3. Open, close and switch between the workspaces, both in the same window and in a new window.
   Watch for anything carried from one project to another: the selected variant, work orders,
   packs.
4. For each question in `QUESTIONS.md`, write one entry in `answers[]` with:
   - the answer;
   - the evidence: a screenshot, a log line or a file;
   - your confidence.
5. Throughout the run, keep an eye on the extension host log (Output → Log (Extension Host)). Any
   unhandled error or rejection is a finding.

## Client journeys (auto)

Goal: learn whether a client reaches its goals with DataPass, and where it gets lost.

1. Do the common setup with the client's auto repository.
2. For each journey in `journeys[]`, in order:
   - start from its `setup` (mode, variant, environment), using DataPass's own UI;
   - reach the `goal` as the client would, using `hints` only as a starting point;
   - judge each `expected` item: met, not met or unclear;
   - take a screenshot at each meaningful step into `screens/<journey>-<nn>.png`;
   - stop at the journey's minute limit: the outcome is then `partly` or `not-reached`.
3. **Retry rule.** Before writing the finding for a `not-reached` journey, retry it once from a
   fresh window.
4. Copy the doc gaps from the bridge's `QUESTIONS.md`, if any, into `clientFeedback[]`.

## The report

1. Write `report.json` and `summary.md` as described in [REPORT_FORMAT.md](REPORT_FORMAT.md). They
   go in `reports/app/<run-id>/` or `reports/client/<run-id>/` of your clone of the audit
   repository (`report.remote` in the config). Put the screenshots in `screens/`.
2. Commit on a branch `report/<run-id>`, push it, and open a pull request titled
   `QA <run-id>: <n> not reached, <n> blocker`. **Never merge it**: DataPass's team reviews it.

## Limits

- Stop at the config's `limits.runMinutes` (two hours by default).
- When something blocks the whole run (VS Code does not start, the VSIX does not install), write
  one `blocked` finding with the exact error and stop.
- At the end, close the isolated VS Code and delete `<run root>\.vscode-user`.
