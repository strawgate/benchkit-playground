# benchkit-playground

End-to-end smoke tests for [benchkit](https://github.com/strawgate/benchkit).

Every workflow exercises a real benchkit feature — format parsing, aggregation, regression detection, monitoring, and dashboards. Run them, break them, and use the findings to improve benchkit.

## Live dashboard

👉 **https://strawgate.github.io/benchkit-playground/**

## Workflows

### Benchmark workflows

These run benchmarks (with synthetic data), stash results, and trigger aggregation.

| Workflow | What it tests | Triggers |
|---|---|---|
| `bench-native.yml` | benchmark-action JSON format (explicit + auto-detect) | push, daily, dispatch |
| `bench-go.yml` | Go `test -bench` format parsing | push, dispatch |
| `bench-hyperfine.yml` | Hyperfine JSON format | push, dispatch |
| `bench-formats.yml` | Rust, pytest-benchmark, benchmark-action, glob merge | push, dispatch |
| `bench-matrix.yml` | Matrix builds with collision-proof `run-id` | push, dispatch |
| `bench-monitor.yml` | `monitor` + `emit-metric` sidecar telemetry | weekly, dispatch |

### Test workflows

These validate specific behaviors and edge cases.

| Workflow | What it tests | Triggers |
|---|---|---|
| `bench-dry-run.yml` | `save-data-file: false` — parse without committing | push, dispatch |
| `bench-fail-regression.yml` | `fail-on-regression: true` detects intentional 3× regression | push, dispatch |
| `bench-pr-compare.yml` | PR regression comments with `compare` action | pull_request |
| `test-aggregate-idempotent.yml` | Running aggregate twice produces no spurious commits | dispatch |
| `test-compare-no-baseline.yml` | Graceful degradation when baseline branch doesn't exist | dispatch |
| `test-phase-mode.yml` | Two-collector phase-decorating monitor mode (7 scenarios) | dispatch |

### Infrastructure workflows

| Workflow | What it does | Triggers |
|---|---|---|
| `aggregate.yml` | Rebuilds indexes + series from raw run files on `bench-data` | bench-data push, dispatch |
| `pages.yml` | Builds and deploys the live dashboard to GitHub Pages | push, bench-data push, dispatch |

## How to trigger manually

All workflows have `workflow_dispatch` and can be triggered from the **Actions** tab or with:

```sh
gh workflow run <workflow>.yml --repo strawgate/benchkit-playground
```

## Sample benchmark files

`benchmarks/` contains static samples used by some workflows:

- `native-search.json` — search algorithm results in benchmark-action format
- `go-parser.txt` — synthetic `go test -bench` output
- `hyperfine-commands.json` — synthetic Hyperfine JSON

---

## Bugs & findings

Issues filed against benchkit based on playground observations:

| # | Issue | Status |
|---|-------|--------|
| [#184](https://github.com/strawgate/benchkit/issues/184) | `stash` retry loop has no jitter — retries exhausted in <3s under concurrent load | ✅ Fixed |
| [#185](https://github.com/strawgate/benchkit/issues/185) | `GITHUB_TOKEN` pushes don't trigger aggregate — docs don't warn about this | ✅ Fixed |
| [#186](https://github.com/strawgate/benchkit/issues/186) | `aggregate` emits misleading "branch does not exist" when bench-data is checked out | ✅ Fixed |
| [#187](https://github.com/strawgate/benchkit/issues/187) | Recommended `aggregate.yml` example is missing `workflow_dispatch:` trigger | ✅ Fixed |
| [#188](https://github.com/strawgate/benchkit/issues/188) | `@benchkit/chart` not published to npm — getting-started dashboard section broken | ✅ Fixed |
| [#200](https://github.com/strawgate/benchkit/issues/200) | `stash` format description only lists 4 of 8 valid formats — `pytest` → opaque error | ✅ Fixed |
| [#201](https://github.com/strawgate/benchkit/issues/201) | `stash` `monitor` input is confusing — unrelated to the `monitor` sidecar action | ✅ Fixed |
| [#204](https://github.com/strawgate/benchkit/issues/204) | `monitor`: no readiness probe after spawn — `emit-metric` hits ECONNREFUSED | ✅ Fixed |
| [#206](https://github.com/strawgate/benchkit/issues/206) | `stash` silently succeeds with 0 benchmarks parsed (Go/Rust non-matching input) | ✅ Fixed |
| [#207](https://github.com/strawgate/benchkit/issues/207) | `compare` shows "No comparable baseline data found" when benchmarks renamed/new | ✅ Fixed |
| [#208](https://github.com/strawgate/benchkit/issues/208) | `aggregate` has no push retry — fails when concurrent stash wins the push race | ✅ Fixed |
| [#209](https://github.com/strawgate/benchkit/issues/209) | `compare` PR comment shows raw `refs/pull/N/merge` instead of `PR #N` | ✅ Fixed |
| [#210](https://github.com/strawgate/benchkit/issues/210) | `stash` default `run-id` collides for matrix jobs — `GITHUB_JOB` is same for all variants | ✅ Fixed |

### What works well ✅

- All supported format parsers: Go, Rust, Hyperfine, pytest-benchmark, benchmark-action, OTLP
- Glob merge: multiple result files combined into one stash run
- Auto-detect format works for all supported formats
- Matrix build collision-proof naming with custom `run-id`
- Aggregate builds complete index + series on bench-data; is idempotent
- Compare posts clean PR comments with regressions; re-runs update (not duplicate) the comment
- Compare with no baseline branch degrades gracefully
- `save-data-file: false` (dry-run mode) still outputs `run-id` and parses benchmarks
- `fail-on-regression: true` correctly fails when regression is detected
- Pages dashboard builds and serves live data
- Monitor sidecar with phase-mode decorates telemetry with benchmark phases

### Workarounds still applied

- Each `bench-*` workflow dispatches `aggregate.yml` via `gh workflow run` after stashing. This is because `GITHUB_TOKEN` pushes don't trigger other workflows. Requires `permissions: actions: write`.
- `pages.yml` clones the benchkit monorepo and builds `@octo11y/core`, `@benchkit/format`, and `@benchkit/chart` from source (workaround until packages are published to npm).
- Matrix workflows use custom `run-id` with the matrix key to avoid file collision.
