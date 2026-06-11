# Orchestration: cloud-manager/lucky-engineblock

## Objective
The Marketplace/Blueprint layer is live on this dev box: `marketplace` schema (Blueprint,
BlueprintPlaybook, BlueprintEvent) plus `VirtualMachine.BlueprintId` migrated into clouddb; the
Blueprint and Marketplace controllers serve CRUD, lifecycle, ordered playbook management, listing,
and instantiate behind the `marketplace` feature flag; the BlueprintRunSequencer auto-runs blueprint
playbooks in order after provision with stop-on-failure; `marketplace.ts` MCP tools are built into
the cloud-manager-mcp dist; seeded `jammy` and `postgres-jammy` blueprints instantiate end-to-end on
a live VM; the existing direct attach/apply flow is regression-checked byte-identical; everything is
deployed, and PRs are open via hv_ship.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/lucky-engineblock/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next (branch: `vulnerable-heliotrope`)
- [x] `Tasks/0001-TASK.md` — **Phase 1**: Data model + migration + seeds (entities, prefixes, DbContext, reversible migration, jammy + postgres-jammy seeds, `marketplace` feature flag)
- [x] `Tasks/0002-TASK.md` — **Phase 2**: API — Blueprint + Marketplace controllers (CRUD, lifecycle, ordered playbook management, marketplace listing, instantiate MVP; feature-flag gating; BlueprintEvents)
- [x] `Tasks/0003-TASK.md` — **Phase 3**: Run sequencer (BlueprintRunSequencer hooked to orchestration-completed and run-succeeded writebacks; ordered auto-run, stop-on-failure + events)
- [x] `Tasks/0004-TASK.md` — **Phase 4**: MCP marketplace tools (`marketplace.ts`, profile registration, dist rebuild)
- [x] `Tasks/0005-TASK.md` — **Final phase**: End-to-end verification, deploy, write Results + Retro/LESSONS, then hv_ship

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

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "lucky-engineblock"
```

Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Build requirements
- cloud-manager-api changed → `python3 .cicd/build-cloud-manager.py --target api`
- cloud-manager-mcp changed → `python3 .cicd/build-cloud-manager.py --target mcp`
- Both → `python3 .cicd/build-cloud-manager.py --target all`
(script lives at `cloud-manager-mcp/.cicd/build-cloud-manager.py`; a task is not done until its
build passes cleanly)

## Deploy (Task 0005, BEFORE hv_ship)
1. Apply the EF migration to live clouddb first (see PLAN.md / fragment for the exact
   `dotnet ef database update --connection ...` invocation)
2. `python3 .cicd/deploy-cloud-manager.py --target api`
3. `python3 .cicd/deploy-cloud-manager.py --target mcp`
4. Surface targets, smoke-test attempts + HTTP codes in `Results/RESULT.md`; include the operator
   `/mcp` restart instruction near the top (MCP dist changes)
A failed deploy is a stop condition — FIX file, resolve, retry.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
