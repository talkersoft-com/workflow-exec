# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `cloud-manager-api` | New `GET /api/v1/vm/{publicId}/secret-binding` on `VirtualMachineController` + backing service method: query `vm_secret_bindings` for the VM, join `secret_bindings` for the name, soft-delete aware; return usage rows (`publicId`, `credName`, `secretBindingId`, `secretBindingName`, `resolvedVaultPath`, `createdAt`) — **path, never value**; 404 when VM missing; camelCase; public_ids only. |
| `cloud-manager-mcp` | Two new tools: `cloud_vm_secret_binding_list` (wraps the new GET; input `id` = `vm_…`; returns usage rows, no value) and `cloud_blueprint_secret_binding_update` (wraps the existing `PATCH /api/v1/blueprint/{blueprintId}/secret-binding/{bsbId}`; inputs `id`, `bsbId`, optional `credName`, optional `position`; send only provided fields). |

Repos not listed will be on the feature branch but skipped by hv_integrate. **No DB, no vorch, no web.**

## Branch
`coppery-canteloupe` — execution branch minted by hv_next in the warmup (2026-06-26), based on
origin/hive across all 16 repos.

## Dependency
Builds on **#2 (`primordial-crystalball`)** — populates `vm.vm_secret_bindings` — and **#1.5
(`wheezy-cottonmouth`)** — the `Secret` entity. Both already on `origin/hive`. The
`vm_secret_bindings` table and the `PATCH` blueprint↔binding endpoint **already exist**; this
workflow only exposes / reads them. No schema change is introduced.

## Manual steps
**MCP `/mcp` reload:** the deploy script rebuilds the MCP dist but does NOT restart the running
cloud-manager-mcp server. Because this workflow adds **new tools** (a contract change), the operator
must reload via `/mcp` in Claude Code before the new tools are callable. Surface this prominently in
`Results/RESULT.md`.

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
```
If already provisioned on a prior feature branch with all PRs merged:
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```
(The warmup already ran `hv_next` → `rakish-coldfront`; Task 0000 verifies and records it.)

## Ship
```
hv_integrate  deck: "cloud-manager"
              message: "feat: VM secret-binding read endpoint + MCP list/update tools (Secret Bindings #3)"
              title:   "feat: VM secret-binding read endpoint + MCP list/update tools (Secret Bindings #3)"
              stage:   "exec"
```
