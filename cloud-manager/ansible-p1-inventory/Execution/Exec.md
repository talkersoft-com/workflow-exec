# Execution: cloud-manager/ansible-p1-inventory

## Objective
Ansible inventories are first-class database records: seven new entities in the `ansible` schema
(Inventory `inv`, InventoryGroup `invg`, InventoryHost `invh`, InventoryGroupMembership `igm`,
GroupVar `gvar`, HostVar `hvar`, InventoryEvent `invev`) with migration, the epic-wide
`ansible-studio` feature flag seeded (disabled), flag-gated CRUD/var/membership APIs that record
append-only InventoryEvents on every mutation, and MCP tools in a new `inventory.ts` module.
Existing playbook/run/blueprint flows are byte-identical. API + MCP built, migration applied,
deployed, and shipped on one exec-stage PR set.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/ansible-p1-inventory/PLAN.md`
- `../../../workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next; record the execution branch in deck.md
- [x] `Tasks/0001-TASK.md` — **Phase 1**: entities + EF configs + migration + `ansible-studio` flag seed
- [x] `Tasks/0002-TASK.md` — **Phase 2**: DTOs/mappers, services, controllers; cycle check; flag gating
- [x] `Tasks/0003-TASK.md` — **Phase 3**: IInventoryEventService; event writes on every mutation; event list endpoint
- [x] `Tasks/0004-TASK.md` — **Phase 4**: MCP `inventory.ts` module + profile registration + dist rebuild
- [x] `Tasks/0005-TASK.md` — **Phase 5**: regression (existing flows byte-identical) + API smoke + repo docs
- [ ] `Tasks/0006-TASK.md` — **Final phase**: migration → deploy api + mcp → Results + Retro/LESSONS → hv_ship stage "exec"

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "ansible-p1-inventory"
```

Or type in the prompt:

  exec "workflow"

## Improvisation policy
- One FIX file per distinct failure; never silently retry; two failed recoveries → stop and
  surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
