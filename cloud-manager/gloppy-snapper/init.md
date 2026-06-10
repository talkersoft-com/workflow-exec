# Vault-backed database connections — resolver wiring + provisioning investigations

## What this workflow does

Executes the plan in `workflow-plans/cloud-manager/beaming-messenger/PLAN.md`. Fixes the one verified bug from the 2026-06-10 pg-test connectivity session — the database-toolkit's Vault credential resolver (`tools/shared/vault-resolver.ts`) exists in source and dist but is imported by nothing, so vault-backed db configs (`vaultPath` entries like `pg-test`) fail with a missing-fields config error — and runs two scoped investigations (postgres VM network provisioning, double-encoded per-VM SSH secret) whose deliverables are findings documents, not speculative code changes. End state: `db_test_connection pg-test` works through an SSH tunnel, every other db tool works against pg-test, non-vault configs are byte-identical to today, and two evidence-based findings docs land in the plan folder.

## Read before starting

- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and execution instructions
- `../../../workflow-plans/cloud-manager/beaming-messenger/PLAN.md` — design, acceptance criteria, and the verified-vs-investigation split
- `../../../workflow-plans/cloud-manager/beaming-messenger/deck.md` — original deck/repo list from planning (now on branch `gloppy-snapper`)
- `../../../workflow-fragments/fragments/rebuild-mcp-artifacts.md` — mandatory rebuild + restart rules after MCP code changes
- `tools/database-toolkit/tools/shared/vault-resolver.ts` — the orphaned resolver; its merge semantics and token sourcing are already correct and must not change
- `tools/database-toolkit/tools/shared/database-config-loader.ts` — the two functions to wire: `loadDatabaseConfigWithFallback` (line ~77), `loadTestDatabaseConfigWithFallback` (line ~130)

## Constraints

- Do not modify vorch-lib, vorch-service, or porch (deck convention). If an investigation points there, the deliverable is a flagged recommendation, not a change.
- Non-vault database configs (`clouddb`, `appdb`, `clouddb_local`) must behave exactly as before the resolver wiring.
- Never log secret material (tokens, passwords, private keys) — not in command output, not in findings docs, not in test evidence. Assert presence with booleans, never print values.
- A Vault failure on one config entry must not break other entries (the resolver already isolates per-entry failures — preserve that).
- Do not change `vault-resolver.ts` itself: token source order (`CM_VAULT_TOKEN` env, then Keychain `vault-admin-token`), `VAULT_ADDR` override, and explicit-entry-fields-win merge semantics are already as designed.
- Tasks 0002 and 0003 produce findings documents; code changes only where the investigation confirms ownership (0002, scoped) or never (0003).
- Keep task/test count proportional — this is one small code change plus two investigations.
