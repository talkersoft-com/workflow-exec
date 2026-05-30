# Orchestration: cloud-manager/agile-conch

## Objective

Replace six in-DB SQL triage patches (`fix_*.sql`) we applied to the postgres playbook this session with real code fixes. Done means: a fresh `cloud_playbook_run` against `vm_GY68JPPKDN` (or any newly-provisioned Jammy VM) succeeds with no manual SQL or Vault CLI ops, writes the postgres password to `cloudmanager/data/playbook-secrets/<runId>/<host>/postgres` using the per-run-scoped Vault token cloud-manager-api already mints, and a full `bootstrap-server.py` rebuild cycle reaches the same green state with no operator intervention.

## Inputs

Read these before Task 0000:

- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/poppy-fiddle/PLAN.md`
- `../../../workflow-plans/cloud-manager/poppy-fiddle/deck.md`

## Task list

Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status; confirm `agile-conch` across all repos
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: porch extravars — inject `run_public_id`, fix `secretsPrefix`; rebuild + restart porch
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: VAULT_ADDR FQDN — `envvars` emits the configured `https://ubuntu-server.talkersoft.com:8200`
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: extend `cloud_playbook_update` (API + MCP) if content patches aren't supported yet
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: revert `pg-14-jammy` + `pg-16-noble` playbooks via `cloud_playbook_update` — back to `{{ run_public_id }}`, drop the six dev-only patches
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: `cloud-manager-api/scripts/vault/bootstrap-policies.py` — writes `playbook-run` policy + role; updates `cloudmanager-playbook-runner` read path
- [ ] `Tasks/0006-TASK.md` — **Phase 6**: `cloud-manager-mcp/.cicd/export-playbooks.py` + `import-playbooks.py`; commit `cloud-manager-api/seed/playbooks/`
- [ ] `Tasks/0007-TASK.md` — **Phase 7**: `bootstrap-worker.py` calls `bootstrap-policies.py` + `import-playbooks.py` after `hv init`
- [ ] `Tasks/0008-TASK.md` — **Phase 8**: E2E verify — fresh playbook run end-to-end, no manual SQL or Vault CLI ops
- [ ] `Tasks/0009-TASK.md` — **Phase 9**: teardown + rebootstrap verify — full `bootstrap-server.py` cycle ends green
- [ ] `Tasks/0010-TASK.md` — **Phase 10**: write `Results/RESULT.md` + `Retro/LESSONS.md` (enumerate the six `fix_*.sql` patches and which phase replaced each); `hv_ship`

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

**Option A — MCP (preferred):** Call the following tool:
```
hv_workflow_run  deck: "cloud-manager"  branch: "agile-conch"
```

**Option B — manual:** Type the following command (do not copy-paste):

    workflow execute

⚠️  DO NOT COPY PASTE — type this command manually. The `workflow` keyword only works when typed directly into the prompt.

## Improvisation policy

- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)

- `Results/RESULT.md`
- `Retro/LESSONS.md`
