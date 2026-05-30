# Result: cloud-manager/minty-rooster — mcp coverage Tier 1

> **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up the new dist. The three new tools (`cloud_vm_update`, `cloud_vm_ip_address_update`, `cloud_vm_events`) become available after the reload.

## Outcome

**SHIPPED**

## Branch

`minty-rooster`

## Pull Requests

| Repo | PR | URL |
|------|----|-----|
| cloud-manager-mcp | (filled after `hv_ship`) | TBD |

All other deck repos sit on `minty-rooster` with no code changes and are skipped by `hv_ship`.

## Phase summary

- **Phase 0000 — Setup:** `hv_status` confirmed all 15 deck repos on `minty-rooster` and clean; deck transition was already done in planning.
- **Phase 0001 — Wrapper hardening:** Audited the shared API client (`src/runner.ts` / `src/config.ts`). Existing implementation already enforces camelCase serialization and surfaces 4xx response bodies in tool errors. No code change required — verified via inspection. No FIX file written.
- **Phase 0002 — `cloud_vm_update`:** Added the tool wrapping `PATCH /api/v1/VirtualMachine/{publicId}`. Partial payloads supported; omitted fields treated as "no change". Registered in `src/index.ts` and built clean.
- **Phase 0003 — `cloud_vm_ip_address_update`:** Added the tool wrapping `PATCH /api/v1/VirtualMachine/{publicId}/ip-address`. Required `{ publicId, ipAddress }` input. Registered and built clean.
- **Phase 0004 — `cloud_vm_events`:** Added the tool wrapping `GET /api/v1/VirtualMachine/{publicId}/events?take=100`. Server caps `take` at 500; client passes through. Returns VmEvent DTOs in `occurred_at DESC` order. Registered and built clean.
- **Phase 0005 — Coverage assertion:** Grep-style check confirmed every VM-controller endpoint either has a tool or is documented as excluded (`retry-teardown` — operator UI only; `run/get-secret` — would leak Vault tokens).
- **Phase 0006 — Build + deploy verify (`@cloud-manager-deploy`):** All three tiers gated cleanly. See deploy log below.
- **Phase 0007 — Results + LESSONS + ship:** This document, `Retro/LESSONS.md`, and `hv_ship`.

## Deploy log (Phase 0006)

| Tier | Action | Result |
|------|--------|--------|
| cloud-manager-api | `dotnet build src/CloudManager.sln` | 0 errors (sanity only — no code changes this tier) |
| cloud-manager-web | `npm ci && npm run build` | Clean build (sanity only — no code changes this tier) |
| cloud-manager-mcp | `npm run build` (Phase 2; verified `dist/` newer than `src/`) | `dist/` timestamps 23:57, `src/` 23:13–23:56; dist on disk current |
| cloud-manager-mcp | restart | **DEFERRED** — the running MCP server in the operator's Claude Code session provides the `hv_ship` tool used below. Operator must `/mcp` reload after the PR merges. |

No EF migration was added this workflow; the `dotnet ef database update` step from the deploy fragment was skipped per Task 0006 step 1.

## Source of truth

- Plan: `planning/workflow-plans/cloud-manager/minty-rooster/PLAN.md`
- Exec dir: `planning/workflow-exec/cloud-manager/minty-rooster/`
- mcp source commit: `44a3a3b mcp tier 1: cloud_vm_update + ip_address_update + events`
