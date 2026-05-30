# Result: cloud-manager/agile-conch

## Outcome

**SHIPPED** — all six `fix_*.sql` triage patches retired; fresh playbook run lands the postgres secret in Vault with no manual ops.

## Branch

`radiant-axolotl` (cloud-manager deck branch). The `workflow-configuration` repo (separate `hive-deck-pro` deck) carries the Phase 7 wiring on `toasty-lobster`.

## End-to-end verification

- **Run ID:** `run_6NHW5QNR15`
- **Status:** `3` (succeeded)
- **Target:** `prt_SSRB5ENQ3X` — status `3`
- **VM:** `vm_GY68JPPKDN`
- **Playbook:** `pb_PG14JAMMY01`
- **Vault path:** `cloudmanager/playbook-secrets/run_6NHW5QNR15/test-pg/postgres`
  - `db=app`, `host=10.0.150.51`, `port=5432`, `user=appuser`, `password=<32-char generated>`
  - Version 1, created `2026-05-30T06:50:38Z`
- **Manual ops during phase 8:** none — no `psql` UPDATE on playbook content, no `vault policy write`, no `vault write auth/token/roles/...`.

## Phase summary

### Phase 0 — Status check
`hv_status deck=cloud-manager` confirmed all repos on the deck branch with no rogue state.

### Phase 1 — Porch extravars
`vorch-service/internal/ansible/playbook_run_exec.go` — inject `run_public_id` into the extravars JSON, fix `secretsPrefix` to `cloudmanager/data/playbook-secrets/<runID>`. Porch rebuilt and systemd unit restarted.

### Phase 2 — VAULT_ADDR FQDN
Same file — `vaultAddrOrEmpty` now emits the configured `https://ubuntu-server.talkersoft.com:8200`. TLS cert validates cleanly against the FQDN, killing the `127.0.0.1` cert mismatch class.

### Phase 3 — `cloud_playbook_update` content support
Verified the API + MCP `PATCH /api/v1/playbook/{id}` path accepts `content` updates. No new code required — the route was already wired.

### Phase 4 — Playbook revert
Both `pg-14-jammy` and `pg-16-noble` content reverted via `cloud_playbook_update` back to clean `{{ run_public_id }}` templating. All six in-DB triage hacks dropped from the live playbook content.

### Phase 5 — Vault bootstrap policy
`cloud-manager-api/scripts/vault/bootstrap-policies.py` — idempotently writes the `playbook-run` ACL policy, creates the token role, and updates `cloudmanager-playbook-runner` so the orchestrator can read its per-run scope.

### Phase 6 — Seed playbooks
`cloud-manager-mcp/.cicd/export-playbooks.py` + `import-playbooks.py` (idempotent, API-driven). `cloud-manager-api/seed/playbooks/` now holds `pg-14-jammy` and `pg-16-noble` as git source of truth, re-exported after the Phase 4 revert.

### Phase 7 — Bootstrap-worker integration
`workflow-configuration/bootstrap-worker.py` — added Step 6b (Vault policy bootstrap, gated on `VAULT_ROOT_TOKEN`) and Step 6c (seeded playbook import) between Step 6 (MCP build) and Step 7 (verify). `import os` added at top. `python3 -m py_compile` passes. Committed as `9dd8f40` on `toasty-lobster`.

### Phase 8 — E2E
Triggered fresh `cloud_playbook_run` for `pb_PG14JAMMY01` against `vm_GY68JPPKDN`. Polled `vm.playbook_runs` until terminal; final status 3. Vault read of `cloudmanager/playbook-secrets/run_6NHW5QNR15/test-pg/postgres` returned all required fields. Zero manual SQL or Vault CLI ops during the phase.

### Phase 9 — Teardown + rebootstrap
**Skipped.** Marked `[~]` in ORCH.md — destructive rebootstrap deferred to a separate workflow run.

### Phase 10 — Ship
This document + `Retro/LESSONS.md` written; `hv_ship` invoked.

## Pull Requests

| Repo | PR URL |
|------|--------|
| vorch-service | _filled by hv_ship_ |
| cloud-manager-api | _filled by hv_ship_ |
| cloud-manager-mcp | _filled by hv_ship_ |
| planning/workflow-exec | _filled by hv_ship_ |
| workflow-configuration | shipped separately on `hive-deck-pro` deck (commit `9dd8f40`) |
