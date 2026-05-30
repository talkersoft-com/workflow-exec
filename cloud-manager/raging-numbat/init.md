# vm-events — per-VM audit log + retry-teardown

## What this workflow does

Stands up `vm.vm_events` as cloud-manager's per-VM audit log, wires it into the existing create / destroy / IP-update / vorch-ack paths, exposes a `GET /events` timeline endpoint and a `POST /retry-teardown` action conditional on the VM's last terminal event being `teardown_failed`, and retires force-delete from cloud-manager-api, cloud-manager-mcp, and cloud-manager-web. When complete, operators can see a full lifecycle timeline per VM in the UI and have a precise, idempotent way to re-trigger vorch's libvirt cleanup when it has actually failed — without the all-or-nothing semantics of the old force-delete button.

## Read before starting

- `../../../workflow-plans/cloud-manager/flaring-balloon/PLAN.md` — full design, including the `vm.vm_events` schema, event-type enum, write-path table, and endpoint contracts.
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls for status / ship.
- `Orchestrate/ORCH.md` — full task list and `/loop` directive.
- cloud-manager-api PR #19 (`vm-soft-delete`) — soft-delete is the foundation; this PR depends on `deleted_at` being authoritative and on the EF query filter being in place.
- `cloud-manager-api/src/Models/CloudManager.Entities/Migrations/20260530005701_AddVmSoftDelete.cs` — pattern for naming and shape of the new `AddVmEvents` migration.
- `cloud-manager-api/src/Services/CloudManager.Data.Services/VirtualMachineService.cs` — `DeleteVirtualMachineAsync` (soft-delete site), `UpdateIpAddressAsync` (IP write-back site), `CreateVirtualMachineAsync` (create site). All three need event recording added.

## Constraints

- **Append-only at the application layer.** `VmEventService` exposes only `RecordEventAsync` + read methods. No Update, no Delete. The `vm.vm_events` row is never mutated after insert.
- **`ON DELETE RESTRICT` on the FK to `vm.virtual_machines`.** Soft-delete means VM rows are never removed; the audit row will always have a parent. Do not change this to CASCADE.
- **No vorch code changes.** Vorch already publishes destroy acks to `OrchestrationCompletionQueue`. The consumer in `CloudManager.AMQP.Consumer` must read those acks and translate them into events — but the ack format and publish path are out of scope.
- **No hard-delete code path.** Do not reintroduce row-delete on the VM table. If a future need for hard-delete emerges, it ships separately with its own design.
- **`cloud_vm_force_delete` MCP tool is removed, not deprecated.** Drop the registration and handler; do not leave a stub.
- **Retry-teardown is operator UI only for v1.** No MCP tool; if scripted retries become common, add the tool in a follow-up PR.
- **400 guard on retry-teardown.** The endpoint must reject requests when the VM's most recent terminal lifecycle event is not `teardown_failed` — no random retries against healthy VMs.
- **camelCase JSON on the wire; public_ids only.** Internal UUIDs never leak in API or MCP surfaces (project standing rule).
