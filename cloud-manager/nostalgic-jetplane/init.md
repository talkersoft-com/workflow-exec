# Cloud Manager — MCP + API gap-closing for VM secret-binding management

## What this workflow does
Closes the last two gaps that stop an agent from fully **managing and verifying** user-defined
secrets, secret bindings, and their blueprint attachments through the cloud-manager MCP. This is
workflow **#3** of the Secret Bindings feature (after #1/#1.5/#2). It adds:

1. **API** — a new read endpoint `GET /api/v1/vm/{publicId}/secret-binding` that exposes the
   `vm.vm_secret_bindings` usage rows #2 records on provision (metadata + Vault **path** only,
   never a value).
2. **MCP** — `cloud_vm_secret_binding_list`, the **verification tool** that wraps the new endpoint
   ("confirm `DBPASSWD` from `sb_…` was injected into this VM").
3. **MCP** — `cloud_blueprint_secret_binding_update`, wrapping the **existing**
   `PATCH /api/v1/blueprint/{blueprintId}/secret-binding/{bsbId}` (change `credName` / reorder).

Together these complete the surface needed for an end-to-end attach → update → provision →
list-usage verification flow.

## Read before starting
- `deck.md` — deck + repos in scope; ship command; the `/mcp` reload operator note
- `Execution/Exec.md` — full task list and execution instructions
- `../../../workflow-plans/cloud-manager/nostalgic-jetplane/PLAN.md` — the authoritative design
  (objective, gap analysis, endpoint/tool shapes, phases, open question)
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns

## Constraints
- **No DB changes, no vorch, no web.** API + MCP only.
- **The secret value is NEVER returned** — the new endpoint surfaces metadata + the resolved Vault
  **path** only (same rule that keeps `GET /run/{id}/secret` off the MCP).
- **camelCase JSON on the wire; public_ids only.** Internal `Guid`s never leak — use
  `PublicIdByGuidAsync` / `GuidByPublicIdAsync`. The new endpoint takes a VM `publicId` and returns
  `vsb_…` / `sb_…` public ids only.
- **Soft-delete authoritative.** The usage-row query is soft-delete aware.
- **404** when the VM does not exist.
- **Mirror the existing paradigm** — model the controller/service on the existing `SecretBinding*`
  controller + service, and the MCP tools on `cloud_blueprint_playbook_reorder` / the existing
  blueprint secret-binding attach tool. Do not invent a new pattern.
- **OUT OF SCOPE:** any new DB column/table, secret rotation, vorch/web changes, returning secret
  values.

## Dependency
Builds on **#2 (`primordial-crystalball`)**, which populates `vm.vm_secret_bindings` on provision,
and **#1.5 (`wheezy-cottonmouth`)**, the Secret entity. Both are on `origin/hive` — the
`vm_secret_bindings` table and the `PATCH` attachment endpoint already exist; this workflow only
exposes/reads them.
