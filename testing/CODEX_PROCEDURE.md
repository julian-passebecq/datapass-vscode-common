# Tester procedure (Codex)

> **Status (2026-09-26).** Checked with Codex on Windows 11 and VS Code 1.139:
>
> - **Verified:** installing the VSIX into an isolated profile (from Codex's sandbox too), and
>   launching the isolated VS Code outside the sandbox. These lines are marked ✔.
> - **Not verified yet:** a whole run inside the Codex desktop app with Computer Use.
>
> When a command fails, write a `blocked` finding with the exact error and stop that step.

There are two kinds of run. Do the **app tests** first, then the **client journeys**.

**One prompt for a whole run (from 1.0.0-rc.1).** An AI session on DataPass's side prepares the run
root, then the person pastes one file into a Codex desktop thread:

```
npx tsx scripts/qa/prepare.ts --auto <auto repository clone> --root <run root> --vsix <file> \
  --sha256 <pinned hash> --datapass-version <version> --commit <released commit> --rc --clone
```

- `--clone` clones the declared folders that are missing; `--sha256` refuses any other VSIX;
  `--datapass-version` tests a newer DataPass with journeys written for an older one.
- `--rc` adds DataPass's release journeys (`qa/rc/journeys/`: installation, Restricted Mode, a
  multi-repository project, Open a Client Project and the palette, a broken project file) to the
  client's journeys, each with its own launch command.
- It writes `<run root>/CODEX_PROMPT.md`: the VSIX path and SHA-256, one launch command per journey,
  the journeys as data, where the report goes, and the rules below. The person only pastes it and
  approves Computer Use for `Code.exe` and the launch commands. Codex then skips steps 1–4 of the
  common setup (already done) and starts at step 5 for each journey.

| | App tests (vsixtest) | Client journeys (auto) |
|---|---|---|
| What is tested | the DataPass VSIX itself, across several projects | DataPass as one client uses it on its project |
| Settings repository | the tester's vsixtest repository | the client's auto repository |
| Config | `purpose: "app"`, several `workspaces[]`, `questions[]` in the app journeys | `purpose: "client"`, one `workspace` |
| Report folder | `reports/app/<run-id>/` in the audit repository | `reports/client/<run-id>/` |
| Formats | [TEST_FORMAT.md](TEST_FORMAT.md), [REPORT_FORMAT.md](REPORT_FORMAT.md) | same |

## Your role

- You are a **user** of DataPass: an engineer at a client company, or a careful first-time user.
  You test through VS Code's UI, the way a person would.
- You never change DataPass's source code, never sign in to a cloud service, never run a cloud CLI
  (`az`, `func`, `databricks`, `fab`) and never deploy anything. Docker is optional.
- You never use the machine's normal VS Code profile, only the isolated one below.
- Findings and answers are claims: write what you saw, with a screenshot or a log line.

## Where you run

- **Host: the Codex desktop app**, in an interactive thread with the Computer Use plugin on.
  - The Codex CLI (`codex exec`) cannot do the UI part: from it, Computer Use sees no app or
    window at all.
  - Shell steps (clone, build, install) work in either host.
- **Computer Use on Windows works only in the foreground.**
  - It takes over the mouse and keyboard of the active desktop, which must stay visible and
    unlocked for the whole run.
  - Nobody can use the PC at the same time. Run when the person is away, or in a Windows virtual
    machine.
- **The person at the PC must approve two things once, in the app:**
  - Computer Use for VS Code (`Code.exe`), with **Always allow**;
  - the launch of VS Code outside the sandbox (step 5).
- **Say it up front.** Start the thread by stating that the VSIX is the user's own build and that
  installing it into the isolated folders is authorized. Otherwise Codex stops to ask before
  installing "software from an unrecognized source". Expect one confirmation turn anyway.

## Common setup (both kinds)

1. **Run root.** Create `%TEMP%\datapass-qa\<run-id>`, where `<run-id>` is
   `<yyyymmdd-hhmm>-<client id>`. Never use a drive root. Everything below happens inside it.
2. **Clone** the settings repository, and every repository its config lists into the `folder`
   each one names. Also clone `https://github.com/julian-passebecq/datapass-vscode` into
   `datapass-vscode`: the preparation helper (and, if needed, the VSIX) come from it.
