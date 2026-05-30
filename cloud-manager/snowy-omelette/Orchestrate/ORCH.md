# Orchestration: cloud-manager/snowy-omelette

## Objective
Ship `vm.vm_events` (append-only per-VM audit log), wire it into create / destroy / IP-update / vorch-ack code paths, expose `GET /events` timeline and `POST /retry-teardown` endpoints (the latter conditional on `teardown_failed`), drop `cloud_vm_force_delete` from cloud-manager-mcp, and replace the cloud-manager-web force-delete button with a conditional `Retry teardown` action plus a VM activity timeline view. Done means: all phases verified end-to-end, force-delete is fully gone, the timeline view renders real events, and the retry button only appears when it should.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/flaring-balloon/PLAN.md`
- `../../../workflow-plans/cloud-manager/flaring-balloon/deck.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status; confirm `snowy-omelette` across all repos
- [x] `Tasks/0001-TASK.md` — **Phase 1**: `AddVmEvents` migration, `VmEvent` entity, DbContext mapping, public_id wiring
- [x] `Tasks/0002-TASK.md` — **Phase 2**: `IVmEventService` / `VmEventService` (`RecordEventAsync`, `GetTimelineByVmPublicIdAsync`)
- [x] `Tasks/0003-TASK.md` — **Phase 3**: wire `created`, `destroy_requested`, `ip_address_updated` into existing `VirtualMachineService` methods
- [x] `Tasks/0004-TASK.md` — **Phase 4**: OrchestrationCompletionQueue consumer emits `teardown_succeeded` / `teardown_failed`
- [x] `Tasks/0005-TASK.md` — **Phase 5**: `RetryTeardownAsync` + `POST /api/v1/vm/{publicId}/retry-teardown` (400 unless last terminal event is `teardown_failed`)
- [x] `Tasks/0006-TASK.md` — **Phase 6**: `GET /api/v1/vm/{publicId}/events?take=100`
- [x] `Tasks/0007-TASK.md` — **Phase 7**: remove force-delete endpoint + `IVirtualMachineService.ForceDelete…` + service method from cloud-manager-api
- [x] `Tasks/0008-TASK.md` — **Phase 8**: remove `cloud_vm_force_delete` tool registration + handler from cloud-manager-mcp
- [ ] `Tasks/0009-TASK.md` — **Phase 9**: cloud-manager-web — VM activity timeline view on VM detail page
- [ ] `Tasks/0010-TASK.md` — **Phase 10**: cloud-manager-web — replace force-delete button with conditional `Retry teardown` action
- [ ] `Tasks/0011-TASK.md` — **Phase 11**: E2E verify — create → destroy → observe terminal event; trigger retry on failed teardown; observe follow-up event
- [ ] `Tasks/0012-TASK.md` — **Phase 12**: write `Results/RESULT.md` + `Retro/LESSONS.md`; `hv_ship`

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

**Option A — MCP (preferred):** Call the following tool:
```
hv_workflow_run  deck: "cloud-manager"  branch: "snowy-omelette"
```

**Option B — manual:** Type the following command (do not copy-paste):

    workflow execute

⚠️  DO NOT COPY PASTE — type this command manually. The `workflow` keyword only works when typed directly into the prompt.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
