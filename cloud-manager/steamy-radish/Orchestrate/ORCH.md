# Orchestration: cloud-manager/steamy-radish

## Objective
Remove `"*"` from `claude_settings.allow` in the hive-deck-pro source and live user config,
bump the MCP package version to `0.1.1`, push to warm-leviathan, then ship and reinit the
deck so all repos get fresh settings without the wildcard.

## Inputs
- `../init.md`
- `../deck.md`

## Task list
- [ ] `Tasks/0000-TASK.md` — **Phase 0**: verify workspace
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: remove `"*"` from config.yaml.example + bump mcp/package.json
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: remove `"*"` from ~/.hv/config.yaml
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: commit + push hive-deck-pro to warm-leviathan
- [ ] `Tasks/0004-TASK.md` — **Final**: write Results + Retro/LESSONS, hv_ship, hv_init

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
/loop Continue executing tasks in /Users/talker/workspace/cloud-manager/planning/execution-workflow/cloud-manager/steamy-radish/Orchestrate/ORCH.md. Start by running
hv_status. For each unchecked task: read the task file, do the work, run the matching
Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box.
When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both
BEFORE calling hv_ship — then ship, call hv_init, and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
