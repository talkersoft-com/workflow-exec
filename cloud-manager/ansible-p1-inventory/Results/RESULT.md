# Result: cloud-manager/ansible-p1-inventory

**Outcome: SUCCESS** — all 7 tasks complete, all tests passed, migration applied to live
`clouddb`, API + MCP deployed.

> **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up
> the new dist (16 new `cloud_inventory_*` / `cloud_*_var_*` tools).

## Branch

`bladed-cantaloupe` (created by hv_next at execution start, 2026-06-11)

## PR table

| Repo | PR |
|------|----|
| cloud-manager-api | (filled after hv_ship) |
| cloud-manager-mcp | (filled after hv_ship) |
| workflow-exec | (filled after hv_ship) |
| workflow-plans | (filled after hv_ship) |

## Phase summary

| Phase | Task | Result |
|-------|------|--------|
| 0 | Workspace init | `hv_next` → `bladed-cantaloupe` across 18 repos; branch recorded in deck.md |
| 1 | Entities + migration | 7 entities (`inv` `invg` `invh` `igm` `gvar` `hvar` `invev`) in schema `ansible`; EF migration `20260612021711_AddAnsibleInventories` with all unique constraints + filtered unique inventory name; `ansible-studio` flag seeded (enabled=false, healthy=true); prefixes registered, no collisions |
| 2 | Services + controllers | `IInventoryService`/`InventoryService`, DTOs/mapper, `InventoryController` with every PLAN route, `[RequireFeatureFlag("ansible-studio")]`; 29/29 local smoke checks passed (CRUD round-trip, cycle→400, duplicates→409, flag-gated→404, no Guids on the wire) |
| 3 | Events | `IInventoryEventService` (record + timeline only), event write on all 15 mutation sites (best-effort try/catch), `GET api/v1/inventory/{id}/event` newest-first; 13-event sequence verified in order |
| 4 | MCP tools | `inventory.ts` with exactly the 16 planned tools, registered in `hv.cloud`; dist rebuilt |
| 5 | Regression + docs | Old vs new build against the same live DB: 8 pre-existing endpoints byte-identical (`diff -r` clean); zero diffs in cloud-manager-web/vorch-lib/vorch-service; READMEs updated (entities, routes, tools) |
| 6 | Migration + deploy + ship | Migration applied to live `clouddb`; API + MCP deployed; PRs opened |

## Deploy targets + smoke

| Step | Evidence |
|------|----------|
| Migration (live clouddb) | `dotnet ef database update` → applied `20260612021711_AddAnsibleInventories`; 7/7 `ansible` tables present; `ansible-studio` row enabled=f healthy=t |
| API deploy | `deploy-cloud-manager.py --target api` → publish + swap + restart; **up after 4s (attempt 3/10, HTTP 200)** |
| MCP deploy | `deploy-cloud-manager.py --target mcp` → `npm ci` (lockfile stable) + tsc build OK |
| Post-deploy smoke | `GET /api/v1/inventory/list` → 404 `Feature 'ansible-studio' is disabled` (correct: flag ships disabled); `GET /api/v1/virtualmachine/list` → 200 |

## Local test evidence (pre-deploy, scratch DB `clouddb_test`, since dropped)

- Full CRUD smoke: **29 passed, 0 failed** — inventory/group/host/membership/vars round-trip,
  camelCase wire, `inv_`/`invg_`/`invh_`/`gvar_` public ids, 0 raw Guids in responses.
- Event timeline: 13 events for a full mutation sequence, newest first, all 9 contract types
  (`created`, `updated`, `deleted`, `group_added`, `group_removed`, `host_attached`,
  `host_detached`, `var_set`, `var_deleted`); soft-deleted inventory timeline → 404; `deleted`
  event present in DB.
- Regression: with flag disabled, `playbook/list`, `playbook/{id}`, `blueprint/list`,
  `marketplace`, `virtualmachine/list`, `featureflags`, `virtualmachine/{vm}/playbook`,
  `run/{run}/target` byte-identical between the deployed old build and the new build on the
  same data.

## Notes

- The Inputs list referenced `master-plans/ANSIBLE-EPIC-CONTRACTS.md`, which does not exist in
  workflow-plans; PLAN.md §Design (entity table, routes, event types) was used as the binding
  contract. See Retro/LESSONS.md.
- As-built deviation recorded in PLAN.md: wire field for the public id is `id` (repo CRUD DTO
  convention), not `publicId` as the C-INV example sketched; event rows use `publicId`
  (VmEvent DTO convention).
- `ansible-studio` ships **disabled**. Enable via `cloud_feature_flag_update_enabled` (or SQL)
  to use the new routes/tools.
