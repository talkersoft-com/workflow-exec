# Orchestration: cloud-manager/jazzy-herring

## Objective
Ship porch — a new Go binary in the `vorch-service` repo (`cmd/porch/main.go`) that owns ansible orchestration (playbook runs, cancels, collection install/remove, healthprobe). `cmd/vorch-service/main.go` keeps only VM/libvirt responsibilities. Retire the Python `cloud-manager-worker` so porch is the sole consumer on `playbook-runs`. After ship: funky-vole's per-run scoped Vault token flow works on 100% of triggers, zero Python in the runtime stack.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/charmed-panda/PLAN.md`
- `../../../workflow-fragments/common/toolkit/hive-deck.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status (confirm jazzy-herring) + read PLAN.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: Module rename — `vorch-service/go.mod` from `module main` to a real path; update every import
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: Package carve-out — move `app.go` to `cmd/vorch-service/main.go`; create `cmd/porch/main.go` scaffold; move handlers/messagesub/healthprobe to `internal/{vm,ansible,vmsub,ansiblesub,amqp,pub,models,vault}`; **verify** assignment-tracking SQL presence in `playbook_run_handler.go`
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: Porch wires consumers + healthprobe — `cmd/porch/main.go` registers `playbook-runs`, `playbook-run-cancel`, `collection-installs`, `collection-removes` + starts healthprobe goroutine; **verify** API endpoint mismatch for healthprobe (PATCH `/api/v1/admin/feature-flags/playbooks` vs `FeatureFlagsController` route) — if broken, ship API fix in same plan
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: Vorch loses ansible — `cmd/vorch-service/main.go` only registers VM consumer; `go list -deps` verified clean both directions
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: Assignment tracking + log redaction parity — if Phase 0002 found assignment SQL absent, implement; add Go-side `hvs\.[A-Za-z0-9_\-]+` regex log scrubber
- [ ] `Tasks/0006-TASK.md` — **Phase 6**: Install scripts + systemd unit — `cloud-manager-api/scripts/porch/*.py` mirroring vorch; `/etc/systemd/system/porch.service` template; `/etc/cloud-manager/porch.env` template; publish porch binary to `/kvm-automator/porch`
- [ ] `Tasks/0007-TASK.md` — **Phase 7**: Cutover — execute the 8-step sequence from PLAN.md; one consumer on `playbook-runs` (porch)
- [ ] `Tasks/0008-TASK.md` — **Phase 8**: Retire the Python worker — stop/disable/remove systemd unit + `/opt/cloud-manager-worker` + pipx venv + worker.env
- [ ] `Tasks/0009-TASK.md` — **Phase 9**: End-to-end verification — pg-on-postgres-test run goes terminal-success; vault secrets at expected path; `vm_playbook_assignments` updated; zero `hvs.*` in porch journal
- [ ] `Tasks/0010-TASK.md` — **Phase 10**: Write Results/RESULT.md + Retro/LESSONS.md; hv_ship

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
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/workflow-exec/cloud-manager/jazzy-herring/Orchestrate/ORCH.md. Start by running hv_status then hv_next per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
