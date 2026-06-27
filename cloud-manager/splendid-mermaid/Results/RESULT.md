# Result: cloud-manager/splendid-mermaid

**Workflow #4 of Secret Bindings — provisioning hardening + instantiate-time param resolution.**
Outcome: **SUCCESS.** A blueprint carrying a templated `{vm_public_id}` secret binding no longer
bricks provisioning; secret bindings resolve **before** the VM record exists; a failure returns a
clean **422** (binding name + Vault path, never the value) with **no orphan**; an operator can supply
the SOURCE VM via `secretBindingParams` so a consumer VM is injected with another VM's existing
creds. Verified end-to-end against the deployed system. No DB migration; vorch/web/CLI unchanged.

## Execution branch
`ventured-hyena` (minted from origin/hive by the warm-up `hv_next`, 16 repos). Used directly as the
Task 0000 execution branch — no second branch created.

## PR table
PRs are opened by `hv_integrate` (auto-merge + transition). Links are reported in the final agent
message and in the `hv_integrate` output. Repos with code changes: **cloud-manager-api**,
**cloud-manager-mcp**, **workflow-exec** (this scaffold/runtime).

| Repo | Change |
|------|--------|
| cloud-manager-api | P0 resolve-before-create + clean 422 + failure-event guard; P1 `secretBindingParams` |
| cloud-manager-mcp | `cloud_marketplace_provision` optional `secretBindingParams` passthrough |
| workflow-exec | orchestration runtime (Exec checkboxes, deck.md branch, Results, Retro) |

## Per-phase summary
- **0000 Setup** — verified deck clean on a feature branch; recorded execution branch `ventured-hyena`
  in deck.md. TEST 0000 ✓ (all repos one feature branch, clean, branch recorded).
- **0001 API P0** — `MarketplaceController.Instantiate` now resolves the blueprint's secret bindings
  **before** creating the VM record. New `SecretBindingResolutionException` (binding name + Vault
  path, never the value); resolver wraps `VaultSecretNotFoundException` / config errors into it. A
  resolution failure → **422 UnprocessableEntity**, no VM row. Defense-in-depth
  `TryRecordInstantiateFailureAsync` records a `blueprint_run_chain_failed` event and marks the VM
  `Failed` for any post-create failure (no eventless `orch=0` orphan). api build green. TEST 0001 ✓.
- **0002 API P1** — `InstantiateRequest.secretBindingParams` (array of `{ secretBindingId, params }`,
  camelCase). `SecretBindingResolver.ResolveForBlueprintAsync(blueprintId, bindingParams)` substitutes
  the supplied `vm_public_id` (the SOURCE VM) into the templated path; required-missing → 422 naming
  the binding; **never** falls back to the new VM's id. api build green. TEST 0002 ✓.
- **0003 MCP** — `cloud_marketplace_provision` gains optional `secretBindingParams` (array of
  `{ secretBindingId, params }`) round-tripped verbatim (camelCase) to the Instantiate body; omitted →
  unchanged. No other tool surface changed. mcp build green. TEST 0003 ✓.
- **0004 E2E + deploy** — see below. TEST 0004 ✓.

## Build + deploy
- `build-cloud-manager.py --target all` → **green** (api + mcp + web).
- `deploy-cloud-manager.py --target api` → publish + restart; **smoke `GET /api/v1/Host/list`
  HTTP 200 after 4 attempts**. Redeployed twice during E2E for FIX-001 + the 422-path refinement;
  each smoke HTTP 200.
- `deploy-cloud-manager.py --target mcp` → dist rebuilt; package-lock did not drift.

## E2E (deployed system — MCP + db/psql + curl)
Fixtures: published templated consumer blueprint `bp_HVTW8NWCZZ` (`splendid-mermaid-consumer`) with
the templated binding `sb_CTHR021ZR5` → `cloudmanager/data/vm/instances/{vm_public_id}/postgres`,
cred `DBPASSWD`. Source VM = `pg-test123` (`vm_W28X31Y8SX`, soft-deleted but its secret persists).

- **TC-003 Happy path (P1):** `POST .../bp_HVTW8NWCZZ/instantiate` with
  `secretBindingParams:[{secretBindingId:sb_CTHR021ZR5, params:{vm_public_id:vm_W28X31Y8SX}}]` →
  **HTTP 200**, VM `vm_ZDQP0MFERM` (`sm-consumer-1`). Provisioned to **orchestrationStatus=2 (Completed)**,
  IP `10.0.150.87` (create-vm command ran end-to-end — **not orphaned**). `vm_secret_bindings` row
  for `DBPASSWD` **recorded** (on create-vm Success). Success is itself proof the SOURCE path was used:
  a fresh VM has no secret at its own path, so resolving to `vm_ZDQP0MFERM`'s id would have 422'd —
  it resolved to pg-test123's path and injected.
- **TC-004 Clean 422 (P0):** same blueprint with **no** `secretBindingParams` → **HTTP 422**
  `{secretBinding:"VM postgres credential", vaultPath:"cloudmanager/data/vm/instances/{vm_public_id}/postgres"}`,
  no secret value. **Zero** `vm.virtual_machines` rows for `sm-consumer-noparam` (verified via psql) —
  no orphan, no eventless `orch=0`.
- **TC-005 Regression:** no-binding blueprint `jammy` → HTTP 200 `vm_M632QA5BMJ` (orch=2, IP
  10.0.150.88); static-binding blueprint `njp-e2e-static-bp` → HTTP 200 `vm_GAFVT5CVV8` (orch=2, IP
  10.0.150.89). Both provision normally — unchanged.

## ⚠ Operator action — `/mcp` reload required
The MCP tool contract changed (`cloud_marketplace_provision` gained `secretBindingParams`) and the
dist was rebuilt. **Restart the cloud-manager MCP via `/mcp` in Claude Code** to pick up the new tool
schema. Until then, this session's MCP server still advertises the old schema — the E2E happy/422
cases were therefore driven via direct `curl` against the deployed Instantiate endpoint.

## FIX files
- `Retro/FIX-001.md` — source-VM validation rejected a soft-deleted source VM (pg-test123). Fixed by
  validating with `IgnoreQueryFilters()` (a secret legitimately outlives its source VM). Confirmed by
  re-run: 422 → 200 + injection.
