# Result: cloud-manager/snowy-omelette

## Outcome
SHIPPED

## Branch
`snowy-omelette`

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| cloud-manager-api | <fill from hv_ship output> | — |
| cloud-manager-mcp | <fill from hv_ship output> | — |
| cloud-manager-web | <fill from hv_ship output> | — |
| workflow-exec | <fill from hv_ship output> | — |

## Phase summary

### Phase 0 — Initialize
hv_status confirmed snowy-omelette across all 15 repos.

### Phase 1 — Schema + entity
AddVmEvents migration created. VmEvent : Audit entity. DbContext mapping with FK ON DELETE RESTRICT to vm.virtual_machines, two indexes (vm_public_id, occurred_at). Build clean.

### Phase 2 — VmEventService
IVmEventService + VmEventService with append-only RecordEventAsync + GetTimelineByVmPublicIdAsync. AddTransient DI registration.

### Phase 3 — Internal writers
CreateVirtualMachineAsync, DeleteVirtualMachineAsync, UpdateIpAddressAsync each record events post-SaveChangesAsync in best-effort try/catch.

### Phase 4 — Ack consumer
OrchestrationCompletionQueue handler in ConsumerService.HandleReceivedMessage emits teardown_succeeded / teardown_failed with full ack body as payload.

### Phase 5 — Retry-teardown
RetryTeardownAsync + POST /api/v1/VirtualMachine/{publicId}/retry-teardown. 400 guard on non-teardown_failed last event.

### Phase 6 — Timeline endpoint
GET /api/v1/VirtualMachine/{publicId}/events?take=100 with server-side cap at 500. VmEventDto + AutoMapper profile.

### Phase 7 — Remove force-delete (API)
ForceDeleteAsync controller action removed. No interface/service force-delete methods existed.

### Phase 8 — Remove cloud_vm_force_delete (MCP)
src/tools/vms.ts tool block removed. npm build clean, lockfile stable.

### Phase 9 — Web timeline view
VmActivityTimeline component (TSX + SCSS), api.vms.getEvents client method, mounted on VM detail page.

### Phase 10 — Web retry-teardown button
Force-delete UI removed (AdvancedVmManagement -49 lines). api.vms.retryTeardown client method. Button conditional on timeline.[0].eventType === "teardown_failed".

### Phase 11 — E2E verify
Three tests passed end-to-end against a live deployment:
- TC-001 happy path: created → destroy_requested → teardown_succeeded all appear in correct order; retry button not visible.
- TC-002 sad path: created → destroy_requested → teardown_failed; retry-teardown POST accepted, retry_teardown_requested then follow-up terminal event appear; retry button visibility tracks last terminal event correctly.
- TC-003 guard: POST /retry-teardown on a healthy VM returns 400 with documented message.

E2E surfaced two prompt/implementation mismatches (recorded in Retro/FIX-001.md): the task prompt referenced REST paths that no longer matched the controller routes (`POST /api/v1/VirtualMachine` vs `POST /api/v1/virtualmachine/create/{HostPublicId}`, list at `/list`, destroy lives on `OrchestrationController` at `DELETE /api/v1/orchestration/destroy/{publicId}`), and `destroy_requested` is written inside the consumer success branch — so it arrives together with the terminal event at ack time, not immediately after the DELETE. Audit feature itself was not broken.

## Verification artifacts
- Build: cloud-manager-api 0 errors; cloud-manager-mcp clean; cloud-manager-web clean.
- E2E: TC-001, TC-002, TC-003 all pass; see Retro/FIX-001.md for prompt-vs-route notes.
- Pending migrations applied at deploy time: AddVmSoftDelete, AddVmEvents.

## Operator follow-ups (post-merge)
- Restart cloud-manager-mcp via /mcp so the new dist (without cloud_vm_force_delete) takes effect in client sessions.
- The three orphan libvirt domains (vm_Z2P33J92PD, vm_3BBYD222DE, vm_HHYR9RRTB9) noted in PLAN.md are still present; cleanup is a separate piece of work.
- Consider moving the `destroy_requested` write to happen synchronously in the destroy endpoint (before publishing AMQP), so the audit shows intent immediately rather than at ack time. (See FIX-001.)
- Update Phase 11 task prompt to use the real routes / request bodies so future re-runs don't waste time re-discovering them.
