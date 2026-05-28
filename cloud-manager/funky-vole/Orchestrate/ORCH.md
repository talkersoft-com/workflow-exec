# Orchestration: cloud-manager/funky-vole

## Objective
Three features ship as one PR set: (A) cloud-manager-api mints a scoped Vault token at run trigger time, vorch forwards it as `VAULT_TOKEN`, and the API revokes it on terminal status; (B) `playbook_collection_requirements` enforced at trigger — 400 with `{ missing: [...] }` if any are unmet on the chosen controller; (C) controller-host selector in the Collections UI backed by `bare_metal.hosts.is_controller`. `pg` against `postgres-test` writes its generated password to `cloudmanager/data/playbook-secrets/<runId>/postgres-test/postgres`. Requirement mismatches caught pre-dispatch. UI never silently picks `hosts[0]`.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../common/toolkit/hive-deck.md`
- `../../../workflow-plans/cloud-manager/nimble-orangutan/PLAN.md`
- `../../../workflow-plans/DATA-MODEL.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record execution branch
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: EF migration — `vault_run_token` on playbook_runs, `is_controller` on hosts + backfill
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: `CloudManager.Vault.Client` project (IVaultClient + HTTP impl) + Serilog redaction enricher + DI
- [ ] `Tasks/0003-TASK.md` — **Phase 3 (A)**: `TriggerMultiVmAsync` mint; PATCH callback + CancelAsync revoke; `VaultRunTokenSweeper` BackgroundService; DTO field + scrub
- [ ] `Tasks/0004-TASK.md` — **Phase 4 (A)**: vorch-service `runDTO.vaultRunToken` + `executeOneTarget` env injection
- [ ] `Tasks/0005-TASK.md` — **Phase 5 (B)**: requirements DTOs + Service + 3 endpoints + DI
- [ ] `Tasks/0006-TASK.md` — **Phase 6 (B)**: enforcement in `TriggerMultiVmAsync` + `RequirementsUnmetException` + 400 mapping + `SatisfiesVersion`
- [ ] `Tasks/0007-TASK.md` — **Phase 7 (C)**: Host.IsController entity + DbContext + mapper + DTO + API surface + `ResolveControllerHostIdAsync`
- [ ] `Tasks/0008-TASK.md` — **Phase 8 (C)**: web controller selector + persistence + Collections list/detail integration + All-controllers summary
- [ ] `Tasks/0009-TASK.md` — **Phase 9 (B)**: `playbookCollectionRequirementsSlice` + Required-collections panel + Add-requirement modal + banner
- [ ] `Tasks/0010-TASK.md` — **Phase 10 (B)**: `AnsibleRunPage` requirement check + disabled Run button + inline message
- [ ] `Tasks/0011-TASK.md` — **Phase 11**: Build + deploy (migration + api + vorch + web)
- [ ] `Tasks/0012-TASK.md` — **Phase 12**: Playwright + Vault audit verification (A.6 / B.5 / C.5 acceptance)
- [ ] `Tasks/0013-TASK.md` — **Phase 13**: Write Results + LESSONS; hv_ship

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
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/workflow-exec/cloud-manager/funky-vole/Orchestrate/ORCH.md. Start by running
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
- **No scope creep**: don't add `host_id` to `playbook_runs`, don't implement auto-install in B, don't add a controller-pool model.

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
