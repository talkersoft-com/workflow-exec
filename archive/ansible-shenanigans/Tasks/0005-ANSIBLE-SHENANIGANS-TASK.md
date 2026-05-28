# Task: Parameter form web UI

## Task ID
`0005-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Three UI surfaces shipped, all gated by `<FeatureGate flagKey="playbooks">`:
- **`/playbooks`** — list + "Register new" modal
- **`/playbook/:pid`** — detail view (read-only schemas)
- **`VmDetailPage` → Playbooks section** — attached playbooks list + "Attach" modal w/ rjsf form + Apply (stub)

## Context
The web app uses Vite + React + Redux Toolkit. Slices live in `src/store/slices/`. API calls in `src/services/api.ts`. Pages in `src/pages/`. The existing `SnapshotTree` is a good shape reference for a vertically-stacked-card UI section.

## Steps

1. **Deps**: `npm install @rjsf/core @rjsf/utils @rjsf/validator-ajv8 --save`. Commit `package.json` + `package-lock.json`.

2. **API service additions** in `src/services/api.ts`:
   ```ts
   playbooks: {
     list, get(pid), register(body), update(pid, body), refresh(pid), delete(pid),
   },
   assignments: {
     listForVm(vmId), attach(vmId, body), updateVars(vmId, asgnId, body), detach(vmId, asgnId),
   },
   ```

3. **Slices**: `playbooksSlice`, `assignmentsByVmIdSlice`. Same pattern as the existing `snapshotsSlice`.

4. **Routes**: add `/playbooks` and `/playbook/:pid` to the router. Both wrapped in `<FeatureGate>`.

5. **`/playbooks` page**: table (name, git_ref, last_refreshed_at, actions). "Register new" modal with four text inputs. On submit → `api.playbooks.register` → optimistic add to slice → close modal.

6. **`/playbook/:pid` page**: name + git_url + git_ref header; rjsf form rendered read-only from `argument_schema`; `output_schema` rendered as a labeled table.

7. **VmDetailPage "Playbooks" section**: new card after Snapshots. Lists `assignments` for this VM (name, last_run_status, last_applied_at, Apply / Edit / Detach). "Attach playbook" modal: select from registered playbooks → rjsf form rendered live from the selected playbook's schema → submit → `api.assignments.attach`.

8. **Apply button**: stub — `addToast({variant:"info", message:"Execution lands in Phase 0006"})`. No backend call yet.

9. **Build + deploy**: `npm run build`, then `sudo cp -r dist/* /opt/cloud-manager-web/`.

## Acceptance Criteria
- With `playbooks` flag off, no Playbooks link in nav, no section on VM detail.
- With flag on, `/playbooks` renders the registered postgres playbook from Phase 0004.
- Clicking the playbook shows its schema fields.
- On a VM, "Attach playbook" → pick postgres → form renders with `postgresql_version` input pre-populated with `15` (the default from `argument_specs.yml`) → submit → row appears in assignments list with `last_run_status: None`.
- Apply button toasts the stub message and does not call any backend.

## Test
`Test/0005-ANSIBLE-SHENANIGANS-TEST.md`

## On Failure
rjsf rendering quirks are common — write Improvise with the schema fragment that broke. Browser cache serves the old bundle often — hard-reload or check Vite's filename hashing.
