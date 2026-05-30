# Harden playbook execution path

## What this workflow does

Replaces the six in-DB SQL triage patches (`fix_pg_run_id.sql`, `fix_hvac.sql`, `fix_delegate.sql`, `fix_become.sql`, `fix_tmp.sql`, `fix_tls.sql`) we applied to the postgres playbook this session with proper code fixes across porch, cloud-manager-api, and the playbook content itself. Porch will inject `run_public_id` and use the FQDN Vault URL so playbooks can use the existing per-run-scoped Vault policy without `delegate_to: localhost`, `validate_certs: false`, or `python3-hvac` installed on every VM. A new export/import pair gives playbook bodies a git source-of-truth, and a `bootstrap-policies.py` script provisions the Vault `playbook-run` policy + role automatically. When complete, a fresh dev-box bootstrap via `bootstrap-server.py` ends with a green postgres playbook run end-to-end, no operator intervention.

## Read before starting

- `../../../workflow-plans/cloud-manager/poppy-fiddle/PLAN.md` — full design + before/after code blocks for every change
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and the Claude autonomous-execution block
- `vorch-service/vorch-service/internal/ansible/playbook_run_exec.go` — the file Phase 1 + Phase 2 patch
- `cloud-manager-api/src/Services/CloudManager.Data.Services/PlaybookRunService.cs` (around lines 30-50, 247-260) — the per-run policy minting code Phase 1 must align with
- `/home/todd/workspace/cloud-manager/planning/workflow-plans/cloud-manager/poppy-fiddle/` — this PLAN's planning artifacts
- `Retro/FIX-001.md` from `workflow-exec/cloud-manager/snowy-omelette/` — the audit-log workflow's retro, for context on prior masked bugs in the same code paths

## Constraints

- **No `psql` UPDATE on `vm.playbooks.content`** — every playbook edit goes via `cloud_playbook_update` (the MCP tool that wraps the API). If the tool doesn't support content updates yet, extend the API + MCP tool as Phase 3 BEFORE editing the playbooks in Phase 4.
- **No `validate_certs: false`** in the final playbooks. If the TLS cert doesn't validate after the VAULT_ADDR change, that's a real bug to fix — not something to paper over.
- **`bootstrap-policies.py` is idempotent** — re-running must produce no Vault state change. Use `vault policy read` / `vault read auth/token/roles/...` to diff before writing.
- **`import-playbooks.py` preserves operator-authored playbooks** in the DB that aren't in `seed/playbooks/`. It syncs only what's in the seed directory.
- **camelCase JSON on the wire; public_ids only.** Standing project rule for any new API surface or MCP tool extension.
- **`@cloud-manager-deploy` post-phase is mandatory.** Porch is the runtime; cloud-manager-api hosts new endpoints; deploy both before `hv_ship` so the running system reflects the merged code.
