# Lessons: hive-deck-pro/gritty-doberman

## What would have helped
1. The earlier partial fix attempt (swirly-baklava branch) left stale `resolve.BranchFor` calls in `branch.go`, `ship.go`, and `ops/init.go` that caused immediate compile failures — a clean revert before starting would have saved one debug cycle.
2. The `validate.go` check requiring `default_target_branch` wasn't caught during planning — any field removal needs a grep for validation/requirement checks, not just struct/caller sweeps.
3. The ORCH.md manual ship commands referenced `gritty-doberman` as the head branch but actual work landed on `wispy-paprika` — workflow scaffolding should use a placeholder like `<execution-branch>` rather than hardcoding the planning branch name.

## Fix files written
None — all issues resolved inline without needing a FIX file.
