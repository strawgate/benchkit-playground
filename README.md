# benchkit-playground

A scratchpad repo for smoke-testing [benchkit](https://github.com/strawgate/benchkit) end-to-end.

Every workflow here exercises a real benchkit feature. Run them, break them, and use the findings to improve benchkit's docs and tests.

## Live dashboard

👉 **https://strawgate.github.io/benchkit-playground/**

## What's being tested

| Workflow | What it exercises |
|---|---|
| `bench-native.yml` | Native JSON format (explicit + auto-detect) |
| `bench-go.yml` | Go `test -bench` format parsing |
| `bench-hyperfine.yml` | Hyperfine JSON format |
| `bench-matrix.yml` | Matrix builds with collision-proof `run-id` |
| `bench-monitor.yml` | `monitor` + `emit-metric` sidecar telemetry |
| `bench-pr-compare.yml` | PR regression detection via `compare` |
| `aggregate.yml` | Incremental aggregation triggered by `bench-data` pushes |
| `pages.yml` | Static dashboard deploy to GitHub Pages |

## How to trigger manually

Any workflow with `workflow_dispatch` can be triggered from the Actions tab.
All `bench-*` workflows also run on every push to `main`.

## Sample benchmark files

`benchmarks/` contains static samples used by some workflows:

- `native-search.json` — search algorithm results in native format
- `go-parser.txt` — synthetic `go test -bench` output
- `hyperfine-commands.json` — synthetic Hyperfine JSON

---

## Bugs & findings

Issues filed against the main benchkit repo based on playground observations:

| # | Issue | Filed |
|---|-------|-------|
| [#184](https://github.com/strawgate/benchkit/issues/184) | `stash` retry loop has no jitter — fails under concurrent load (3 retries exhausted when 4+ workflows push simultaneously) | ✅ |
| [#185](https://github.com/strawgate/benchkit/issues/185) | `GITHUB_TOKEN` pushes don't trigger the `aggregate` workflow — getting-started guide doesn't warn about this | ✅ |
| [#186](https://github.com/strawgate/benchkit/issues/186) | `aggregate` emits misleading "branch does not exist" warning when bench-data is already checked out | ✅ |
| [#187](https://github.com/strawgate/benchkit/issues/187) | Recommended `aggregate.yml` example is missing `workflow_dispatch:` trigger | ✅ |
| [#188](https://github.com/strawgate/benchkit/issues/188) | `@benchkit/chart` is not published to npm — getting-started dashboard section is broken | ✅ |

### What works well ✅

- Native, Go, and Hyperfine format parsing all work correctly
- Matrix build collision-proof naming with custom `run-id` works
- Aggregate builds complete index + series + views on bench-data
- Compare posts clean PR comments with regressions at top; re-runs update (not duplicate) the comment
- Pages dashboard builds from benchkit source and serves live data from bench-data

### Workarounds applied in this repo

- Each `bench-*` workflow explicitly dispatches `aggregate.yml` via `gh workflow run` after stashing (workaround for #185). Requires `permissions: actions: write`.
- `aggregate.yml` checks out `main` (not `bench-data`) so the aggregate action can fetch bench-data into its own worktree (fix for #186).
- `aggregate.yml` has `workflow_dispatch:` so it can be triggered manually (fix for #187).
- `pages.yml` clones the benchkit monorepo and builds packages from source (workaround for #188).
