# How to prepare a client for DataPass

For the AI that prepares a client project (the **client AI**). DataPass is the vendor; you are the
client. You own your code, your facts and your bridge; DataPass delivers the extension, this
repository and answers to your questions.

## 1. Roles

| Who | Does | Does not |
|---|---|---|
| **DataPass** (vendor) | the extension, this repository, the answers to questions, the releases; may write a **first draft** of your bridge | never writes your native code; never maintains your bridge for you |
| **Client AI** (you) | the native code, which works **with or without DataPass**, its tests and CI; the client's facts; the bridge; the auto repository | wait for DataPass to write your code |
| **Bridge repository** | the link repository shared between DataPass and you | contain code |
| **The person** (your user) | feature questions, facts only they know; pastes hand-offs between DataPass and you | — |
| **Tester** (Codex, auto mode) | tests the released extension on your client, as a user would | change your code |

Who maintains which bridge file:

| Bridge file | First draft | Maintained by |
|---|---|---|
| `.datapass/project.json`, `graph.json`, `options.json`, `sheet.json` | you, or DataPass from [`templates/bridge/`](templates/bridge/) | you |
| `AGENTS.md`, `README.md`, `docs/ARCHITECTURE.md` | same | you |
| `DATAPASS_ONBOARDING.md`, `DATAPASS_REQUESTS.md` | DataPass | DataPass (requests only) |
| `QUESTIONS.md` | you | you ask; DataPass answers by improving this repository |

## 2. The cycle

| # | Who | What | When | Why | Where the result is |
|---|---|---|---|---|---|
| 1 | DataPass | onboarding request in the bridge | new client or major release | say what is expected and where to read | `DATAPASS_ONBOARDING.md` |
| 2 | you | native code + tests + CI | before any DataPass file | the code must live without DataPass | your code repositories |
| 3 | you | bridge `.datapass/*.json` | after 2 | describe the architecture | the bridge, valid against [`schemas/`](schemas/) and DataPass's *Check bridge files* |
| 4 | tester or you | auto repository: settings, journeys, fixtures | after 3 | tell the tester what to test | your auto repository ([testing/](testing/README.md)) |
| 5 | tester | tests of the released extension | each release | check as a user | a report in the audit repository ([testing/REPORT_FORMAT.md](testing/REPORT_FORMAT.md)) |
| 6 | you | anything unclear | any time | never guess silently | `QUESTIONS.md` in the bridge |
| 7 | DataPass | triage: product gap → release work; doc gap → this repository; client mistake → request in the bridge | after 5 or 6 | improve the product without touching your code | release notes, this repository, `DATAPASS_REQUESTS.md` |
| 8 | DataPass | new release | when ready | deliver | the extension + [`VERSION`](VERSION) |
| 9 | you | update | after 8 | stay aligned | your bridge and auto repository |

## 3. The hand-off

DataPass hands work to you with a file in the bridge (`DATAPASS_ONBOARDING.md`, later
`DATAPASS_REQUESTS.md`: what to do, where to read, in which order, how to answer) and a short
message the person pastes into your conversation. One PR per repository; merge when its tests pass.

## 4. The rule

When you lack information, DataPass improves this documentation and the hand-off instead of doing
your work. A DataPass draft of a client file is a **proposal** you adopt, change or replace; it is
yours once delivered.

## 5. Simulators versus provider profiles

A local pilot may stand in for a cloud step with an ADF-like pipeline JSON, a Functions-style
handler runnable without the `func` CLI, or a Fabric-item folder — as long as each one is declared
**truthfully**:

- Give the simulating component a **generic, `python` or `docs` profile** (never `adf`,
  `functions` or `fabric`, which claim a deployable native package); a real provider profile
  belongs only to a component whose files that provider's own tooling could actually deploy.
- Mark the cloud target it stands in for with `status: "planned"` in `graph.json`. DataPass's
  readiness reads a `planned` or simulated component as **planned/simulated**, never
  **deployable** — it never claims a step is ready to run in the cloud because a same-shaped local
  file exists.
- Say so once in `docs/ARCHITECTURE.md` or the component's `description`: which files are the
  cloud-native shape (kept for realism or a future migration) and which script actually runs them
  locally.

This is the correct pattern for an early-stage pilot; do not "upgrade" a simulator's profile just
to make readiness look further along.

## 6. What to read

1. [README](README.md): current version and rules.
2. [`templates/bridge/`](templates/bridge/) and its [`EXPECTATIONS.md`](templates/bridge/EXPECTATIONS.md).
3. [`schemas/`](schemas/) and a complete example: [`examples/doc-pipeline/`](examples/doc-pipeline/).
4. The field reference and step-by-step guide in the extension repository:
   [PREPARING_A_PROJECT.md](https://github.com/julian-passebecq/datapass-vscode/blob/main/docs/PREPARING_A_PROJECT.md),
   [docs/guide/](https://github.com/julian-passebecq/datapass-vscode/tree/main/docs/guide), and the
   [ready-to-paste prompt](https://github.com/julian-passebecq/datapass-vscode/blob/main/docs/guide/06_PROMPT_FOR_THE_CLIENT_AI.md).
5. [knowledge/](knowledge/README.md) for tools and MCP servers, [testing/](testing/README.md) for tests.
