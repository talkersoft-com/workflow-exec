# Result: cloud-manager/magical-canary

## Outcome
SHIPPED

## Branch
`magical-canary`

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-api` | TBD — filled from hv_ship | — |
| `cloud-manager-mcp` | TBD — filled from hv_ship | — |
| `cloud-manager-web` | TBD — filled from hv_ship | — |
| `planning/workflow-exec` | TBD — filled from hv_ship | — |

## Phase summary

### Phase 0 — Initialize
Task 0000 verification complete. Ran hv_status for deck "cloud-manager" and confirmed all 15 repos across 6 nodes (ai-infra, os-images, planning, tools, vm-infra, vm-infra/cloud-manager) are on branch magical-canary. 14/15 repos are clean. The single DIRTY repo is planning/workflow-exec, and its dirty state was verified to be EXACTLY the expected single untracked path `?? cloud-manager/magical-canary/` (this workflow's scaffold, to be shipped at Phase 6). No other modified, staged, or untracked paths in workflow-exec. The expected exception is satisfied.

### Phase 1 — API service layer
Implemented Phase 1: added new PagedResult<T> record under CloudManager.DTO.Models, extended IPlaybookRunService with the ListAsync signature (using DtoModel alias + PlaybookRunStatus enum), and added the ListAsync implementation in PlaybookRunService.cs as a sibling to ListByAssignmentAsync. ListByAssignmentAsync was left untouched. Implementation copies the task body verbatim: pageSize clamped to [1,200], page Math.Max(1,page), public_id→Guid translation via GuidByPublicIdAsync helpers, vmId filter joins through PlaybookRunTargets with any-target semantics, batched playbook public_id resolution via ToDictionaryAsync (no N+1), and VaultRunToken scrubbed to null on every emitted item. Build target api exits 0; all five TCs from Test/0001-TEST.md pass.

### Phase 2 — API controller
Phase 2 implemented: added a new [HttpGet] List action to RunController.cs that exposes PlaybookRunService.ListAsync at GET /api/v1/Run with query parameters vmId, playbookId, status, page, pageSize. The pre-existing [HttpGet("{runId}")] Get action was kept intact, and the class-level [RequireFeatureFlag("playbooks")] gate is inherited (no per-action override). Added `using CloudManager.Entities.Enums;` so PlaybookRunStatus is in scope. Build (`build-cloud-manager.py --target api`) succeeded with 0 errors / 9 pre-existing warnings. Deploy (`deploy-cloud-manager.py --target api`) successfully ran dotnet publish, restarted the service, and the new binary is live (service active, binary at /opt/cloud-manager-api/CloudManager.API.dll dated 15:53). The deploy script's built-in smoke test against /api/v1/Host failed with a 404, but that is a pre-existing/unrelated issue (HostController only exposes /api/v1/Host/list) and not caused by this change. The new /api/v1/Run endpoint returns HTTP 200 with the expected envelope shape and no leaked VaultRunToken values.

### Phase 3 — MCP tool
Added the cloud_playbook_run_list MCP tool to cloud-manager-mcp/src/tools/ansible.ts directly below the cloud_run_cancel registration, using the exact code from the task file. The build succeeded (tsc exit 0), and the new tool appears in dist/tools/ansible.js. All TC-001 through TC-004 checks pass.

**Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up the new dist.

### Phase 4 — Web grid
Implemented Phase 4 in full. Extended `api.runs` with a paginated `list(q)` helper above the existing `get`; assembled the query string only for set fields and typed the response with `{ items, page, pageSize, total }`. Added optional `page` / `pageSize` mirror fields to `ansibleRunsSlice`'s `FilterState`. Rewrote `AnsibleHistoryPage`: URL is authoritative via `useSearchParams`, an effect mirrors URL → Redux, a second effect fetches via `api.runs.list`, and the body now renders `RunHistoryFilters` + `VmFilterCombobox` + `RunHistoryGrid` + `Pagination` (Spinner while loading, inline error on failure). `EmptyState` remains only in the FeatureGate fallback (allowed by TC-002). Built three new components with sass modules matching the project's BEM + CSS-variable conventions: `Pagination` (prev / "Page N of M" / next, disables at boundaries), `RunHistoryGrid` (table with status badge, playbook id, started/completed, computed duration; row click + keyboard activation navigate to `/ansible/runs/${id}`), and `VmFilterCombobox` (debounced filter sourced from `api.vms.list()`, clearing the input emits `""` so the parent drops `?vmId=` from the URL).

### Phase 5 — E2E verify
Deployed the web target via the cicd script (build + rsync + restart, came up HTTP 200 after 2s). Ran the three executable E2E checks against the live API at localhost:5250. TC-001, TC-002, and TC-003 all pass. TC-004 (MCP) and TC-005 (browser) are skipped as instructed — they require operator-driven actions outside this agent. Note: the task's psql snippet for picking a VM does NOT filter soft-deleted rows; the top result (vm_GY68JPPKDN) is soft-deleted and correctly returns total=0 from the API (which has a global query filter on VirtualMachines.DeletedAt). Re-ran TC-002 with the top non-deleted VM (vm_31K8J7WQFP) and the API total matches the DB COUNT(DISTINCT run_id).

#### Web deploy output (last lines)
```
[ deploy-cloud-manager ] target=web
[ web ] build + rsync cloud-manager-web → /opt/cloud-manager-web/...
  → npm ci
  → vite build
  → stop cloud-manager-web
  → rsync dist → /opt/cloud-manager-web/
  → start cloud-manager-web
  → waiting (attempt 1/10)
    ✓ up after 2s (HTTP 200)
