# Orchestration: cloud-manager/primordial-crystalball

## Objective
Inject Secret Binding values into new VMs as **systemd encrypted credentials** at provision time.
Done means: instantiating a blueprint that has an attached Secret Binding produces a VM where, on
first boot, the secret is sealed into `/etc/credstore.encrypted/<credName>` (plaintext shredded)
and the appliance unit reads it via `LoadCredentialEncrypted` — with no secret in any log, API
response, or persisted plaintext; a `vm.vm_secret_bindings` usage row recorded; and a VM with no
bindings provisioning exactly as before.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/primordial-crystalball/PLAN.md` — the authoritative design

## Dependency
**#1.5 (Secret Manager) must be merged before executing this.** Task 0000 verifies the `Secret`
entity / `secret_id` column / `vm.vm_secret_bindings` table exist before proceeding — if they do
not, STOP and surface that #1.5 hasn't landed.

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record branch; verify #1.5 schema present
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: API — `ReadKvSecretAsync` (KV v2) on IVaultClient/VaultClient
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: API — resolve attached bindings at instantiate → read Vault → resolved secrets
- [ ] `Tasks/0003-TASK.md` — **Phase 3**: Contract — `Secrets` payload on VmCommandData (C#) + createvm.VM (Go), YAML lockstep
- [ ] `Tasks/0004-TASK.md` — **Phase 4**: vorch — cloud-init `write_files` + first-boot `systemd-creds encrypt` + shred
- [ ] `Tasks/0005-TASK.md` — **Phase 5**: API — populate `vm.vm_secret_bindings` on provision success
- [ ] `Tasks/0006-TASK.md` — **Phase 6**: E2E + build + deploy; write Results + Retro/LESSONS, then hv_integrate

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
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "primordial-crystalball"
```

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, …)
- Never silently retry — write the FIX file first, then apply the fix
- If a failure cannot be recovered after two attempts: stop and surface to operator

## Project conventions (apply to every task)
- **VMs are NEVER Vault clients.** Secret values NEVER appear in logs, run output, API responses,
  or MCP output — use vorch's `internal/redact` writer for any new log path.
- **Values ride in `write_files` (base64), never in `runcmd`/shell args** (shell-injection
  guardrail). `credName` is charset-validated before use in paths.
- **camelCase JSON; public_ids only** (use `PublicIdByGuidAsync` / `GuidByPublicIdAsync`).
- **C# ↔ Go YAML lockstep** for the `Secrets` payload.
- **Backward compatible:** no bindings → empty list → provisioning identical to today.

## Build & deploy (mandatory, BEFORE hv_integrate)
- Build per changed target: `python3 .cicd/build-cloud-manager.py --target {api|all}`; build the Go
  repos (`vorch-lib`, `vorch-service`) per their own build.
- Deploy api: `python3 .cicd/deploy-cloud-manager.py --target api`.
- **vorch redeploy is manual** (hypervisor host) — see `../deck.md` Manual steps; record the exact
  command in `Results/RESULT.md`.

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
