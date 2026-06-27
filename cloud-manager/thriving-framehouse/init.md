# Orchestration init: cloud-manager/thriving-framehouse

## Source plan
`/home/todd/workspace/cloud-manager/planning/workflow-plans/cloud-manager/thriving-framehouse/PLAN.md`
(approved via `hv_plan_approve`, workflow `feature@cloud-manager`)

## Title
Secret-binding verification fixes (MCP param guard + persisted resolved path)

## Objective
Fix the two verification-layer defects found during the live end-to-end test so templated
secret bindings are usable **through the MCP** and the verification endpoint reports the truth.
The feature itself is correct — a consumer VM was injected with a source Postgres VM's actual
credential, hash-verified against Vault. These are correctness fixes only:

- **#2 (MCP):** stop `cloud-manager-mcp`'s `apiRequest` camelCase guard from rejecting legitimate
  `secretBindingParams[].params` map keys (placeholder names like `vm_public_id`), so
  `cloud_marketplace_provision` can drive templated bindings.
- **#3 (API):** persist the **actual** resolved Vault path on the `vm.vm_secret_bindings` usage
  row and return it from `GET /api/v1/vm/{publicId}/secret-binding`, instead of recomputing it
  with the listed VM's own id (which is wrong for cross-VM templated bindings).

This is workflow **#5** of the Secret Bindings feature. No vorch/web changes; no change to
resolution/injection logic.

## Scope
- `cloud-manager-api` — one reversible migration (`resolved_vault_path` on `vm.vm_secret_bindings`),
  populate at provision from `SecretBindingResolver`, return stored value (recompute only when null).
- `cloud-manager-mcp` — narrow camelCase guard exemption for `secretBindingParams[].params` keys.

## Conventions to respect
- camelCase JSON on the wire; public_ids only (no internal Guids leak).
- Soft-delete authoritative for `vm.virtual_machines`; append-only audit log.
- Vault **paths** are fine to surface; secret **values** never appear in output or logs.
- No vorch changes. Mirror existing controller/service + MCP code paths.

## Backward compatibility
- Old usage rows (`resolved_vault_path` null) → best-effort recompute (today's behavior).
- New rows → the true resolved path.
- MCP guard change is additive — previously-valid calls stay valid.

## Open question (from plan)
Recompute fallback assumed kept for null (pre-change) rows. Backfilling old rows in the migration
instead is acceptable; most are throwaway test rows, so fallback is simpler.
