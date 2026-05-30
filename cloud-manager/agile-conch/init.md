# Harden playbook execution path

## What this workflow does

Executes the plan in `workflow-plans/cloud-manager/poppy-fiddle/PLAN.md`. Replaces six in-DB SQL triage patches we applied to the postgres playbook during interactive troubleshooting with proper code fixes across porch, cloud-manager-api, and the playbook content itself. Porch will inject `run_public_id` and use the FQDN Vault URL so playbooks can use the existing per-run-scoped Vault policy without `delegate_to: localhost`, `validate_certs: false`, or `python3-hvac` installed on every VM. A new export/import pair gives playbook bodies a git source-of-truth, and a `bootstrap-policies.py` script provisions the Vault `playbook-run` policy + role automatically. When complete, a fresh dev-box bootstrap via `bootstrap-server.py` ends with a green postgres playbook run end-to-end, no operator intervention.

## Read before starting

- `../../../workflow-plans/cloud-manager/poppy-fiddle/PLAN.md` — full design + before/after code blocks for every phase
- `../../../workflow-plans/cloud-manager/poppy-fiddle/deck.md` — original deck/repo list (now on branch `agile-conch`)
- `deck.md` — this branch's deck wiring (repos in scope, ship call)
- `Orchestrate/ORCH.md` — the task list and `hv_workflow_run` entrypoint
- `vorch-service/vorch-service/internal/ansible/playbook_run_exec.go` — Phase 1 + Phase 2 patch target
- `cloud-manager-api/src/Services/CloudManager.Data.Services/PlaybookRunService.cs` lines 30-50 + 247-260 — the per-run policy + token minting code Phase 1 must align with

## Constraints

- **No `psql` UPDATE on `vm.playbooks.content`.** Every playbook edit goes via `cloud_playbook_update`. If the tool doesn't yet support content updates, extend the API + MCP tool first (Phase 3 BEFORE Phase 4).
- **No `validate_certs: false`** in the final playbook content. If TLS fails after the VAULT_ADDR change, that's a real bug to fix — not papered over.
- **`bootstrap-policies.py` is idempotent.** Re-running produces no Vault state change.
- **`import-playbooks.py` preserves operator-authored playbooks** in the DB that aren't in `seed/playbooks/`. It syncs only what's in the seed directory.
- **camelCase JSON on the wire; public_ids only.** Standing project rule.
- **`@cloud-manager-deploy` post-phase is mandatory.** Porch is the runtime; cloud-manager-api hosts new endpoints; deploy both before `hv_ship` so the running system reflects the merged code.
