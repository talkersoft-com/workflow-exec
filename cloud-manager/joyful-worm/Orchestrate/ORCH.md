# Orchestration: cloud-manager/joyful-worm

## Objective
Update the `install-config` Makefile target so it never silently overwrites files. Default behavior copies only missing files and warns about conflicts. `FORCE=1` restores the old overwrite behavior.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../common/toolkit/hive-deck.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next
- [x] `Tasks/0001-TASK.md` — **Phase 1**: Update Makefile install-config target
- [x] `Tasks/0002-TASK.md` — **Final phase**: write Results + Retro/LESSONS, then hv_ship

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
/loop Continue executing tasks in /Users/talker/workspace/cloud-manager/planning/execution-workflow/cloud-manager/joyful-worm/Orchestrate/ORCH.md. Start by running hv_status then hv_init (or hv_next) per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
