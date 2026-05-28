# Orchestration: cloud-manager/lilac-redpanda

## Objective
An operator on `/ansible/collections` can add `community.postgresql` to the catalog, click Install on the controller host, watch it flip Pending → Installing → Installed within ~30s with a real version populated, click Remove to flip it back to Removed, and Install again to reinstall. The end-to-end pipeline (API → AMQP → vorch → ansible-galaxy → API callback → UI) is fully wired. The `pg` playbook against `postgres-test` no longer fails with "module not found" for `community.postgresql.postgresql_db` or `community.hashi_vault.vault_kv2_write`.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../common/toolkit/hive-deck.md`
- `../../../workflow-plans/cloud-manager/peppery-kudu/PLAN.md`
- `../../../workflow-plans/DATA-MODEL.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record execution branch
- [x] `Tasks/0001-TASK.md` — **Phase 1**: EF Core migration (ansible schema, 4 tables, CollectionInstallStatus enum, seed)
- [x] `Tasks/0002-TASK.md` — **Phase 2**: API entities + DTOs + AutoMapper + service interfaces + DI
- [x] `Tasks/0003-TASK.md` — **Phase 3**: API endpoints (10 routes under /api/v1/ansible)
- [x] `Tasks/0004-TASK.md` — **Phase 4**: API AMQP publishers (CollectionInstallPublisher + CollectionRemovePublisher)
- [x] `Tasks/0005-TASK.md` — **Phase 5**: vorch-lib message structs (camelCase JSON tags)
- [x] `Tasks/0006-TASK.md` — **Phase 6**: vorch-service subscribers + handlers + widen ReadWritePaths
- [x] `Tasks/0007-TASK.md` — **Phase 7**: Web Collections sub-nav tab + list/detail/modal + 3 slices + polling
- [x] `Tasks/0008-TASK.md` — **Phase 8**: Build + deploy (migration apply, all three install-*.py)
- [x] `Tasks/0009-TASK.md` — **Phase 9**: Playwright e2e verification of the six acceptance criteria
- [x] `Tasks/0010-TASK.md` — **Phase 10**: Write Results + LESSONS; hv_ship

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order:
   a. Read the task file
   b. Do the work
   c. Run the matching Test file
   d. On failure: write `Retro/FIX-NNN.md`, apply fix, re-run test
   e. On pass: check the box, move to the next task
3. When every box is checked, the workflow is complete

## Autonomous execution
```
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/workflow-exec/cloud-manager/lilac-redpanda/Orchestrate/ORCH.md. Start by running
hv_status then hv_init (or hv_next) per deck.md. For each unchecked task: read
the task file, do the work, run the matching Test file. On failure write
Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked
write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship —
then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator
- **No scope creep**: don't add requirements enforcement to the Run trigger, don't widen ReadWritePaths beyond the one collection path, don't add a Galaxy API integration, don't multi-controller-ize. Schema + UI flows only.

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
