# Result: Secret Bindings — authoring foundation (cloud-manager/sour-lemonade)

## Outcome: ✅ Complete

Operators can author **Secret Bindings** (reusable, parameterized Vault-secret definitions) and
attach them to marketplace blueprints, across **cloud-manager-api**, **cloud-manager-mcp**, and
**cloud-manager-web**. Both canonical bindings round-trip across API + MCP + UI. Provisioning/
injection remain out of scope; only the schema seams (`params[].source`, `injection_provider`,
`cred_name`) landed.

- Git execution branch: `trusted-jetplane` (minted from `origin/hive` by Task 0000).
- Orchestration id: `sour-lemonade`.

## What shipped

| Repo | Change |
|------|--------|
| `cloud-manager-api` | `SecretBinding` + `BlueprintSecretBinding` EF entities; one **reversible** migration `AddSecretBindings` creating `marketplace.secret_bindings` + `marketplace.blueprint_secret_bindings`; `SecretBindingService` + `SecretBindingController` with CRUD + attach/detach/reorder/list (camelCase, public_ids `sb_`/`bsb_`, soft-delete authoritative, two-phase position renumber) — mirrors `blueprint_playbooks`. Gated behind the `marketplace` feature flag. No instantiate changes. |
| `cloud-manager-mcp` | New `secret-bindings.ts` tool module: `cloud_secret_binding_list/get/create/update/delete` + `cloud_blueprint_secret_binding_attach/detach/list`. Registered in `index.ts` + `profiles.json` (`hv.cloud`). README updated. |
| `cloud-manager-web` | New **Secret Bindings** management page (`/secret-bindings`) with a params editor that scans `path_template` for `{placeholders}` and renders a source picker per row; **Attached Secrets** section on the Blueprint detail page (attach / detach / reorder / set credName). Redux slice + API client + types + nav/route added. |

## Canonical bindings (authored via the live UI, confirmed across all three surfaces)

| Name | path_template | params |
|------|---------------|--------|
| manual stored secret | `/manual/secret/i/stored` | `[]` (no params — the manual pattern) |
| VM postgres credential | `cloudmanager/data/vm/instances/{vm_public_id}/postgres` | one `vm` param (`vm_public_id`) |

The **VM postgres credential** binding is attached to blueprint `postgres-jammy` (`bp_0ec6545632`) at
position 0 with `cred_name = DBPASSWD`, and the attachment survives reload.

## Verification (all green)

- **Task 0001** — migration up/down/up clean against live `clouddb`; CRUD round-trip; attach + unique-position; soft-delete hides row but keeps `deleted_at`; `--target api` build exits 0.
- **Task 0002** — `--target mcp` build exits 0; compiled tool handlers create/list/attach the two examples against the live API; inputs use `sb_`/`bp_` public_ids; camelCase contract enforced.
- **Task 0003** — `--target web` build exits 0 (tsc + vite); placeholder scan adds/clears param rows live; both bindings authored via the form; blueprint attach/detach/reorder persists after reload.
- **Task 0004** — migration applied **before** the api restart; `deploy --target all` succeeded; `cloud_health_check` → **Healthy**; web reachable (HTTP 200 at https://ubuntu-server.talkersoft.com). Final round-trip: UI = API = MCP for the catalog and the blueprint attachment (camelCase, public_ids, **no Guid leak**).

## ⚠ Operator action required

The MCP server **dist was rebuilt** but the running MCP process serves the old tool set. Run **`/mcp`**
in Claude Code to reload `cloud-manager-mcp` and pick up the new `cloud_secret_binding_*` /
`cloud_blueprint_secret_binding_*` tools.

## Pull Requests

All four PRs opened and **auto-merged** by `hv_integrate`; the deck auto-transitioned to the next
branch (`bunched-seagull`).

| Repo | PR | Status |
|------|----|--------|
| cloud-manager-api | [#37](https://github.com/talkersoft-com/cloud-manager-api/pull/37) | merged |
| cloud-manager-mcp | [#28](https://github.com/talkersoft-com/cloud-manager-mcp/pull/28) | merged |
| cloud-manager-web | [#20](https://github.com/talkersoft-com/cloud-manager-web/pull/20) | merged |
| workflow-exec (orchestration record) | [#76](https://github.com/talkersoft-com/workflow-exec/pull/76) | merged |

> Note: this RESULT.md PR-table fill lands as a small follow-up `hv_integrate` on `bunched-seagull`,
> since the URLs only exist after the first integrate (the three code PRs above are already merged).
