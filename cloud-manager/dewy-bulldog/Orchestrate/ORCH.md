# Orchestration: cloud-manager/dewy-bulldog

## Objective

Replace `auto_merge`/`require_merged_pr` booleans with a `pr_mode` enum in hive-deck-pro. Add `await_merge` polling op and `hv_await_merge` MCP tool. Update configs on Mac and server. All existing behaviour preserved.

## Inputs

Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/toasty-brownie/PLAN.md`

## Task list

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status verify
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: Config struct — PRMode enum + UnmarshalYAML backward compat
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: ops/ship.go — dispatch on pr_mode
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: ops/await_merge.go — polling loop + contract updates
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: MCP hv_await_merge tool + rebuild
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: Update configs + final tests + Results + Lessons + ship

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
/loop Continue executing tasks in /Users/talker/workspace/cloud-manager/planning/workflow-exec/cloud-manager/dewy-bulldog/Orchestrate/ORCH.md. Start by running hv_status then hv_next per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy

- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)

- `Results/RESULT.md`
- `Retro/LESSONS.md`
