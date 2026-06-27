# RESULT — cloud-manager/thriving-framehouse

Secret-binding verification fixes (MCP param guard + persisted resolved path) — workflow #5.

## ⚠️ OPERATOR ACTION REQUIRED — `/mcp` reload
The MCP guard fix ships in `cloud-manager-mcp`'s **dist**, but the **running** MCP server still
carries the old guard (a build does not restart it). **Reload it with `/mcp`** so the running
server accepts `secretBindingParams[].params` snake_case placeholder keys. Until then,
`cloud_marketplace_provision` with `params:{vm_public_id:…}` still raises the camelCase guard
error (verified live — see E2E below). After reload, the live through-the-MCP E2E (Pass A + B)
can be run.

## Outcome
- **Branch (execution):** `fuchsia-deertrail` (deck `cloud-manager`, minted from `origin/hive`).
- **Status:** Both code fixes implemented, build clean, API deployed, migration applied to live
  `clouddb`. Live through-the-MCP E2E is **operator-gated** on the `/mcp` reload above; the
  deployed API was verified to accept the fixed request shape directly (see Pass A).

## Deploy
- **Target:** `cloud-manager-api` (`python3 .cicd/deploy-cloud-manager.py --target api`).
- **Smoke:** `localhost:5250/api/v1/Host` → **HTTP 200** after 4s (attempt 3/10). Deploy complete.
- MCP dist rebuilt by its build; the running MCP server is **not** restarted by deploy → `/mcp`
  reload required (above).

## PR links
| Repo | PR | Title |
|------|----|-------|
| _filled by hv_integrate below_ | | |

(See the "Integrate" section / `gh pr list` — PR links appended after `hv_integrate`.)

## Per-phase summary
| Phase | Repo | What changed | Verified |
|-------|------|--------------|----------|
| 0000 | deck | Minted `fuchsia-deertrail` from `origin/hive`; recorded in deck.md | hv_status clean across 16 repos |
| 0001 | cloud-manager-api | Migration `AddResolvedVaultPathToVmSecretBindings` (nullable `resolved_vault_path` on `vm.vm_secret_bindings`); resolver surfaces the resolved path via new `ResolvedSecretBinding`; `MarketplaceController` records usage **at provision time** with the true source path (consumer ack can't carry it back); read endpoint returns stored value, recomputes only when null | TC-001..005: migration reversible+additive, api builds clean |
| 0002 | cloud-manager-mcp | `assertCamelCaseKeys` exempts **only** the direct keys of a `secretBindingParams[].params` map; every structural/wire field (incl. `secretBindingId`, `params` itself, siblings) still guarded | TC-001..004: guard unit-tested (6 cases), mcp builds clean, dist carries fix |
| 0003 | api + mcp | Migration applied to live `clouddb`; `--target all` builds clean; api deployed (smoke 200); E2E (below) | TC-001..003 green; TC-004/005 operator-gated |

## E2E (Task 0003 step 4)
- **Pass A — provision through MCP without guard error.**
  - **Live MCP (running server):** still STALE → reproduces the bug:
    `cloud_marketplace_provision(... params:{vm_public_id:"vm_PROBE000"})` →
    `apiRequest: snake_case key "vm_public_id" at $.secretBindingParams[0].params is not allowed`.
    This is the pre-`/mcp`-reload state and confirms the bug + that the fix isn't loaded yet.
  - **Deployed API accepts the fixed shape (equivalent, reload-independent):** direct
    `POST /api/v1/marketplace/bp_M963Q04W6H/instantiate` with
    `secretBindingParams:[{secretBindingId:"sb_KXF8GCB19H", params:{vm_public_id:"vm_PROBE000"}}]`
    → **HTTP 422** `Secret binding 'pg-consumer-binding' could not be resolved` (source VM not
    found) — i.e. the API did **not** reject the snake_case `params` keys; it parsed them and
    reached the resolver, creating no VM (resolution precedes VM creation). After `/mcp` reload the
    identical MCP call reaches the API exactly like this curl. **Guard unit test:** 6/6 cases pass
    (params keys accepted; top-level `bad_key`, `secret_binding_id`, and `extra_field` sibling all
    still rejected; nested-under-a-params-value still guarded).
  - **To complete Pass A live (after `/mcp` reload):** call `cloud_marketplace_provision` with a
    real source VM id in `params.vm_public_id` — the call succeeds (no guard error) and enqueues.
- **Pass B — read endpoint returns the true SOURCE path.** Operator-gated: needs a real source
  Postgres VM whose creds exist in Vault, then a consumer provision. The deployed code persists
  `resolved_vault_path` at provision time and the read endpoint returns it (recompute only when
  null) — verified by trace + clean build + live migration. **Steps after reload:**
  1. Provision source Postgres VM (blueprint `bp_0ec6545632` postgres-jammy) on host
     `host_KW7YKFF3YM`; let its playbook write creds to Vault (`…/vm/instances/<srcId>/postgres`).
  2. `cloud_marketplace_provision` consumer `bp_M963Q04W6H` with
     `secretBindingParams:[{secretBindingId:"sb_KXF8GCB19H", params:{vm_public_id:<srcId>}}]`.
  3. `cloud_vm_secret_binding_list <consumerId>` → `resolvedVaultPath` is the **source** path
     `cloudmanager/data/vm/instances/<srcId>/postgres` (NOT the consumer's id), read from the
     stored column. No secret value ever appears — Vault path only.

## Migration
- `20260627022632_AddResolvedVaultPathToVmSecretBindings` — `Up` adds nullable `resolved_vault_path`
  (text) to `vm.vm_secret_bindings`; `Down` drops it; no other table touched.
- Applied to live `clouddb` (localhost:5432). Column confirmed present (db-toolkit schema read).
  Note: the connection string in the task docs carried the stale dev password — see `Retro/FIX-001.md`.

## Conventions honored
camelCase on the wire; public_ids only; Vault **paths** surfaced, secret **values** never; no
vorch/web change (resolved path kept server-side, never added to the create-vm message); guard
change additive; old `resolved_vault_path`-null rows fall back to recompute.
