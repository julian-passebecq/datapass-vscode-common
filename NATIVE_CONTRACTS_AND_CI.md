# Native contracts, native CLIs and metadata-only CI

What DataPass assumes about a client's **native** code, CLIs and contracts, and what CI a bridge may
run. Applies to every client; field details are in the
[field reference](https://github.com/julian-passebecq/datapass-vscode/blob/main/docs/PREPARING_A_PROJECT.md).

## 1. DataPass is contract-agnostic

A client's native contracts (request, receipt and result schemas, CLI names, output layouts) belong to
the native repository and change there, on the client's schedule. DataPass does not implement, validate
or migrate them:

- no DataPass code, test, example or schema in `datapass-vscode`, this repository or the hub targets a
  client's native request/receipt schema, CLI entry point or result path;
- a breaking native revision needs **no DataPass change**; the client updates the bridge files that
  name native paths (`graph.json` `artifacts`, the native `.vscode/tasks.json` test task) in the same
  change, and DataPass then checks the new paths against the disk;
- DataPass never infers a contract that is not implemented from another one (for example a review or
  publication step from a receipt schema);
- old native layouts stay in the client's Git history; DataPass never migrates them silently.

## 2. A native CLI that emits a versioned manifest

When a native CLI (for example a campaign or build compiler) writes a versioned manifest and its outputs,
declare them on the component in `graph.json`; DataPass discovers them as **file references only**:

```jsonc
"artifacts": { "repoRef": "lab", "root": ".",
  "files": [ { "path": "tools/campaign.py", "role": "entry" } ],
  "generated": [ { "path": ".lab/build/manifest.json", "producer": "lab campaign CLI",
                   "how": "run the CLI's build command, then its apply command" } ] }
```

| DataPass does | DataPass never does |
|---|---|
| checks each declared path exists in the named repository and shows *present* or *to generate* (with the producer and `how`) | run the CLI, or any client file, during discovery |
| lets the AI read the file as context when asked | keep private state about the manifest in the extension; everything is in the client's repositories |
| runs the component's own `test` task on an explicit click, with a receipt (field reference §15) | treat a present manifest as a deployment, a preview/apply approval or a cloud check |

Preview and apply stay two separate native commands described in `how`; DataPass shows them, the client
runs them. Replacing an older flow waits for the client's own parity tests.

## 3. Metadata-only CI is allowed

A bridge or auto repository holds configuration only, no native code. A CI workflow there that validates
its JSON against DataPass's **published JSON Schemas** ([`schemas/`](schemas/), pinned to a version) is
allowed and encouraged. It proves metadata shape only: native tests stay in the native repositories, and
a green metadata check is not a native test, a deployment or runtime evidence. DataPass keeps source
revision, CI result and runtime evidence separate.

The same answer, as given to a client, is in the Codex Wind Lab bridge:
[EXPECTATIONS.md › Metadata-only CI](https://github.com/julian-passebecq/codex-datapass-bridge/blob/main/EXPECTATIONS.md#metadata-only-ci-in-the-bridge-or-auto-repository).
