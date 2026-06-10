# Result: cloud-manager/gloppy-snapper — vault-backed database connections

## Outcome — complete

All four phases delivered. Vault-backed database config entries now work end-to-end in the
database-toolkit MCP: `db_test_connection`, `db_list_databases`, and `db_execute_sql_query` all
succeed against `pg-test`, whose credentials live in Vault KV v2
(`cloudmanager/vm/instances/vm_SMAZ3C2YVZ/postgres`), routed through an SSH tunnel
(`localhost:15432 → 10.0.150.61:5432`). Non-vault entries (`clouddb`, `appdb`, `clouddb_local`)
verified unchanged.

## What shipped

| Repo | Change |
|------|--------|
| `database-toolkit` | `resolveVaultEntries` wired into BOTH loader stacks: `tools/shared/database-config-loader.ts` (`loadDatabaseConfigWithFallback`/`loadTestDatabaseConfigWithFallback`) and `tools/shared/config-provider.ts` (`AutoConfigManager.loadDatabaseConfig` — the stack the MCP tools actually use); dist rebuilt |
| `cloud-manager-api` | `seed/playbooks/pg-14-jammy.yaml`, `seed/playbooks/pg-16-noble.yaml` — postgres playbooks now configure `listen_addresses` + scram pg_hba scoped to the VM network and KVM host (Phase 2's conditional fix; a playbook does own postgres setup) |
| `workflow-plans` | `FINDINGS-0002-postgres-provisioning.md`, `FINDINGS-0003-ssh-secret-shape.md` in `cloud-manager/beaming-messenger/` |
| `workflow-exec` | This orchestration folder (tasks, tests, 3 FIX notes, this result) |

Local config (not in any repo): `~/.hv/toolkit/db/config/database.json` pg-test entry gained
`vaultPath`, `usernameOverride: "appuser"`, `host: "localhost"`, `port: 15432`.

## Verification evidence

- Fresh-server stdio smoke (post-reload-equivalent): pg-test connect / list-dbs / `SELECT
  version()` (PostgreSQL 14) all PASS; clouddb + identifier-set regressions PASS.
- TC-001..TC-005 from Test/0001 passed during execution (TC-002 amended per FIX-002 — pre-existing
  typecheck OOM; TC-003 amended per FIX-003 — both loader files intentionally changed).

## Deviations from plan

- The resolver had to be wired into a second loader (`config-provider.ts`) not named in the plan —
  the plan's own acceptance line ("apply to all config load paths used by the MCP tools") covered
  it. See Retro/FIX-003.md.
- Operator `/mcp` reload note: any long-running session (including the one that executed this
  workflow) still runs the pre-rebuild dist until its database-toolkit MCP server is restarted.
  The dist itself is proven good.

## Follow-up candidates (not shipped)

1. `database-lib/database/factory.ts:54` — trigger provider resolution on the new
   `credentialProvider`/`tunnelProvider` fields (auto-credentials + auto-tunnel per connection;
   would obsolete manual tunnel setup for vault entries).
2. SSH secret shape normalization — blocked on vorch no-change convention; see
   `FINDINGS-0003-ssh-secret-shape.md` for the writer/reader map and recommendation.
