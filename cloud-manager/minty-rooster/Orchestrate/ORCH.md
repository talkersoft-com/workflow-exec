# Orchestration: cloud-manager/minty-rooster — mcp coverage Tier 1

## Objective

Ship three new MCP tools in cloud-manager-mcp (`cloud_vm_update`,
`cloud_vm_ip_address_update`, `cloud_vm_events`) plus minor hardening of the
shared API client. No api / web / vorch changes. cloud-manager-mcp dist is
built and committed; operator reloads via `/mcp` after PR merges.

## Inputs

Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/minty-rooster/PLAN.md`
- `../../common/toolkit/hive-deck.md`
- `@cloud-manager-feature-intro`
- `@cloud-manager-deploy`

## Task list

Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status, confirm `minty-rooster` across all 15 repos
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: audit + harden shared API client wrapper (camelCase enforcement, 4xx surfacing)
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: implement `cloud_vm_update`
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: implement `cloud_vm_ip_address_update`
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: implement `cloud_vm_events`
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: coverage assertion — every VM-controller endpoint either has a tool or is explicitly excluded
- [ ] `Tasks/0006-TASK.md` — **Phase 6**: build + deploy verify per `@cloud-manager-deploy` (api/web smoke tests, mcp dist built but NOT reloaded)
- [ ] `Tasks/0007-TASK.md` — **Final phase**: write `Results/RESULT.md` + `Retro/LESSONS.md`, then `hv_ship`

## Execution steps

1. Read all inputs above before starting Task 0000.
2. For each unchecked task in order:
   a. Read the task file.
   b. Do the work.
   c. Run the matching Test file.
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test.
   e. On pass: check the box, move to the next task.
3. When every box is checked, the workflow is complete.

## Autonomous execution

**Option A — MCP (preferred):** Call the following tool:
hv_workflow_run  deck: "cloud-manager"  branch: "minty-rooster"

**Option B — manual:** Type the following command (do not copy-paste):
workflow execute

⚠️  DO NOT COPY PASTE — type this command manually.

## Improvisation policy

- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, …).
- Never silently retry — write the FIX file first, then apply the fix.
- If a failure cannot be recovered after two attempts: stop and surface to operator.

## End-of-workflow outputs (write BEFORE hv_ship)

- `Results/RESULT.md` — include the operator `/mcp` restart instruction near the top
- `Retro/LESSONS.md`