3. **Get the VSIX** named by `datapass.vsix` (a local path under the run root). From **1.0.0-rc.1**
   on, the release is also a GitHub prerelease with the VSIX attached and its sha256 in the notes —
   download it instead of building, and verify the hash before installing. Releases before
   1.0.0-rc.1 have no tag and no download; build them locally from the **released commit** of
   `datapass.version` (each one listed below).

   Build it in a separate clone, `dp-release`: the `datapass-vscode` clone stays on `main`, where
   the preparation helper lives.

   ```
   # ✔ verified with 0.26.0
   cd <run root>
   git clone datapass-vscode dp-release
   cd dp-release
   git checkout <released commit>
   npm ci
   npm run build
   npx vsce package --out ..\vsix\datapass-vscode-<version>.vsix
   ```

   `npm run build` is needed: without it `vsce` stops with "Extension entrypoint(s) missing".

   | DataPass version | Released commit | Download | VSIX SHA-256 |
   |---|---|---|---|
   | 0.26.0 | `c78f01f` | — (build locally) | — |
   | 1.0.0-rc.1 | `5a4f8d9cb1e160e68893dc546ae7aa3eb3db26d1` | [GitHub prerelease v1.0.0-rc.1](https://github.com/julian-passebecq/datapass-vscode/releases/tag/v1.0.0-rc.1) | `8850b1274c369fb11f1d2c5e776afda30a13a6fd33d0178099023e941d0ac5bd` |
   | 1.0.0-rc.2 | `651957ffba2d4c3ee94f4148c4594c48e0a30cd1` | [GitHub prerelease v1.0.0-rc.2](https://github.com/julian-passebecq/datapass-vscode/releases/tag/v1.0.0-rc.2) | `d06e09d28957eb7ac29e13e199b6b485e3632b4e76807069d0dafb570b57c7a9` |

   For a later release, DataPass's team adds its commit, download and hash to this table.
   `qa:prepare` records the VSIX's sha256 it installs; it must match the table's.

4. **Prepare the run.** The helper does the following:
   - validates the settings;
   - checks each folder's remote;
   - installs the VSIX into an isolated profile and records its sha256;
   - writes the `.code-workspace` file(s) and `run.json`.

   ```
   cd <run root>\datapass-vscode
   npm run qa:prepare -- --auto <run root>\<settings repository folder> --root <run root>
   ```

   - Exit code 0 means ready.
   - Exit code 2 means it cannot prepare, and the reason is printed. Fix your clone (not
     DataPass) or write a `blocked` finding.

   ✔ Verified end to end with 0.26.0: the helper installs into `<root>\.vscode-ext` and
   `<root>\.vscode-user`. Its printed launch line has no `--disable-workspace-trust`: add it (step 5).
   The install it runs is the verified one. By hand it is:

   ```powershell
   # ✔ install into an isolated profile (works from Codex's sandbox too)
   code --user-data-dir "<root>\.vscode-user" --extensions-dir "<root>\.vscode-ext" --install-extension "<root>\vsix\datapass-vscode-<version>.vsix"
   # ✔ check: only DataPass is listed
   code --user-data-dir "<root>\.vscode-user" --extensions-dir "<root>\.vscode-ext" --list-extensions --show-versions
   ```

5. **Launch the isolated VS Code outside the sandbox.**
   - From Codex's sandbox, VS Code installs extensions but cannot open a window: the GPU process
     dies and its storage is read-only.
   - So ask for an escalated (unsandboxed) run for this one command, or use the launch command
     `qa:prepare` printed:

   ```powershell
   # ✔ launch — OUTSIDE the Codex sandbox
   & "$env:LOCALAPPDATA\Programs\Microsoft VS Code\Code.exe" --user-data-dir "<root>\.vscode-user" --extensions-dir "<root>\.vscode-ext" --new-window --disable-workspace-trust "<root>\<workspace>.code-workspace"
   ```

   - `--disable-workspace-trust` makes the run **trusted**. In Restricted Mode DataPass reads files
     but runs neither Git nor commands, so an untrusted run would test the wrong thing. Drop this
     flag only in a journey that tests the first run itself, then click the trust dialog.
   - A fresh profile opens VS Code's Welcome page. DataPass's walkthrough is under
     Help → Welcome.
   - Then use Computer Use on the window "… - Visual Studio Code" and nothing else.

6. **Screenshots.** Computer Use sees the screen but saves no file. Save each screenshot with a
   shell command, run escalated like the launch, for example:

   ```powershell
   Add-Type -AssemblyName System.Windows.Forms, System.Drawing
   $b = [System.Windows.Forms.Screen]::PrimaryScreen.Bounds
   $img = New-Object System.Drawing.Bitmap $b.Width, $b.Height
   [System.Drawing.Graphics]::FromImage($img).CopyFromScreen($b.Location, [System.Drawing.Point]::Empty, $b.Size)
   $img.Save("<report folder>\screens\<journey id>-<what>.png")
   ```

   Name each file `<journey id>-<what>.png`, with lowercase letters, digits and `-` after the id,
   for example `J01-architecture.png`.

**Isolation is not total.** `--user-data-dir` and `--extensions-dir` isolate settings and
extensions, but VS Code still opens a machine-wide store,
`%USERPROFILE%\.vscode-shared\sharedStorage\state.vscdb` (UI state only). Leave it alone: never
delete it.

## App tests (vsixtest)

Goal: find what breaks in DataPass itself, and answer the open questions listed in the app
journeys' `questions[]`.

1. Do the common setup with the vsixtest settings (example:
   [examples/app/](examples/app/)). `qa:prepare` writes one `.code-workspace` per entry of
   `workspaces[]`.
2. Walk each app journey (`kind: "app"`) of the settings repository.
3. Open, close and switch between the workspaces, both in the same window and in a new window.
   Watch for anything carried from one project to another: the selected variant, work orders,
   packs.
4. For each question in a journey's `questions[]`, write one entry in `answers[]` with:
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
   - take a screenshot at each meaningful step into `screens/<journey id>-<what>.png`;
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
- At the end, close the isolated VS Code and delete `<run root>\.vscode-user` and
  `<run root>\.vscode-ext`. Never delete `%USERPROFILE%\.vscode-shared`.
