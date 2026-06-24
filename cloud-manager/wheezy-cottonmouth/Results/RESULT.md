# Result — Secret Manager (CM-owned secrets + referential-integrity model)

**Orchestration:** cloud-manager/wheezy-cottonmouth
**Execution branch:** `fortified-deadwood` (minted by hv_next from origin/hive, 2026-06-24)
**Outcome:** ✅ Complete — all five phases implemented, every TEST file green, built + deployed + verified live.

## What shipped
A first-class, CM-owned `Secret` entity and a **Secret Manager** so an operator can create / list /
delete Vault secrets manually. Creating a secret writes the value to Vault under the **locked**
`cloudmanager/data/manual/{slug}` path **and** records a DB row (strict DB-first state machine), and a
**3-level referential-integrity delete gate** (`Secret → Binding → VM`) is enforced by `ON DELETE
RESTRICT` FKs. Path-lock is enforced in three layers (DB CHECK, API validation, UI fixed-prefix).

## Phase summary
| Phase | Area | Result | Test |
|------|------|--------|------|
| 0000 | Setup | Branch `fortified-deadwood` minted + recorded in deck.md | 0000 ✓ (clean, on feature branch) |
| 0001 | API — entities, migration, CRUD + state machine + path-lock | `Secret` + `VmSecretBinding` entities, nullable `secret_id` on `SecretBinding`, one reversible migration `20260624212127_AddSecretManager`, Secret CRUD, KV v2 Vault I/O on `IVaultClient` | 0001 ✓ (up/down/up, CHECK, create→Active, 409 guard, leaf-delete hard-deletes Vault, FK RESTRICT, build) |
| 0002 | MCP — `cloud_secret_list/get/create/delete` | New `secrets` tool module + profile entry; mirrors the API; value never echoed | 0002 ✓ (registered, round-trip, 409 surfaced verbatim, lockfile-stable build) |
| 0003 | Web — Secret Manager page + binding→Secret reference | New page (fixed-prefix create, status badges, guarded delete) + Source toggle on the binding modal (Templated path / Reference managed Secret, derived read-only path) | 0003 ✓ (gated, prefix-locked, round-trip, 409 surfaced, both binding modes, aligned, build) |
| 0004 | Verify + build + deploy + ship | End-to-end on live API+DB+Vault, build all, deploy all, health green | 0004 ✓ (create, path-lock both layers, gate L2 + L1, build, deploy healthy) |

## End-to-end verification (live, port 5250 + clouddb + Vault)
- **Create** `e2e-demo` → `vaultPath = cloudmanager/data/manual/e2e-demo`, DB `status=Active`, value present in Vault. ✓
- **Path-lock** — API rejects an un-normalizable slug (400, no Vault write); DB `secrets_vault_path_owned_check` rejects a non-owned path. ✓
- **Gate L2 (Secret←Binding)** — referenced secret delete → **409** naming the binding; after unbind, leaf-delete → `status=Deleted`, Vault key **404 (destroyed)**. ✓
- **Gate L1 (Binding←VM)** — a `vm.vm_secret_bindings` usage row blocks the binding delete via FK RESTRICT; removing the row releases it. ✓
- All transient e2e artifacts cleaned up.

## Deploy smoke
- `deploy --target all`: api restarted → `GET /api/v1/Host/list` HTTP 200; mcp dist rebuilt (lockfile stable); web restarted → `:3000` HTTP 200.
- `cloud_health_check` → **Healthy**. `/api/v1/secret/list` → 200 (endpoints live).
- Migration `20260624212127_AddSecretManager` applied to live `clouddb` **before** the api restart.

## Repos changed (integrated from `fortified-deadwood`)
- **cloud-manager-api** — `Secret` + `VmSecretBinding` entities, nullable `secret_id` FK on `SecretBinding`, `SecretController` + `SecretService` + `SecretPath`, KV v2 methods on `IVaultClient`/`VaultClient`, one reversible migration, prefix registry (`sec`/`vsb`), DTO/profile/DI.
- **cloud-manager-mcp** — `src/tools/secrets.ts` (`cloud_secret_list/get/create/delete`), registered in `index.ts` + `profiles.json`.
- **cloud-manager-web** — `SecretsPage.tsx` + `secretsSlice` + route/nav/breadcrumb; binding modal "Reference managed Secret" mode; `secretId` on the binding type/api/slice.
- **planning/workflow-exec** — Exec checkboxes, deck.md branch, `Retro/FIX-001.md`, `Results/RESULT.md`, `Retro/LESSONS.md`.

## ⚠️ Operator action required
**Restart `cloud-manager-mcp` via `/mcp` in Claude Code** to pick up the rebuilt dist — the four new
`cloud_secret_*` tools are not live in this session until the MCP server reloads.

## Pull requests
See the hv_integrate output / `hv_list_pulls` — PRs open feature `fortified-deadwood` → integration
branch (`hive`) for each changed repo. (Reported by the agent after integrate.)
