# Secret Binding provisioning hardening + instantiate-time param resolution (splendid-mermaid)

## What this workflow does
Workflow #4 of the Secret Bindings feature. Fixes the regression where a blueprint carrying a
templated `{vm_public_id}` secret binding bricks every VM provisioned from it, and makes templated
bindings usable by resolving `{vm_public_id}` to an operator-chosen source VM at instantiate.
- **P0** — resolve secret bindings BEFORE the VM record is created; a resolution failure returns a
  clean 422 (binding name + Vault path, never the value) with no orphaned VM.
- **P1** — `InstantiateRequest.secretBindingParams` supplies per-binding param values; templated
  `{vm_public_id}` resolves to the supplied source VM; required-param-missing → 422, never bricks.
- **MCP** — `cloud_marketplace_provision` gains an optional `secretBindingParams` passthrough.

## Read before starting
- `deck.md` — scope and hv MCP calls
- `Execution/Exec.md` — task list
- `../../../workflow-plans/cloud-manager/splendid-mermaid/PLAN.md`

## Constraints
- **No DB migration. Vorch unchanged.** camelCase on the wire; public_ids only
  (`PublicIdByGuidAsync` / `GuidByPublicIdAsync`); soft-delete authoritative.
- Secret values NEVER appear in API output, logs, or run streams — errors carry only the binding
  name + Vault path.
- Backward compatibility is mandatory: no-binding blueprints, static bindings, and concrete-path
  templated bindings must remain byte-for-behavior identical.
- Build per phase (`--target api` / `--target mcp`); deploy api (and rebuild mcp) before ship;
  E2E verification against the deployed system.
