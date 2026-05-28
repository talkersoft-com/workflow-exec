# Orchestration: cloud-manager/brisk-finchling

## Objective

Add `UpdateHealthAsync` to the feature-flag service layer and expose a `/api/v1/admin/feature-flags/{key}` route alias that accepts `{healthy, last_probed_at}` so vorch-service's health probe can write its result back to the API. No migration needed.

## Inputs

Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager-ansible/hazy-kite/PLAN.md`

## Task list

Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: Service layer — add `UpdateHealthAsync` to interface and implementation
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: Controller — nullable request, route alias, dispatch logic
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: Tests + smoke — unit tests, integration test, manual probe verification; write Results/RESULT.md + Retro/LESSONS.md; hv_ship

## Execution steps

1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked write `Results/RESULT.md` and `Retro/LESSONS.md` BEFORE calling hv_ship

## Autonomous execution

```
/loop Continue executing tasks in /Users/talker/workspace/cloud-manager/planning/workflow-exec/cloud-manager/brisk-finchling/Orchestrate/ORCH.md. Start by running hv_status then hv_next per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy

- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)

- `Results/RESULT.md`
- `Retro/LESSONS.md`
