# Orchestration: cloud-manager/indigo-torpedo

## Objective
Produce a five-file ansible-management planning bundle at
`cloud-manager/planning/planning/cloud-manager-ansible/` covering the API,
web UI, Go event processor, and any data-model deltas needed to fully manage
ansible playbooks and roles end-to-end. Plans only — no code changes.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../planning/DATA-MODEL.md`
- `../../../planning/PLAYBOOK-INTEGRATION-PLAN.md` (note: partially stale, see init.md)
- `../../../planning/ANSIBLE-INSTALL-PLAN.md`

## Task list
- [x] `Tasks/0000-TASK.md` — **Phase 0**: verify workspace + record current feature branch in `../deck.md`
- [x] `Tasks/0001-TASK.md` — **Phase 1**: create bundle folder + write `README.md`
- [x] `Tasks/0002-TASK.md` — **Phase 2**: write `API-PLAN.md` (cloud-manager-api)
- [x] `Tasks/0003-TASK.md` — **Phase 3**: write `WEB-PLAN.md` (cloud-manager-web)
- [x] `Tasks/0004-TASK.md` — **Phase 4**: write `VORCH-PLAN.md` (vorch-service + vorch-lib)
- [x] `Tasks/0005-TASK.md` — **Phase 5**: write `DATA-MODEL-DELTAS.md`
- [x] `Tasks/0006-TASK.md` — **Final**: write `Results/RESULT.md` + `Retro/LESSONS.md`, then `hv_ship`

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work (write the markdown plan file)
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/execution-workflow/cloud-manager/indigo-torpedo/Orchestrate/ORCH.md. Start by running
hv_status. For each unchecked task: read the task file, do the work, run the matching
Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box.
When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both
BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
