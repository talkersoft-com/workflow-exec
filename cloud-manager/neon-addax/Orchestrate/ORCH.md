# Orchestration: cloud-manager/neon-addax

## Objective
Implement the full ansible REST surface in `cloud-manager-api` per
`cloud-manager/planning/planning/cloud-manager-ansible/API-PLAN.md`. Lands
one migration (Task 0001) and walks through the 9 phases of API-PLAN (Tasks
0002–0010). Done = every endpoint in API-PLAN is callable, `dotnet build`
clean, smoke tests pass, both PRs (cloud-manager-api + execution-workflow
Results) merged.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../planning/cloud-manager-ansible/API-PLAN.md`
- `../../../planning/cloud-manager-ansible/DATA-MODEL-DELTAS.md`
- `../../../planning/DATA-MODEL.md`

## Task list
- [x] `Tasks/0000-TASK.md` — **Phase 0**: verify workspace + record current feature branch in `../deck.md`
- [x] `Tasks/0001-TASK.md` — **Migration**: `AddAnsibleConstraintsAndTargetTimestamps`
- [x] `Tasks/0002-TASK.md` — **API Phase 1**: AnsibleRole + RoleFile CRUD
- [x] `Tasks/0003-TASK.md` — **API Phase 2**: revision write-side + revision read routes
- [x] `Tasks/0004-TASK.md` — **API Phase 3**: PlaybookRevision auto-create + read routes
- [ ] `Tasks/0005-TASK.md` — **API Phase 4**: PlaybookGlobalRoleRef + ArgumentSpecsTranslator hook
- [ ] `Tasks/0006-TASK.md` — **API Phase 5**: playbook-local roles full surface
- [ ] `Tasks/0007-TASK.md` — **API Phase 6**: PlaybookRunTarget reads
- [ ] `Tasks/0008-TASK.md` — **API Phase 7**: materialization endpoint
- [ ] `Tasks/0009-TASK.md` — **API Phase 8**: multi-VM run-trigger endpoint
- [ ] `Tasks/0010-TASK.md` — **API Phase 9**: run-cancel endpoint + `IPlaybookRunCancelPublisher`
- [ ] `Tasks/0011-TASK.md` — **Final**: write Results + Retro/LESSONS, then hv_ship

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work (write the C# code, run dotnet build/test)
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/execution-workflow/cloud-manager/neon-addax/Orchestrate/ORCH.md. Start by running
hv_status. For each unchecked task: read the task file, do the work, run the matching
Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box.
When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both
BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator
- `dotnet build` errors count as test failures — write FIX, fix, re-run before moving on
- If a service-layer hook (revision auto-create, argument-specs re-derive) isn't running inside an EF transaction, that's a fix, not a "good enough"

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
