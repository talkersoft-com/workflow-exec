# Orchestration: cloud-manager/dapper-cannoli

## Objective
Comment out `Bash(*)` from `config.yaml.example` in hive-deck-pro, deploy it locally via `make install-config FORCE=1`, and reinitialize the cloud-manager deck so all repos lose unrestricted Bash access in their Claude Code settings.

## Inputs
- `../init.md`
- `../deck.md`

## Task list
- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: Comment out Bash(*) in config.yaml.example, commit to hive-deck-pro
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: Deploy via make install-config FORCE=1
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: Reinit cloud-manager deck, verify settings
- [ ] `Tasks/0004-TASK.md` — **Final phase**: write Results + Retro/LESSONS, then hv_ship

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
/loop Continue executing tasks in /Users/talker/workspace/cloud-manager/planning/execution-workflow/cloud-manager/dapper-cannoli/Orchestrate/ORCH.md. Start by running hv_status then hv_init (or hv_next) per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