Deploy complete.
```

#### TC-001 curl
```
$ curl -sS 'http://localhost:5250/api/v1/Run?pageSize=50' | jq '{page, pageSize, total, count: (.items|length)}'
{
  "page": 1,
  "pageSize": 50,
  "total": 50,
  "count": 50
}
```

#### TC-002 VM selection + cross-check
```
$ VMID=$(PGPASSWORD='P@ssw0rd!' psql -h localhost -U cloudmanager -d clouddb -tAc "SELECT v.public_id FROM vm.virtual_machines v JOIN vm.playbook_run_targets t ON t.vm_id = v.id GROUP BY v.public_id ORDER BY COUNT(*) DESC LIMIT 1")
VMID=vm_GY68JPPKDN   # <-- but this VM is soft-deleted (deleted_at=2026-05-30 14:11:51), so the API correctly returns total=0
$ curl -sS "http://localhost:5250/api/v1/Run?vmId=vm_GY68JPPKDN&pageSize=50" | jq '{page,pageSize,total,count:(.items|length),vmId:"vm_GY68JPPKDN"}'
{"page":1,"pageSize":50,"total":0,"count":0,"vmId":"vm_GY68JPPKDN"}
```

The task's psql snippet does NOT exclude soft-deleted VMs. The API's VirtualMachines DbSet has a global query filter (HasQueryFilter e => e.DeletedAt == null) per CloudManagerDbContext.cs:988, so vmPublicId lookup returns null and the endpoint returns an empty envelope — correct behavior.

Re-ran TC-002 against the top NON-deleted VM (vm_31K8J7WQFP):
```
$ curl -sS "http://localhost:5250/api/v1/Run?vmId=vm_31K8J7WQFP&pageSize=50" | jq '{page,pageSize,total,count:(.items|length),vmId:"vm_31K8J7WQFP"}'
{
  "page": 1,
  "pageSize": 50,
  "total": 1,
  "count": 1,
  "vmId": "vm_31K8J7WQFP"
}
$ PGPASSWORD='P@ssw0rd!' psql -h localhost -U cloudmanager -d clouddb -tAc "SELECT COUNT(DISTINCT t.run_id) FROM vm.playbook_run_targets t JOIN vm.virtual_machines v ON v.id = t.vm_id WHERE v.public_id = 'vm_31K8J7WQFP'"
1
```
=> total matches DB cross-check exactly.

#### TC-003 curl
```
$ curl -sS 'http://localhost:5250/api/v1/Run?pageSize=200' | jq '{page,pageSize,total,count:(.items|length),tokensLeaked:([.items[].vaultRunToken]|map(select(.!=null))|length)}'
{
  "page": 1,
  "pageSize": 200,
  "total": 50,
  "count": 50,
  "tokensLeaked": 0
}
```

#### TC-004 SKIPPED
MCP server requires operator-driven `/mcp` reload in Claude Code after Phase 3 rebuilt dist/. Cannot exercise from this agent. Operator must run `/mcp` and then invoke `cloud_playbook_run_list` with pageSize: 5 to verify the envelope shape.

#### TC-005 SKIPPED
No playwright access from this agent (Phase 5 lane). Confirmed web is serving:
```
$ curl -sS -o /dev/null -w "HTTP %{http_code}\n" http://localhost:3000/ansible/history
HTTP 200
```
Operator should open http://localhost:3000/ansible/history in a browser to exercise the five interactions (grid render, Next pagination → ?page=2, VM combobox → ?vmId=vm_XXXX&page=1, row click → /ansible/runs/:runId, browser Back preserves filters).

#### Postgres credential note
cloudmanager DB password is `P@ssw0rd!` (sourced from /home/todd/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/config/cloud-manager-config.env). No ~/.pgpass on the host; use PGPASSWORD env var.

### Phase 6 — Deploy MCP + write results
MCP deploy (`deploy-cloud-manager.py --target mcp`) ran `npm ci` + `tsc build` cleanly. Deploy script intentionally does not restart cloud-manager-mcp — operator must reload via `/mcp` in Claude Code to pick up the new dist. api + web were already deployed during Phases 2 and 5 respectively.
