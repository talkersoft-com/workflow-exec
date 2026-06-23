# RabbitMQ Marketplace Blueprint — Execution

## What this workflow does
Adds RabbitMQ as a one-click marketplace item in Cloud Manager (jammy + noble),
mirroring the existing `postgres-jammy` blueprint. New seed playbooks + one EF
migration; reuses `import-playbooks.py` and the `marketplace` feature flag. No new
C# controllers/services.

## Read before starting
- `../../workflow-plans/cloud-manager/tigerish-deli/PLAN.md` — the approved plan
- `deck.md` — deck/repos in scope + pre-written hv calls
- `Execution/Exec.md` — task list and execution instructions
- Prior art: `cloud-manager-api/seed/playbooks/pg-14-jammy.*`, `pg-16-noble.*`,
  migration `20260611040351_AddMarketplaceBlueprints.cs`

## Constraints
- camelCase JSON / public_ids only on the wire — no raw Guids leak.
- Reuse `.cicd/import-playbooks.py` unchanged; reuse the `marketplace` feature flag.
- No new controllers/services — the generic Playbook/Marketplace/Blueprint API handles it.
- EF migration must be reversible (`Down` deletes the seeded rows).
- Apply the migration to the live DB BEFORE `deploy --target api`.
