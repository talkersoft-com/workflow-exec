# Orchestration: cloud-manager/phantom-papaya

## Objective
A user can submit the Create VM form on cloud-manager and the VM enters provisioning successfully. The fix is rooted in evidence captured during Phase 1, scoped to the responsible repo(s), and verified by re-running the original reproduction post-deploy.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../common/toolkit/hive-deck.md`
- `../../../workflow-plans/cloud-manager/blizzard-sparrow/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next
- [x] `Tasks/0001-TASK.md` — **Phase 1**: Reproduce the VM-creation failure end-to-end on `ubuntu-server.talkersoft.com`; capture request/response, screenshots, and journals; write `Retro/REPRODUCTION.md`
- [x] `Tasks/0002-TASK.md` — **Phase 2**: Diagnose the failure layer from captured evidence; identify the root cause at file:line; write `Retro/DIAGNOSIS.md`
- [x] `Tasks/0003-TASK.md` — **Phase 3**: Implement the minimal fix in the diagnosed repo(s); typecheck/build clean
- [x] `Tasks/0004-TASK.md` — **Phase 4**: Deploy via `install-*.py` in correct order; re-run reproduction; confirm success; write `Results/RESULT.md`
- [x] `Tasks/0005-TASK.md` — **Phase 5**: Write `Retro/LESSONS.md` and `Results/RESULT.md` (finalize); then `hv_ship`

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/workflow-exec/cloud-manager/phantom-papaya/Orchestrate/ORCH.md. Start by running
hv_status then hv_init (or hv_next) per deck.md. For each unchecked task: read
the task file, do the work, run the matching Test file. On failure write
Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked
write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship —
then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator
- **No fix speculation before Phase 2** — until Phase 1 reproduction artifacts exist, no source code may be edited

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
