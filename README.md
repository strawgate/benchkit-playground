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

## Findings log

Open issues in this repo document any unexpected behaviors discovered here.
