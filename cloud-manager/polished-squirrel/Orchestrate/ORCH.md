# Orchestration: cloud-manager/polished-squirrel

## Objective

`hv_ship` with `pr_mode: await_merge` uses an infinite polling loop (no timeout, no two-phase split). After opening PRs, if `open_browser: true` is set in config, each PR URL is opened in the default browser. The `continue_transaction` approach is fully removed.

## Inputs

Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/steamy-kangaroo/PLAN.md`

## Task list

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status hive-deck-pro
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: Remove continue_transaction; restore infinite await_merge loop
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: Add open_browser config flag + browser tab opening
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: Build Go + MCP, rebuild artifacts, ship hive-deck-pro

## Execution steps

1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. Write `Results/RESULT.md` and `Retro/LESSONS.md` BEFORE calling hv_ship

## Autonomous execution

```
/loop Continue executing tasks in /Users/talker/workspace/cloud-manager/planning/workflow-exec/cloud-manager/polished-squirrel/Orchestrate/ORCH.md. Start by running hv_status then hv_next per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
