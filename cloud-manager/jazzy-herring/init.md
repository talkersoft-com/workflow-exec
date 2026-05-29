# porch — execution workflow

## What this workflow does
Carves the ansible orchestration responsibilities out of `vorch-service` into a new sibling binary called **porch**. Both binaries ship from the same Go module (`vorch-service` repo) but cleanly separated under `cmd/vorch-service/` and `cmd/porch/`, sharing only `internal/` plumbing. After this lands, `vorch-service` is libvirt-only, `porch` is ansible-only, and the legacy Python `cloud-manager-worker` (currently a parallel consumer on `playbook-runs`) is fully uninstalled. End state: one consumer on `playbook-runs` (porch), zero Python in the runtime stack, funky-vole's per-run scoped Vault token flow works on 100% of triggers.

## Read before starting
- `deck.md` — deck name, repos in scope, hv MCP calls
- `Orchestrate/ORCH.md` — task list and `/loop` directive
- `../../../workflow-plans/cloud-manager/charmed-panda/PLAN.md` — source plan (authoritative)
- `../../../workflow-fragments/common/toolkit/hive-deck.md` — hv MCP call patterns

## Constraints
- Go is the source of truth. No new Python in the runtime stack.
- camelCase JSON on the wire; public_ids only; internal UUIDs never leaked.
- Vault tokens NEVER appear in logs (regex `hvs\.[A-Za-z0-9_\-]+` → `hvs.<REDACTED>`).
- Vault tokens NEVER appear in run output streams.
- No silent fallback to long-lived cloudmanager token — porch reads `runDTO.vaultRunToken` first; only falls through to vorch's existing `MintChildToken` path when empty.
- Revoke on terminal is idempotent (already enforced on the API side).
- One feature, one PR per affected repo. Expected: `vorch-service`, `cloud-manager-api` (conditional), `workflow-exec`.
- Neither `cmd/vorch-service` nor `cmd/porch` may import the other's domain package (`internal/vm*`, `internal/ansible*`). Verified by `go list -deps`.
- After deployment, exactly ONE consumer on `playbook-runs`. Verified by `rabbitmqctl list_consumers`.
