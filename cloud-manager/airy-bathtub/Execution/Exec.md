# Orchestration: cloud-manager/airy-bathtub

## Objective
Every newly provisioned VM boots behind a default-deny host firewall: ufw enabled with
`22/tcp` allowed from anywhere, default deny incoming / allow outgoing — installed by the
vorch cloud-init user-data builder. Each service playbook then opens **only its own** port
to the trusted subnet (`10.0.150.0/24` + `10.0.144.140/32`): Postgres gains a ufw rule for
`5432`; RabbitMQ tightens its existing open-to-anywhere rule to subnet-only for `5672` +
`15672`. Seed YAML is the source of truth and is synced to the live DB with
`import-playbooks.py`. Done = a plain VM is SSH-only from off-subnet, a Postgres VM accepts
`5432` from an on-subnet peer but not from off-subnet, the DB matches the seed YAML, and no
already-running VM was touched.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/airy-bathtub/PLAN.md` (the approved plan)

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_next/hv_init; record branch
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: vorch-lib — cloud-init ufw baseline (lockout-safe block + golden user-data test)
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: seed playbooks — per-service subnet rules (pg add; rabbitmq tighten) + import-playbooks.py sync
- [ ] `Tasks/0003-TASK.md` — **Final phase**: E2E + build + manual vorch redeploy + deploy; write Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs above before starting Task 0000.
2. For each unchecked task in order:
   a. Read the task file.
   b. Do the work.
   c. Run the matching Test file (`Test/NNNN-TEST.md`).
   d. On failure: write `Retro/FIX-NNN.md`, apply the fix, re-run the test.
   e. On pass: check the box, move to the next task.
3. When every box is checked, the workflow is complete.

## Build
Run the relevant build before checking a box (script: `cloud-manager-mcp/.cicd/build-cloud-manager.py`):
- `vorch-lib` (Go) changed → build vorch per its repo (Phase 0003 records the manual build + redeploy cmd).
- `cloud-manager-api` (.NET) changed → seed YAML edits are data, not code; no .NET rebuild
  required to change the seeds. The DB sync is done by `import-playbooks.py`, not a build.
- If any TS/.NET/web source actually changes → `--target <api|mcp|web|all>`. A task is not
  done until its build passes cleanly.

## Deploy (after all phases pass, BEFORE hv_integrate)
- vorch: **manual build + redeploy on the hypervisor** — record the exact command in
  `Results/RESULT.md`. This is the only way the new cloud-init baseline reaches future VMs.
- Seed playbooks: `python3 cloud-manager-mcp/.cicd/import-playbooks.py` against the live API
  to PATCH the pg + rabbit DB rows. Confirm PATCH/no-op output; the DB must match the seed YAML.
- No `cloud-manager-api`/`-web` service redeploy is required (no api/web source change).
- A failed deploy is a stop condition: write `Retro/FIX-NNN.md`, resolve, retry.

## Autonomous execution
Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "airy-bathtub"
```
Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...).
- Never silently retry — write the FIX file first, then apply the fix.
- If a failure cannot be recovered after two attempts: stop and surface to the operator.

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
