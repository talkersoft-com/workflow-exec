# Init: cloud-manager/minty-rooster — mcp coverage Tier 1

## What this workflow does

Closes the highest-traffic coverage gap in `cloud-manager-mcp` by adding three new
MCP tools that wrap existing `cloud-manager-api` endpoints on the `VirtualMachine`
controller:

| Tool | Endpoint |
|------|----------|
| `cloud_vm_update` | `PATCH /api/v1/VirtualMachine/{publicId}` |
| `cloud_vm_ip_address_update` | `PATCH /api/v1/VirtualMachine/{publicId}/ip-address` |
| `cloud_vm_events` | `GET /api/v1/VirtualMachine/{publicId}/events?take=100` |

Plus a small hardening pass on the shared API client (camelCase enforcement +
clean 4xx surfacing) gating the new tools.

## Workflow extension

Built on top of `@cloud-manager-feature` (intro + deploy fragments). The intro
fragment's project conventions (camelCase on the wire, public_ids only,
soft-delete semantics, append-only audit log, Vault-token redaction, no vorch
edits) apply throughout. The deploy fragment's build-and-deploy gate runs
before `hv_ship`.

## Tier 1 constraints (HARD)

- **Only 3 new tools.** No fourth tool sneaks in. `cloud_vm_retry_teardown` and
  `cloud_run_get_secret` are explicitly out of scope.
- **No schema changes.** No EF migration. No new column. No new table.
- **No cloud-manager-web changes.** UI stays exactly as it was.
- **No cloud-manager-api changes.** The endpoints already exist; this is a
  pure MCP-surface PR.
- **No vorch / porch changes.**

If a phase looks like it requires anything beyond cloud-manager-mcp source +
build artifacts, stop and surface to the operator.

## Why mcp is built but not redeployed in this workflow

The cloud-manager-mcp dist is loaded by the running MCP server inside the
operator's own Claude Code session — the same session that calls `hv_ship` at
the end of this workflow. Reloading the dist mid-workflow would invalidate the
hive-deck MCP tools we still need to ship the PR.

Per `@cloud-manager-deploy`: the workflow runs `npm run build` so the dist on
disk is correct, commits it, and `Results/RESULT.md` flags the operator to
restart the mcp via `/mcp` after the PR merges. The cloud-manager-api and
cloud-manager-web deploy steps in that fragment are no-ops here (those repos
have no code changes), but the deploy-verify task still confirms both services
are alive before `hv_ship`.

## Inputs

- `../../../workflow-plans/cloud-manager/minty-rooster/PLAN.md`
- `../../../workflow-plans/cloud-manager/minty-rooster/deck.md`
- `../../common/toolkit/hive-deck.md`
- `@cloud-manager-feature-intro` (project conventions)
- `@cloud-manager-deploy` (build-and-deploy gate)
