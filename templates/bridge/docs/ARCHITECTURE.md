# TODO-CLIENT: project title — architecture

<!-- TODO-CLIENT: replace every section. Keep it short; graph.json and options.json are the detail. -->

## What the project does

TODO-CLIENT: one paragraph, in plain words, for someone who has never seen the project.

## Data flow (current architecture)

```text
TODO-CLIENT: input storage ──► processing ──► output storage
                                   ▲
                    infrastructure as code (planned)
```

Each box is a component of `.datapass/graph.json`, with the same name.

## Repositories

| Key in project.json | Repository | What it holds |
|---|---|---|
| `bridge` | this repository | links and DataPass JSON only |
| `pipeline` | TODO-CLIENT | TODO-CLIENT |
| `infra` | planned | TODO-CLIENT |

## Environments

| Id | What it is | Who deploys, with which official tool |
|---|---|---|
| `dev` | TODO-CLIENT | TODO-CLIENT |
| `prod` | TODO-CLIENT | TODO-CLIENT |

## Open decisions

The variants still being compared are in `.datapass/options.json` (current option first). Choosing
one in DataPass is a preview: nothing is tested, activated or deployed by that choice.

## What is not decided or not known

TODO-CLIENT: list what is open, with who decides and when.
