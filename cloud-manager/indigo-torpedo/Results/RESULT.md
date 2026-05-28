# Result: cloud-manager/fearless-heron (indigo-torpedo workflow)

## Outcome
SHIPPED

## Branch
`fearless-heron` (workflow folder name: `indigo-torpedo` — workflow ID, not branch)

## Pull Requests
Filled in after `hv_ship` returns.

| Repo | PR | Status |
|------|----|--------|
| `planning` | TBD — from hv_ship output | merged |
| `execution-workflow` | TBD — from hv_ship output (Results + Lessons) | merged |

All other deck repos skipped (no changes).

## Phase summary

### Phase 0 — Initialize
hv_status confirmed all 14 repos clean on `fearless-heron`. Branch recorded in `deck.md`. Test 0000: 3/3 TCs pass.

### Phase 1 — README.md
Created `cloud-manager/planning/planning/cloud-manager-ansible/` and wrote `README.md` with all 5 required sections (Problem statement, Bundle contents, What's already shipped, Relationship to existing plans, Implementation order). Calls out which sections of `PLAYBOOK-INTEGRATION-PLAN.md` remain authoritative vs. superseded. Test 0001: 3/3 TCs pass.

### Phase 2 — API-PLAN.md
Wrote the full REST surface for ansible management: AnsibleRole CRUD, RoleFile + RoleFileRevision (with auto-revision on PATCH and the ArgumentSpecsTranslator hook for `meta/argument_specs.yml`), PlaybookRevision (auto-created when Playbook content changes), PlaybookGlobalRoleRef attach/detach, PlaybookRole + nested files + revisions, PlaybookRunTarget reads, multi-VM run trigger (`POST /api/v1/playbook/{id}/run`), run cancellation (`POST /api/v1/run/{id}/cancel`), and the materialization endpoint (`GET .../materialized`) returning `files[]` + `secret_inputs[]` + `playbook_public_id` for the worker. Test 0002: 10/10 TCs pass.

### Phase 3 — WEB-PLAN.md
Wrote the clean-separation Ansible UI design: top-level `/ansible/*` section with sub-nav (Playbooks / Roles / Run / History) gated on `feature_flags.playbooks.enabled`; `VmPlaybooks` deleted; new `AnsibleRunPage` (multi-VM trigger), `AnsibleHistoryPage` (with by-playbook/by-VM filters), `AnsibleRunDetailPage` (live-polling + cancel button). Plus `SettingsPage` + `FeatureFlagToggle` always-visible at `/settings` for in-UI flag management. `AttachedVmsPanel` provides the reverse-direction view inside the Ansible section now that `VmPlaybooks` is gone. Monaco chosen as the editor (rationale included). 12-phase rollout. Test 0003: 13/13 TCs pass.

### Phase 4 — VORCH-PLAN.md
Wrote the Go event processor design for `vorch-service`: subscribes to `playbook-runs`, materializes playbook trees from clouddb (not git) via the API materialization endpoint, execs `ansible-runner run --json` as a subprocess. Explicit two-token Vault strategy documented: worker's own env-supplied token reads SSH keys + resolves `x-secret-path` inputs + mints child tokens; per-run child token (scoped to per-(vm, playbook) prefix via the `playbook-run` role) is the one ansible-runner receives so the playbook can write outputs. Cancel via separate `playbook-run-cancel` queue + per-run context cancellation + SIGTERM/SIGKILL. Worker Vault auth: static service token for v1, AppRole as follow-on. `cloud-manager-worker/` Python scaffold marked for deletion at Phase 9. 9-phase rollout. Test 0004: 12/12 TCs pass.

### Phase 5 — DATA-MODEL-DELTAS.md
Audited every column/index need surfaced by the three plans above. Three changes required, grouped into one migration `AddAnsibleConstraintsAndTargetTimestamps`:
1. Unique index on `vm.role_files (role_id, file_path)`
2. Unique index on `vm.playbook_role_files (playbook_role_id, file_path)`
3. Add `started_at` + `completed_at` nullable timestamptz columns on `vm.playbook_run_targets`

Eight other candidates evaluated and explicitly SKIPPED with reasoning (active_revision_id, soft-delete, parent_path, diff_summary, log_path, queued_message_id, worker_id, cancel_requested_at). No new entities, no new EntityPrefixRegistry entries. All changes reversible. Test 0005: 4/4 TCs pass.

### Phase 6 — Ship
Wrote `Results/RESULT.md` and `Retro/LESSONS.md`, then ran `hv_ship`. Single `hv_ship` call produced two PRs simultaneously (planning bundle + execution-workflow Results/Lessons) — expected, by design. All other deck repos skipped.
