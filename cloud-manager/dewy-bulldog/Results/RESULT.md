# Result: cloud-manager/dewy-bulldog

## Outcome
SHIPPED

## Branch
`dewy-bulldog` (planning) / `velvet-ladybug` (hive-deck-pro execution)

## Pull Requests

| Repo | PR | Status |
|------|----|--------|
| `hive-deck-pro` | TBD — filled from hv_ship output | — |
| `workflow-exec` | planning files | — |

## Phase summary

### Phase 0 — Verify
Both decks clean. hive-deck-pro on `velvet-ladybug`.

### Phase 1 — Config struct
Added `PRMode` type + 3 constants. Replaced `AutoMerge bool` + `RequireMergedPR bool` with `PRMode PRMode` + `MergePollInterval` + `MergePollTimeout`. Added `UnmarshalYAML` on `ShipConfig` for backward compat — old `auto_merge: true` maps to `PRModeAutoMerge`. Updated `ops/next.go` and `ops/teardown.go` to derive `RequireMergedPR` from `PRMode` (only `manual` gates on merged PR).

### Phase 2 — ops/ship.go dispatch
Replaced boolean conditionals with `switch l.Setup.Ship.PRMode`. `auto_merge` → auto-merge flag + checkout. `await_merge` → open PRs, return immediately. `manual` → open PRs, merge-gate message.

### Phase 3 — ops/await_merge.go
New `RunAwaitMerge` — polls open PRs every interval (default 30s), calls checkout on completion, times out after 30m. Wired into dispatch. `AwaitMergeInput` lives in `await_merge.go`. CONTRACT.md and schema.json updated.

### Phase 4 — MCP hv_await_merge
New `await_merge.ts` with BACKGROUND TASK descriptor. Registered in index.ts as `"await-merge"`. MCP rebuilt clean.

### Phase 5 — Configs + ship
`~/.hv/config.yaml` updated to `pr_mode: auto_merge`. All tests pass. Both ships complete.
