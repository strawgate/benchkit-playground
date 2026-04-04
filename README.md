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

| # | Issue | Status |
|---|-------|--------|
| [#184](https://github.com/strawgate/benchkit/issues/184) | `stash` retry loop has no jitter — 3 retries exhausted in <3s under concurrent load | Fixed in [PR #211](https://github.com/strawgate/benchkit/pull/211) |
| [#185](https://github.com/strawgate/benchkit/issues/185) | `GITHUB_TOKEN` pushes don't trigger aggregate — docs don't warn about this | Fixed in [PR #211](https://github.com/strawgate/benchkit/pull/211) |
| [#186](https://github.com/strawgate/benchkit/issues/186) | `aggregate` emits misleading "branch does not exist" when bench-data is checked out | Fixed in [PR #211](https://github.com/strawgate/benchkit/pull/211) |
| [#187](https://github.com/strawgate/benchkit/issues/187) | Recommended `aggregate.yml` example is missing `workflow_dispatch:` trigger | Fixed in [PR #211](https://github.com/strawgate/benchkit/pull/211) |
| [#188](https://github.com/strawgate/benchkit/issues/188) | `@benchkit/chart` not published to npm — getting-started dashboard section broken | Open |
| [#200](https://github.com/strawgate/benchkit/issues/200) | `stash` format description only lists 4 of 8 valid formats — `pytest` → opaque error | Fixed in [PR #211](https://github.com/strawgate/benchkit/pull/211) |
| [#201](https://github.com/strawgate/benchkit/issues/201) | `stash` `monitor` input is confusing — unrelated to the `monitor` sidecar action | Open |
| [#204](https://github.com/strawgate/benchkit/issues/204) | `monitor`: no readiness probe after spawn — `emit-metric` hits ECONNREFUSED | Fixed in [PR #211](https://github.com/strawgate/benchkit/pull/211) |
| [#206](https://github.com/strawgate/benchkit/issues/206) | `stash` silently succeeds with 0 benchmarks parsed (Go/Rust non-matching input) | Fixed in [PR #211](https://github.com/strawgate/benchkit/pull/211) |
| [#207](https://github.com/strawgate/benchkit/issues/207) | `compare` shows "No comparable baseline data found" when benchmarks renamed/new | Open |
| [#208](https://github.com/strawgate/benchkit/issues/208) | `aggregate` has no push retry — fails when concurrent stash wins the push race | Open |
| [#209](https://github.com/strawgate/benchkit/issues/209) | `compare` PR comment shows raw `refs/pull/N/merge` instead of `PR #N` | Open |
| [#210](https://github.com/strawgate/benchkit/issues/210) | `stash` default `run-id` collides for matrix jobs — `GITHUB_JOB` is same for all variants | Open (docs) |

### What works well ✅

- Native, Go, Hyperfine, Rust, pytest-benchmark, benchmark-action format parsing all work
- Glob merge: multiple files merged into one stash run
- Auto-detect format works for all supported formats
- Matrix build collision-proof naming with custom `run-id` works
- Aggregate builds complete index + series + views on bench-data; idempotent
- Compare posts clean PR comments with regressions at top; re-runs update (not duplicate) the comment
- Compare with no baseline branch degrades gracefully (returns `has-regression: false`)
- `save-data-file: false` (dry-run mode) still outputs `run-id` and parses benchmarks
- `fail-on-regression: true` works correctly — step outcome is `failure` when regression detected
- Pages dashboard builds from benchkit source and serves live data from bench-data

### Workarounds applied in this repo

- Each `bench-*` workflow explicitly dispatches `aggregate.yml` via `gh workflow run` after stashing (workaround for #185). Requires `permissions: actions: write`.
- `aggregate.yml` checks out `main` (not `bench-data`) so the aggregate action can fetch bench-data into its own worktree (fix for #186).
- `aggregate.yml` has `workflow_dispatch:` so it can be triggered manually (fix for #187).
- `pages.yml` clones the benchkit monorepo and builds packages from source (workaround for #188).
- Matrix workflows use `run-id: ${{ github.run_id }}-${{ matrix.os }}-${{ matrix.arch }}` to avoid run-id collision (#210).

