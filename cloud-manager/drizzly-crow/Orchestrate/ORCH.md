# Orchestration: cloud-manager/drizzly-crow

## Objective

All hive-deck config lives in `workflow-configuration` git repo inside the cloud-manager deck. `$HV_HOME` points at it via `~/.hv/.env`. `make configure` writes the env file. `config.LoadSetup()` supports `$HV_HOME/` directly (no `.hv/` append).

## Inputs

Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/silver-pigeon/PLAN.md`

## Task list

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status verify
- [x] `Tasks/0001-TASK.md` — **Phase 1**: HV_HOME direct path in config.LoadSetup()
- [x] `Tasks/0002-TASK.md` — **Phase 2**: Create workflow-configuration repo + seed + add to deck
- [x] `Tasks/0003-TASK.md` — **Phase 3**: make configure script + shell sourcing reminder
- [x] `Tasks/0004-TASK.md` — **Phase 4**: Verify end-to-end + Results + Lessons + ship

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
/loop Continue executing tasks in /Users/talker/workspace/cloud-manager/planning/workflow-exec/cloud-manager/drizzly-crow/Orchestrate/ORCH.md. Start by running hv_status then hv_next per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy

- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)

- `Results/RESULT.md`
- `Retro/LESSONS.md`
