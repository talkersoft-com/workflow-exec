# Paginated playbook run history with VM filter

## What this workflow does

Executes the plan in `workflow-plans/cloud-manager/gracious-churro/PLAN.md`. Turns the stubbed `/ansible/history` page into a real paginated browse surface (50 runs per page, ordered StartedAt DESC, filterable by VM) backed by a new `GET /api/v1/Run` list endpoint, a `cloud_playbook_run_list` MCP tool, and a React grid bound to URL-driven filter state. Each row links to `/ansible/runs/:runId` — the same detail view reached by pressing Run, so the live-run experience is unchanged. When complete, an operator can land on `/ansible/history`, narrow to a specific VM via a combobox, page through every run that ever targeted it, and deep-link straight into the per-run output.

## Read before starting

- `../../../workflow-plans/cloud-manager/gracious-churro/PLAN.md` — full design, before/after code blocks for every phase, open-question proposals
- `../../../workflow-plans/cloud-manager/gracious-churro/deck.md` — original deck/repo list (now on branch `magical-canary`)
- `deck.md` — this branch's deck wiring (repos in scope, ship call)
- `Orchestrate/ORCH.md` — the task list and `hv_workflow_run` entrypoint
- `cloud-manager-api/src/Services/CloudManager.Data.Services/PlaybookRunService.cs` lines 105-127 — `ListByAssignmentAsync` is the sibling pattern (DTO scrub of `VaultRunToken` lives there)
- `cloud-manager-api/src/CloudManager.API/Controllers/RunController.cs` — existing per-run endpoints + `[RequireFeatureFlag("playbooks")]` gate to preserve
- `cloud-manager-web/src/pages/AnsibleHistoryPage.tsx` — the EmptyState stub Phase 4 replaces; mentions FIX-001
- `cloud-manager-web/src/store/slices/ansibleRunsSlice.ts` — reserves `vmFilter?`/`playbookFilter?` and notes deep-linkable filters as the eventual shape

## Constraints

- **`VaultRunToken` MUST be null on every list-response item.** Same rule as `ListByAssignmentAsync` (`PlaybookRunService.cs:124`). Only `GetByIdAsync` may surface the real token. Scrubbing belongs in the service layer, not the controller — keeps every caller honest.
- **camelCase JSON on the wire; public_ids only.** Query params accept `vm_XXXX` / `pb_XXXX` strings; never raw UUIDs. The service translates public_id → Guid internally.
- **Feature-flag gate preserved.** `[RequireFeatureFlag("playbooks")]` already decorates the controller; the new action inherits it. Do not weaken the gate.
- **`vmId` filter semantics = "any target matches".** A run is "for VM X" if any of its `playbook_run_targets` rows has `VmId = X`. Multi-target runs surface for every targeted VM. Confirmed with operator up front.
- **Filters live in the URL.** `vmId`, `playbookId`, `status`, `page`, `pageSize` are URL query params; Redux mirrors them. Deep-linkable, bookmarkable. The slice extension is mirror-only; the URL is authoritative.
- **Live run page (`/ansible/runs/:runId`) is unchanged.** Both the Run button and History grid rows route there. No new abstraction for "active vs historical" — the existing detail view already handles both.
- **No N+1 in the list query.** Resolve playbook public_ids in one batched lookup; do not loop `PublicIdByGuidAsync` per row.
- **`@cloud-manager-deploy` post-phase is mandatory.** API hosts a new endpoint, MCP gains a new tool, Web renders a new view — deploy all three before `hv_ship` so the running system reflects the merged code. Operator must `/mcp` reload after the MCP dist rebuilds.
