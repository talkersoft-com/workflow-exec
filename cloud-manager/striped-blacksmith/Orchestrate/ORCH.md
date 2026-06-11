# Orchestration: cloud-manager/striped-blacksmith

## Objective

The marketplace backend works end-to-end on the live system: Blueprint entities exist with ordered
playbooks and an audit trail; the Blueprint/Marketplace API (feature-flagged `marketplace`)
supports CRUD, publish lifecycle, ordered playbook management, and instantiate; instantiating a
published blueprint creates a provenance-stamped VM, copies its playbooks to assignments,
provisions it through the existing orchestration path, and the BlueprintRunSequencer auto-applies
the chain in order; MCP marketplace tools drive all of it; seeded `jammy` (image-only) and
`postgres-jammy` blueprints prove both the degenerate and the full case on live VMs. Existing
direct attach/apply behavior is unchanged.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/lucky-engineblock/PLAN.md`
- `../../common/toolkit/hive-deck.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status; confirm `striped-blacksmith` across all repos
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: data model — entities, prefixes, DbContext, reversible migration, seeds (jammy, postgres-jammy), `marketplace` feature flag
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: Blueprint + Marketplace controllers/services — CRUD, lifecycle, ordered playbooks, listing, instantiate (MVP: assignments + provision)
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: BlueprintRunSequencer — ordered auto-run on provision-complete / run-succeeded writebacks, stop-on-failure + events
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: MCP marketplace tools + profile registration + dist rebuild
- [ ] `Tasks/0005-TASK.md` — **Final phase**: live e2e verification, migration+deploy, Results/RESULT.md + Retro/LESSONS.md, then hv_ship

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
hv_orchestrate_run  deck: "cloud-manager"  branch: "striped-blacksmith"
```

Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
