# Execution: cloud-manager/windward-steamboat

## Objective
The complete Ansible Epic plan series exists and is awaiting human review on one plan-stage PR:
a contracts file every plan conforms to, nine dependency-ordered plans (P1–P9, correct workflow
refs, proportional phases, before/after contract examples), a ledger initialized to "planned",
and a coverage map proving every epic requirement has an owner (or an explicit deferral). No code
changed anywhere.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/windward-steamboat/PLAN.md`
- /home/todd/workspace/hive-deck-pro/planning/workflow-plans/master-plans/ANSIBLE-EPIC.md

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: research current state; write ANSIBLE-EPIC-CONTRACTS.md
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: write plans P1–P5 (backend/data series)
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: write plans P6–P9 (porch + web series)
- [ ] `Tasks/0004-TASK.md` — **Final phase**: ledger + cross-consistency + coverage map; ship stage "plan"; announce PR and STOP

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "windward-steamboat"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
