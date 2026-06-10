# Orchestration: cloud-manager/gloppy-snapper

## Objective

Make vault-backed database config entries actually work in the database-toolkit MCP by wiring the orphaned `resolveVaultEntries` resolver into both config-loader paths, then close out the two adjacent findings from the 2026-06-10 pg-test connectivity session with evidence-based investigations. Done means: `db_test_connection` (and every other db tool) succeeds against the `pg-test` entry whose credentials live in Vault KV v2 (`cloudmanager/vm/instances/vm_SMAZ3C2YVZ/postgres`); non-vault entries behave byte-identically to today; and two findings docs sit in `workflow-plans/cloud-manager/beaming-messenger/` — one establishing whether postgres VM provisioning should configure network access, one mapping the writer and all readers of the double-encoded per-VM SSH secret.

## Inputs

Read these before Task 0000:

- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/beaming-messenger/PLAN.md`
- `../../../workflow-plans/cloud-manager/beaming-messenger/deck.md`
- `../../../workflow-fragments/fragments/rebuild-mcp-artifacts.md`

## Task list

Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status; confirm `gloppy-snapper` across all repos
- [x] `Tasks/0001-TASK.md` — **Phase 1**: wire `resolveVaultEntries` into `loadDatabaseConfigWithFallback` + `loadTestDatabaseConfigWithFallback`; rebuild dist; verify pg-test end-to-end through the SSH tunnel; non-vault configs unchanged
- [x] `Tasks/0002-TASK.md` — **Phase 2**: investigate postgres VM provisioning gap → findings doc; conditional, scoped playbook change only if a playbook owns postgres setup
- [x] `Tasks/0003-TASK.md` — **Phase 3**: investigate double-encoded SSH secret → map writer + all readers; written recommendation only, no code change
- [x] `Tasks/0004-TASK.md` — **Phase 4**: write `Results/RESULT.md` + `Retro/LESSONS.md`, then `hv_ship`; confirm PRs

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
/loop Continue executing tasks in Orchestrate/ORCH.md under
planning/workflow-exec/cloud-manager/gloppy-snapper/. Start by running hv_status
per deck.md. For each unchecked task: read the task file, do the work, run the
matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check
the box. When all boxes are checked write Results/RESULT.md and
Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy

- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator
- Task 0001's MCP-level test cases require an operator `/mcp` reload after the dist rebuild — pausing there for the operator is expected, not a failure

## End-of-workflow outputs (write BEFORE hv_ship)

- `Results/RESULT.md`
- `Retro/LESSONS.md`
