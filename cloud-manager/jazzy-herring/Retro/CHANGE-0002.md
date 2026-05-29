# CHANGE-0002 — Package carve-out + verifications

## Layout
Repo `vorch-service` reorganized per PLAN.md Final Repo Shape diagram:
- `cmd/vorch-service/main.go` (was `app.go`)
- `cmd/porch/main.go` (stub — Phase 3 will wire it)
- `internal/vm/vm_handlers.go` (was `handlers/vm_handlers.go`)
- `internal/ansible/` — `playbook_run_handler.go`, `playbook_run_exec.go`, `playbook_cancel_handler.go`, `cancel_registry.go`, `collection_install_handler.go`, `collection_remove_handler.go`
- `internal/ansible/healthprobe/probe.go` (was `healthprobe/probe.go`)
- `internal/vmsub/messagesub.go` (was `messagesub/messagesub.go` — package renamed to `vmsub`)
- `internal/ansiblesub/` — `playbook_run_sub.go`, `playbook_cancel_sub.go`, `collection_install_sub.go`, `collection_remove_sub.go` (package renamed to `ansiblesub`)
- `internal/pub/messagepub.go` (was `messagepub/messagepub.go` — package renamed to `pub`)
- `internal/models/command_messages.go`
- `internal/vault/client.go`

Old top-level dirs (`handlers/`, `messagesub/`, `messagepub/`, `models/`, `vault/`, `healthprobe/`) removed.

`go build ./...` and `go vet ./...` both exit 0.

## Deviation — no `internal/amqp/` package
PLAN.md called for an `internal/amqp/` package to hold shared AMQP connection bootstrap. On inspection, the AMQP `Dial` lives directly in `app.go` and the previous `messagesub.go` only holds the `StartConsumer` (VM consumer). There is no shared connection-bootstrap logic worth its own package — each `main.go` will dial directly. Skipping `internal/amqp/` for now; can be added later if porch and vorch both grow non-trivial AMQP setup.

## Assignment-tracking verification (PLAN-required)
**Finding**: Neither `vorch-service/internal/ansible/playbook_run_handler.go` NOR `cloud-manager-api`'s service layer currently writes `vm_playbook_assignments.LastRunId/LastRunStatus/LastAppliedAt`. The Python `cloud-manager-worker` was the sole writer (its `main.py:317-330`). Verified by:

```
grep -rE "vm_playbook_assignments|last_run_id|last_applied_at|LastRunStatus" internal/ansible/ → 0 writes
grep -rE "\.LastRunId\s*=|\.LastRunStatus\s*=|\.LastAppliedAt\s*=" cloud-manager-api/src/Services/CloudManager.Data.Services/ → 1 match (DTO-read only, not write)
```

Vorch's `playbook_run_handler.go:144-146` even has a comment confirming the gap:
```
// Run-level status writeback is target-aggregate; assignment writebacks
// happen in patchTarget per-target. The dedicated run PATCH endpoint
// doesn't exist yet — see FIX-001 for the gap.
```

So after retiring the Python worker, the assignments UI WILL go stale unless we add the writeback. Phase 5 will implement it in **`cloud-manager-api/src/Services/CloudManager.Data.Services/PlaybookRunTargetService.cs`** at the point where it rolls up the run terminal status (the existing place that sets `run.Status = terminal`). Add the `vm_playbook_assignments` UPDATE in the same SaveChanges.

This pushes part of the work into the `cloud-manager-api` PR, not porch — but the funky-vole pattern already established that's where rollups live.
