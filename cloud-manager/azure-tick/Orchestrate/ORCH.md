# Orchestration: cloud-manager/azure-tick

## Objective
Close every porch follow-up from jazzy-herring. End state: pg-on-postgres-test installs postgres-16 on the VM, the `app` DB exists with `appuser`, and the generated password lands at `cloudmanager/playbook-secrets/<runId>/postgres-test/postgres` in Vault. Porch logs + journal show zero `hvs.*`. `porch.service` has `ProtectHome=yes`. No `--project-dir` flag in porch's exec call.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/nutty-cobbler/PLAN.md`
- `../../cloud-manager/jazzy-herring/Retro/FIX-002.md`
- `../../cloud-manager/jazzy-herring/Retro/LESSONS.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — **Phase 0**: hv_status; read PLAN + LESSONS + FIX-002
- [x] `Tasks/0001-TASK.md` — **Phase 1**: EF migration `ip_address` on `vm.virtual_machines`; entity + DTO + AutoMapper
- [x] `Tasks/0002-TASK.md` — **Phase 2**: `UpdateIpAddressAsync` service method + `PATCH /api/v1/vm/{publicId}/ip-address` controller; GET responses include `ipAddress`; target DTO surfaces VM IP
- [x] `Tasks/0003-TASK.md` — **Phase 3 (Gap 2)**: `PlaybookMaterializationService` writes every `RelativePath` under `project/`
- [x] `Tasks/0004-TASK.md` — **Phase 4 (Gap 1 server-side)**: Vorch `CreateVMCommand` PATCHes API with the VM IP after successful create
- [x] `Tasks/0005-TASK.md` — **Phase 5**: `scripts/porch/backfill-vm-ip-addresses.py` — `virsh domifaddr` per row, PATCH API
- [x] `Tasks/0006-TASK.md` — **Phase 6 (Gap 1 runner-side)**: Porch writes `inventory/hosts` from `target.IpAddress/VMName`, passes `--inventory`, fails target on empty IP
- [x] `Tasks/0007-TASK.md` — **Phase 7 (Gap 2 close-out)**: Porch drops `--project-dir <workdir>` workaround
- [x] `Tasks/0008-TASK.md` — **Phase 8 (Gap 3)**: `redactingWriter` → `internal/redact/`; applied to `cmd.Stderr` + stdout scan loop + log publisher; unit test
- [x] `Tasks/0009-TASK.md` — **Phase 9 (Gap 4)**: `install-porch-service.py` prefers apt > pipx --global; sets `ANSIBLE_RUNNER_BIN`; flips `ProtectHome=yes`; uninstall cleans FIX-001 symlinks
- [x] `Tasks/0010-TASK.md` — **Phase 10**: Build + deploy (publish API; build porch; apply migration; run backfill; restart services)
- [x] `Tasks/0011-TASK.md` — **Phase 11 (Gap 5)**: SQL cleanup of stale Queued runs; record counts in CHANGE-0011
- [x] `Tasks/0012-TASK.md` — **Phase 12**: E2E verify pg-on-postgres-test (postgres-16 installed, Vault path populated, zero `hvs.*`) — conditional pass; see Retro/FIX-002 (playbook-authoring + stale-revision gaps out of scope)
- [ ] `Tasks/0013-TASK.md` — **Phase 13**: Write Results/RESULT.md + Retro/LESSONS.md; hv_ship

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
/loop Continue executing tasks in /home/todd/workspace/cloud-manager/planning/workflow-exec/cloud-manager/azure-tick/Orchestrate/ORCH.md. Start by running hv_status then hv_next per deck.md. For each unchecked task: read the task file, do the work, run the matching Test file. On failure write Retro/FIX-NNN.md and retry. On pass check the box. When all boxes are checked write Results/RESULT.md and Retro/LESSONS.md — both BEFORE calling hv_ship — then ship and stop.
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## End-of-workflow outputs (write BEFORE hv_ship)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
