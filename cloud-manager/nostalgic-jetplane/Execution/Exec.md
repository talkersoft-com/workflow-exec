# Orchestration: cloud-manager/nostalgic-jetplane

## Objective
Close the two gaps that stop an agent from fully managing and verifying secret bindings through the
cloud-manager MCP. Done means: (1) `GET /api/v1/vm/{publicId}/secret-binding` returns a VM's
`vm_secret_bindings` usage rows (metadata + resolved Vault **path**, never a value; 404 on unknown
VM; camelCase; public_ids only); (2) `cloud_vm_secret_binding_list` wraps it as the verification
tool; (3) `cloud_blueprint_secret_binding_update` wraps the existing PATCH attachment endpoint
(credName/position, send-only-provided); and the full attach → update → provision → list-usage
round-trip works against the live API. No DB, no vorch, no web changes.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/nostalgic-jetplane/PLAN.md` — the authoritative design

## Dependency
**#2 (`primordial-crystalball`) and #1.5 (`wheezy-cottonmouth`) must be merged** (both on
origin/hive). Task 0000 confirms the `vm.vm_secret_bindings` table and the existing
`PATCH /api/v1/blueprint/{blueprintId}/secret-binding/{bsbId}` endpoint are present before
proceeding — if either is absent, STOP and surface it.

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record branch; verify #2/#1.5 surface present
- [x] `Tasks/0001-TASK.md` — **Phase 1**: API — `GET /api/v1/vm/{publicId}/secret-binding` + backing service (join, public_ids, no value, 404)
- [x] `Tasks/0002-TASK.md` — **Phase 2**: MCP — `cloud_vm_secret_binding_list` (wraps the new endpoint; no value)
- [x] `Tasks/0003-TASK.md` — **Phase 3**: MCP — `cloud_blueprint_secret_binding_update` (wraps the existing PATCH; optional credName/position)
- [x] `Tasks/0004-TASK.md` — **Phase 4**: Verify round-trip + build + deploy; write Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file (`Test/NNNN-TEST.md`)
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "nostalgic-jetplane"
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, …)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## Project conventions (apply to every task)
- **The secret value is NEVER returned** in any API response or MCP output — only metadata + the
  resolved Vault **path**. Same rule that keeps `GET /run/{id}/secret` off the MCP.
- **camelCase JSON; public_ids only** (use `PublicIdByGuidAsync` / `GuidByPublicIdAsync`). The
  endpoint takes a VM `publicId` and returns `vsb_…` / `sb_…` ids — never a `Guid`.
- **Soft-delete authoritative** — the usage-row query honors `deleted_at`.
- **404** when the VM does not exist.
- **Mirror the existing paradigm** — base the controller/service on the existing `SecretBinding*`
  controller + service; base the MCP tools on `cloud_blueprint_playbook_reorder` and the existing
  blueprint secret-binding attach tool. `update` sends only the fields the caller provided.
- **No DB / vorch / web changes.**

## Build & deploy (mandatory, BEFORE hv_integrate)
- Build per changed target: `python3 .cicd/build-cloud-manager.py --target api` (Phase 1) and
  `--target mcp` (Phases 2–3), or `--target all` at the end.
- Deploy api: `python3 .cicd/deploy-cloud-manager.py --target api`; smoke via `cloud_health_check`.
- Deploy mcp: `python3 .cicd/deploy-cloud-manager.py --target mcp` (rebuilds dist; does NOT restart
  the running server).
- **No EF migration** is added — do NOT run `dotnet ef database update`.
- Because new MCP tools were added (a contract change), `Results/RESULT.md` MUST carry, near the top:
  > **Operator action required:** Restart cloud-manager-mcp via `/mcp` in Claude Code to pick up the new tools.

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
