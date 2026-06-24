# Orchestration: cloud-manager/wheezy-cottonmouth

## Objective
Ship the **Secret Manager** for Cloud Manager: a first-class, CM-owned `Secret` entity (create
writes to Vault under a locked `cloudmanager/data/manual/{slug}` path **and** records a DB row),
a strict DB-first create/delete state machine, and a 3-level referential-integrity delete gate
(`Secret → Binding → VM`) enforced by `ON DELETE RESTRICT` FKs. Done means: an operator creates a
manual secret in the UI (lands in Vault + DB `Active`), references it from a Secret Binding,
and the delete gate blocks at each level — across api + mcp + web, built and deployed.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/wheezy-cottonmouth/PLAN.md` — the authoritative design

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: API — `Secret` + `VmSecretBinding` entities, nullable `secret_id` on `SecretBinding`, one reversible migration, Secret CRUD + create/delete state machine + path-lock
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: MCP — `cloud_secret_list / get / create / delete`
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: Web — Secret Manager page (fixed-prefix create, guarded delete) + binding→managed-secret reference
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: Verify + build + deploy; write Results + Retro/LESSONS, then hv_integrate

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
Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "wheezy-cottonmouth"
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, …)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## Project conventions (apply to every task)
- **camelCase JSON on the wire; public_ids only** — never leak Guids (use the existing
  `PublicIdByGuidAsync` / `GuidByPublicIdAsync` helpers).
- **Soft-delete authoritative** — stamp `deleted_at`; rely on the EF global query filter.
- **Vault tokens NEVER appear in logs or run output.** Secret *values* never appear in API
  responses, MCP output, or logs.
- **Mirror the existing `secret_bindings` service/controller code paths** — do not invent a
  parallel paradigm.
- **No vorch/porch changes** — this workflow does not touch the provisioning plane.

## Build & deploy (mandatory, BEFORE hv_integrate)
- Build per changed target: `python3 .cicd/build-cloud-manager.py --target {api|mcp|web|all}`
- Apply the EF migration to the live DB **before** `deploy --target api` (see PLAN / deploy
  fragment for the exact `dotnet ef database update` command).
- Deploy: `python3 .cicd/deploy-cloud-manager.py --target all`
- Restart cloud-manager-mcp via `/mcp` only if the MCP tool contract changed (it does — new tools).

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
