# Orchestration: hive-deck-pro/gritty-doberman

## Objective
Replace the global `default_target_branch` / per-repo `target_branches` override map with a single `branch:` field at the top of each deck YAML. When done: every repo in a deck uses the same branch for cloning, PR targeting, and feature-branch basing; `hv init` creates the branch on the remote if it doesn't exist yet; `hv ship` ensures it exists before opening a PR; the Go codebase has no per-repo branch override logic.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../oaty-gibbon/PLAN.md` (full design, before/after examples, phase breakdown)

## Task list

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status confirm clean on gritty-doberman
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: Config & resolve — remove Setup fields, add DeckFile.Branch, update resolve.Build, delete BranchFor
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: provision & init — push deck branch after fallback clone; EnsureRemoteBranch in ops/init.go
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: branch & next — createAll uses plan.Branch not git-detected default
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: ship — deck branch as PR base, EnsureRemoteBranch before commitsAhead, fix pre-flight
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: YAML updates — add `branch: hive` to all deck YAMLs, remove dead keys from config.yaml
- [ ] `Tasks/0006-TASK.md` — **Phase 6**: Cleanup & build — delete dead code, go build, make install, write Results + Retro, hv_ship

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## ⚠️ DO NOT USE hv MCP tools to ship this workflow

This workflow fixes the very bug that makes `hv_ship`, `hv_next`, and `hv_status` unreliable against a `hive` target branch. Using those tools during execution will hit the broken behaviour you are fixing.

**Ship manually instead:**

```bash
# For hive-deck-pro repo:
cd /Users/talker/workspace/hive-deck-pro/hive-deck-pro
git add -A && git commit -m "add per-deck branch field, remove global target branch config"
git push origin gritty-doberman
gh pr create --base hive --head gritty-doberman \
  --title "Add per-deck branch field, remove global target branch config" --body ""
gh pr merge --merge

# For workflow-configuration repo:
cd /Users/talker/workspace/hive-deck-pro/workflow-configuration
git add -A && git commit -m "add branch: hive to deck YAMLs, remove global target branch config"
git push origin gritty-doberman
gh pr create --base hive --head gritty-doberman \
  --title "Add branch: hive to deck YAMLs, remove global target branch config" --body ""
gh pr merge --merge

# For workflow-exec (results + retro):
cd /Users/talker/workspace/hive-deck-pro/planning/workflow-exec
git add -A && git commit -m "results: per-deck branch field"
git push origin gritty-doberman
gh pr create --base hive --head gritty-doberman \
  --title "results: per-deck branch field" --body ""
gh pr merge --merge
```

After all PRs are merged, rebuild and install:
```bash
cd /Users/talker/workspace/hive-deck-pro/hive-deck-pro && make install
```

Then use `hv_next` **only after the binary is rebuilt** — the new binary will handle the transition correctly.

## Autonomous execution

```
hv_workflow_run  deck: "hive-deck-pro"  branch: "gritty-doberman"
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
