# Execution: cloud-manager/syrupy-jadeplant

## Objective
A freshly provisioned VM boots **SSH-only by default**: cloud-init enables `ufw` with port `22`
allowed from anywhere and everything else inbound denied, applied in a lockout-safe order that never
drops the provisioning SSH session. Each exposing service playbook opens **its own** port to the
trusted VM subnet only — Postgres `5432` and RabbitMQ `5672` (+ `15672` if wanted) allowed from
`10.0.150.0/24` + `10.0.144.140/32`, mirroring the existing `pg_hba_sources`. Peer VMs keep direct
on-subnet DB access; off-subnet/operator access goes via SSH tunnel. The empty-secrets user-data
path stays byte-stable apart from the ufw block. No api/web/db changes; no migration;
vorch-service unchanged. Verified end-to-end against a real provision on the hypervisor.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/syrupy-jadeplant/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: vorch-lib — cloud-init ufw baseline in `buildUserData` runcmd + golden/unit test
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: service playbooks — Postgres + RabbitMQ open their port to `{{ ufw_sources }}` (trusted subnet)
- [ ] `Tasks/0003-TASK.md` — **Final phase**: E2E (plain + Postgres VM) + build vorch + manual redeploy; write Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "syrupy-jadeplant"
```

Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- Two failed recoveries on the same failure → stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
