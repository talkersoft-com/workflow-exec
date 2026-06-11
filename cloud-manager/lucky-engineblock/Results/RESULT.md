# Result: cloud-manager/lucky-engineblock

## Outcome
SHIPPED

> **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up
> the new dist (12 new `cloud_blueprint_*` / `cloud_marketplace_*` tools).

## Branch
`vulnerable-heliotrope` (orchestration id: `lucky-engineblock`)

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-api` | [#27](https://github.com/talkersoft-com/cloud-manager-api/pull/27) | merged |
| `cloud-manager-mcp` | [#24](https://github.com/talkersoft-com/cloud-manager-mcp/pull/24) | merged |
| `workflow-exec` | [#38](https://github.com/talkersoft-com/workflow-exec/pull/38) | merged |

## What shipped
- **marketplace schema** — `Blueprint` (`bp_`, soft-delete, Draft/Published/Archived),
  `BlueprintPlaybook` (`bpp_`, ordered join, unique (blueprint, playbook) and
  (blueprint, position)), `BlueprintEvent` (`bpev_`, append-only), nullable
  `vm.virtual_machines.blueprint_id` provenance FK. One reversible migration
  (`20260611040351_AddMarketplaceBlueprints`) incl. seeds: `jammy` (image-only) and
  `postgres-jammy` (pg-14-jammy at position 0), both Published; `marketplace` feature flag
  seeded disabled.
- **API** — `/api/v1/blueprint` (CRUD, publish/archive lifecycle, ordered playbook
  attach/reorder/detach with atomic renumbering) and `/api/v1/marketplace` (published listing +
  `POST /{id}/instantiate`), all behind `[RequireFeatureFlag("marketplace")]`; BlueprintEvents on
  every state change. Instantiate mirrors the wizard create path exactly (MAC/IP allocation,
  VmEvents, IProvisionService), copies blueprint playbooks to per-VM assignments and returns
  `{virtualMachineId, runPlan}`.
- **BlueprintRunSequencer** — hooks the create-vm Success ack and the run-target terminal
  writeback; derives chain state (never stored); waits in the background for guest SSH (poll :22,
  5 min cap) before starting the chain; advances one playbook per success; stops on failure with
  VmEvent `blueprint_run_chain_failed`; idempotent against duplicate writebacks; never touches
  VMs with `BlueprintId == null`.
- **MCP** — `src/tools/marketplace.ts`: `cloud_blueprint_list/get/create/update/delete`,
  `cloud_blueprint_publish/archive`, `cloud_blueprint_playbook_attach/detach/reorder`,
  `cloud_marketplace_list`, `cloud_marketplace_provision`; registered in `hv.cloud` (86 tools
  total in dist).
- **Deploy script fix** — api smoke test URL corrected to `/api/v1/Host/list` (old URL had no
  route and could never pass).

## Deploy
- Targets run: `--target api` (×3: Phase 2, Phase 3, Phase 5/FIX-003) and `--target mcp` (×1).
- Migration applied to live `clouddb` before the first api deploy.
- Final smoke: api up after 4s (HTTP 200 on /api/v1/Host/list); `/api/v1/health/check` → 200
  `{"status":"Healthy"}`; mcp dist rebuilt clean (no lockfile drift).
- First api deploy failed its smoke test → FIX-002 (stale smoke URL, service was healthy).

## E2E evidence (live, on host_KW7YKFF3YM)
- **jammy** (zero playbooks): instantiated → `vm_NWX5M2N395`, runPlan `[]`, provisioned to
  Completed/Running with IP; zero runs enqueued; `instantiated` BlueprintEvent +
  `blueprint_instantiated` VmEvent recorded.
- **postgres-jammy**: first attempt failed on an SSH readiness race (FIX-003) — which also
  proved stop-on-failure: `blueprint_run_chain_failed` recorded, no further enqueue. After the
  fix: `vm_KYP7P8ZRGS`, runPlan `[asgn_2WPZNENW73]`, sequencer auto-enqueued pg-14-jammy
  ~40s after IP writeback (post-SSH-wait), run `run_2Y75XZ12HQ` **Succeeded** in 86s on the live
  VM (porch logs show Vault password persisted). No manual apply involved.
- **Re-entrancy**: duplicate terminal PATCH on the succeeded target → no second run (count
  stayed 1).
- **Regression** (wizard + direct attach/apply on non-blueprint VM `vm_3BS3SHT1XE`): create 200
  (BlueprintId null), zero auto-runs, attach 201 (same response shape), apply 202 + porch
  consumed the run identically to pre-feature behavior (zero-target no-op — pre-existing apply
  path behavior, untouched by this change; see LESSONS). VM list/get response shapes unchanged.
- **API test sweep** (Phase 2, against deployed API): flag gating (404 disabled / 200 enabled),
  CRUD, lifecycle transitions incl. 409 on deleting Published, duplicate attach 409, atomic
  reorder + renumber-on-detach, soft delete, full blueprint event trail.
- All three test VMs destroyed after evidence capture.

## Phase summary
### Phase 0 — Initialize
hv_next has transitioned the deck; all 18 repos clean on `vulnerable-heliotrope`; branch recorded.
### Phase 1 — Data model + migration + seeds
Entities/prefixes/DbContext + one reversible migration (up/down/up verified on scratch DB); seeds
verified against live data.
### Phase 2 — Blueprint + Marketplace API
Full controller/service/DTO/mapper stack; deployed early (flag-gated, disabled) after FIX-001
(hardcoded port blocks a parallel test instance); tested against the deployed API.
### Phase 3 — Run sequencer
Hooks in ConsumerService (create-vm Success) and PlaybookRunTargetService (terminal writeback);
deployed; live verification merged into Phase 5.
### Phase 4 — MCP marketplace tools
12 tools, profile-registered, dist verified by direct JSON-RPC invocation against the live API.
### Phase 5 — E2E + deploy + ship
See E2E evidence above. FIX-003 (SSH readiness race) found and fixed by the live test.

## Fix files
- FIX-001 — test API instance could not bind (port 5250 hardcoded); resolved by early flag-gated deploy
- FIX-002 — deploy script smoke URL had no route; corrected to /api/v1/Host/list
- FIX-003 — sequencer fired before guest SSH was ready; background readiness wait added
