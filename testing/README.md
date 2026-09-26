# Testing DataPass as a user would

DataPass is tested by an outside AI tester (today: Codex). The tester installs the **released
VSIX** into an isolated VS Code and uses it through the UI, the way a person would. There are two
kinds of run, done in this order:

1. **App tests (vsixtest)** cover DataPass itself across several projects: installing; opening,
   closing and switching projects; reloads; themes; errors. The run also answers DataPass's open
   questions.
2. **Client journeys (auto)** are goals one client has on its own project, written in the client's
   own words ("open my project", "compare my variants").

| File | For |
|---|---|
| [CODEX_PROCEDURE.md](CODEX_PROCEDURE.md) | The tester: role, setup, launch, app tests, client journeys, the report, the limits |
| [TEST_FORMAT.md](TEST_FORMAT.md) | Whoever writes test settings: the config (`datapass.codex-tests`) and journeys (`datapass.test-journey`) |
| [FEATURES.md](FEATURES.md) | The fixed list of feature tags that journeys and findings use |
| [JOURNEYS.md](JOURNEYS.md) | Example client journeys J01–J10 |
| [REPORT_FORMAT.md](REPORT_FORMAT.md) | `report.json` and `summary.md` |
| [examples/](examples/) | Complete, valid settings: [client/](examples/client/) (config, J01–J10, a report) and [app/](examples/app/) (config, A01) |

Three repositories take part:

- **This folder** (public) says how testing works. DataPass writes only this folder and the
  extension.
- A **settings repository** holds settings only: a config, `journeys/` and `fixtures/`. The
  client's side writes it (the client's AI or the tester) after reading this folder, starting from
  [examples/](examples/): a vsixtest repository for app tests, the client's **auto repository**
  for client journeys.
- The **audit repository** holds one folder per run, under `reports/app/` or `reports/client/`.

The formats a client prepares for DataPass itself (manifest, graph, options…) are listed in the
[README](../README.md) of this repository.
