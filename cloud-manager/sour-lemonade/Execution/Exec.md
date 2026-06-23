# Orchestration: cloud-manager/sour-lemonade — Secret Bindings (authoring foundation)

## Objective
Operators can author **Secret Bindings** (reusable, parameterized Vault-secret definitions) and
attach them to marketplace blueprints — across cloud-manager-api, cloud-manager-mcp, and
cloud-manager-web. Done = both canonical bindings (manual no-param `/manual/secret/i/stored`,
and postgres `cloudmanager/data/vm/instances/{vm_public_id}/postgres` with one `vm` param) can be
created in the UI, attached to a blueprint, and round-tripped via API + MCP. Provisioning/injection
is OUT OF SCOPE (follow-up workflow) — only the schema seams land here.

## Inputs (read before Task 0000)
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/sour-lemonade/PLAN.md`

## Task list
- [x] `Tasks/0000-TASK.md` — Phase 0: hv_status + hv_init/hv_next
- [x] `Tasks/0001-TASK.md` — Phase 1: API entities + migration + CRUD + attach endpoints
- [x] `Tasks/0002-TASK.md` — Phase 2: MCP tools
- [x] `Tasks/0003-TASK.md` — Phase 3: web Secret Bindings page + blueprint "Attached Secrets" UI
- [ ] `Tasks/0004-TASK.md` — Phase 4: verify + build + deploy; then Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs before Task 0000.
2. For each unchecked task in order: read task -> do work -> run matching Test -> on fail write `Retro/FIX-NNN.md`, fix, re-run -> on pass check the box.
3. When every box is checked, write `Results/RESULT.md` + `Retro/LESSONS.md`, then `hv_integrate`.

## Autonomous execution
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "sour-lemonade"
```

## Project conventions (hard rules)
- camelCase JSON on the wire; **public_ids only** (never leak Guids — use PublicIdByGuidAsync / GuidByPublicIdAsync).
- **Soft-delete authoritative**; EF global query filter hides deleted rows.
- Mirror the existing `blueprint_playbooks` service/controller/DTO code paths — do not invent a new paradigm.
- Migration is **reversible** (clean up + down).
- Build after each change (`.cicd/build-cloud-manager.py`); a task isn't done until its build passes.
- Deploy api + web BEFORE the final hv_integrate; apply the migration BEFORE the api restart.

## Improvisation policy
- One FIX file per distinct failure (FIX-001, ...). Never silently retry. Two failed attempts -> STOP and surface.

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
