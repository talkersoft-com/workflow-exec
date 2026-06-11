# Result: cloud-manager/copper-kiosk

## Outcome
SHIPPED

## Branch
`stringy-nymph` (orchestration id: `copper-kiosk`)

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-web` | _pending hv_ship_ | |
| `workflow-plans` | _pending hv_ship_ | |
| `workflow-exec` | _pending hv_ship_ | |

## What shipped (cloud-manager-web ONLY — zero backend diffs)
- **Plumbing** — `api.ts` blueprint CRUD/lifecycle/ordered-playbook + marketplace list/instantiate
  sections (camelCase, public ids, shared error envelope); `blueprintsSlice` + `marketplaceSlice`
  modeled on `vmsSlice`; routes `/marketplace`, `/blueprints`, `/blueprints/:id` + nav items
  (Marketplace/Blueprints as peers of Virtual Machines), all gated on the `marketplace` flag via
  the exact `playbooks`/FeatureGate pattern.
- **Marketplace storefront** — cards over `GET /api/v1/marketplace` (OS badge from vmi config,
  playbook count with expandable names, RAM/disk chips); modal instantiate flow (host select,
  wizard-identical name validation, defaults pre-filled/editable) → POST instantiate → redirect
  to `/virtualmachine/{id}`; API errors surfaced inline in the modal.
- **Blueprint builder** — `/blueprints` list with Draft/Published/Archived badges + create modal
  (image picker, wizard option sets); `/blueprints/:id` composer: metadata edit, ordered playbook
  attach/up-down-reorder/detach, per-playbook varsOverride JSON editor (client-side JSON
  validation), Publish/Archive lifecycle, Delete surfacing the API's 409 reason on Published.
- **VM provenance + live run-chain panel** — `BlueprintRunChain` component on VM detail (behind
  FeatureGate): provenance from the `blueprint_instantiated` VmEvent (the VM DTO carries no
  blueprintId), ordered steps from blueprint detail + instantiate-time runPlan, live status from
  `GET /run?vmId=` (FIX-001), failure marks "Chain stopped here" + deep-link to
  `/ansible/runs/{runId}`. Live via notificationHub messages + a bounded poll that stops at
  terminal state (the hub has no run-status broadcasts — see findings).
- **Phase 0005 findings** — `workflow-plans/cloud-manager/copper-kiosk/FINDINGS-vorch-status-surface.md`:
  verdict **sufficient**, no vorch/porch/api changes required, no follow-up plan drafted.

## Deploy
- `deploy-cloud-manager.py --target web` once: lockfile clean, build green, rsync to
  `/opt/cloud-manager-web/`, smoke ✓ up after 2s (HTTP 200) on first attempt. No api/mcp deploys.

## E2E evidence (deployed app at https://ubuntu-server.talkersoft.com, live host host_KW7YKFF3YM)
- **Flag OFF regression**: no Marketplace/Blueprints nav; `/marketplace` → "Marketplace disabled"
  EmptyState (same pattern as `/ansible` with playbooks off); zero console errors. Wizard create
  (`ck-wiz-reg` → vm_CTBP44WQ01): 4-step flow unchanged, provisioned to Running, zero auto-runs,
  no provenance block. Direct attach 201 (same shape) + apply 202 → zero-target no-op run —
  byte-identical to lucky-engineblock's documented pre-feature baseline.
- **Flag ON full loop**:
  - Storefront browses exactly `jammy` ("image only", 2 GB/10 GB) + `postgres-jammy`
    ("1 playbook" → expandable detail names `pg-14-jammy`, 4 GB/20 GB).
  - `jammy` instantiate (`ck-final-jammy` → vm_HA6MY3ABRW): 3 clicks → VM detail → Running;
    provenance "Created from blueprint jammy"; zero chain steps; zero runs.
  - `postgres-jammy` instantiate (`ck-final-pg` → vm_8EF923PTT7): chain panel recorded
    in-browser with NO reload: Pending (t=0) → Queued (t=63s) → **Succeeded** (t=147s) — within
    the backend's known ~2 min envelope (SSH wait + 86s run).
  - Builder round-trip: `ck-scratch` created → Published → Archived → Deleted (404 confirmed);
    seeds untouched.
- **Failure path** (Phase 4, dev server): always-fail fixture blueprint → chain marked Failed +
  "Chain stopped here" + link `/ansible/runs/run_3R5DPA0NS0`; stop-on-failure honored.
- **Earlier phase evidence**: TC sweeps of Tests 0001–0004 all green (flag gating both ways, API
  client shapes against live seeds, duplicate-name 409 surfaced in modal with no orphan VM,
  reorder persistence confirmed by `cloud_blueprint_get`, vars invalid-JSON client rejection +
  valid persistence, provenance absent on admin VM).
- All test VMs destroyed; scratch/fixture blueprints deleted.

## Flag end-state
`marketplace` **disabled** (the task's default — flag was seeded disabled by lucky-engineblock;
re-enable via Settings or `cloud_feature_flag_update_enabled` when the operator wants the
feature visible).

## Known leftovers / environment notes
- Fixture playbook `pb_C3KTN7YD0H` (`ck-always-fail`) could not be deleted: its assignment is
  orphaned behind the soft-deleted TC-003 VM and the API blocks playbook deletion with
  assignments (404 on the assignment endpoint for soft-deleted VMs). Harmless; flagged in
  LESSONS as a backend quirk.
- Transient `WebSocket closed 1006` console errors appear on the deployed app after ~60s idle
  (proxy timeout on the app-level notificationHub connection; auto-reconnect recovers).
  Pre-existing behavior, not introduced by this change.
- `http://localhost:3000` (bare static service) does not proxy `/notificationHub` — the public
  URL does. Pre-existing infra topology.

## Fix files
- FIX-001 — chain panel derived state from `assignment.lastRunStatus`, which the API only writes
  on the run-target writeback, so the Queued phase was invisible; switched the live status source
  to `GET /run?vmId=` with assignments as runPlan-resolution + fallback.

## Phase summary
### Phase 0 — Initialize
hv_next minted `stringy-nymph` across all 18 repos; branch recorded in deck.md; flag enabled.
### Phase 1 — Plumbing
api.ts + slices + gated routes/nav; build green; flag gating verified both ways with Playwright;
live API spot-checks (camelCase, public ids, seeds).
### Phase 2 — Storefront
Cards + modal instantiate; jammy instantiated to provisioning VM in 3 clicks; 409 duplicate-name
surfaced in-modal; no orphan VM.
### Phase 3 — Builder
Compose/attach/reorder/vars/lifecycle all UI-driven and confirmed server-side via MCP.
### Phase 4 — Provenance + chain
FIX-001 found by live test; chain live to Succeeded with no reload; failure path + admin-VM
absence verified.
### Phase 5 — Assessment
Findings doc: status surface sufficient; zero diffs in the four protected repos verified.
### Phase 6 — Deploy + E2E + ship
See Deploy and E2E evidence above; flag left disabled; shipped via hv_ship.
