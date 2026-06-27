# Execution: cloud-manager/serpentish-shuttlecock

## Objective
Secret-binding **credential injection** works on **both** Ubuntu 22.04 (jammy, systemd 249) and
24.04 (noble, systemd 255). The on-VM seal step becomes **version-aware**: systemd ≥250 keeps
`systemd-creds encrypt` → `/etc/credstore.encrypted/DBPASSWD`; systemd <250 falls back to a
root-only `0600` plaintext credstore file at `/etc/credstore/DBPASSWD`. `/run/cm-seed` is **always**
shredded (success or failure) and a seal failure is **loud**, never silent. The consuming sample
(`tests/hello-db-web`) picks `LoadCredentialEncrypted` vs `LoadCredential` by the VM's systemd
version, and `server.js` stays credential-only. Done = `cloud-manager-api/tests/test_secret_injection.py`
exits **0** with both the jammy and noble cases PASS, run on ubuntu-server, with no plaintext left in
`/run/cm-seed` on either VM. No API/web/DB-schema changes; the haproxy/wildcard-cert path still works.

## Inputs
Read these before Task 0000:
- `../init.md`
- `../deck.md`
- `../../../workflow-plans/cloud-manager/serpentish-shuttlecock/PLAN.md`
- The acceptance test (the spec): `cloud-manager-api/tests/test_secret_injection.py`

## Task list
Check the box when the task is implemented AND its test passes.

- [ ] `Tasks/0000-TASK.md` — **Phase 0**: hv_status + hv_init/hv_next; record the execution branch in deck.md
- [ ] `Tasks/0001-TASK.md` — **Phase 1**: vorch-lib version-aware seal — `injectSecrets` / generated `seal-credentials.sh`: ≥250 → `systemd-creds encrypt`; <250 → root-only `0600` `/etc/credstore/<name>`; always shred `/run/cm-seed`; fail loud. Unit/golden test of both branches
- [ ] `Tasks/0002-TASK.md` — **Phase 2**: sample consumption per-OS — `tests/hello-db-web` `deploy.sh` + `hello-db-web.service` select `LoadCredentialEncrypted` (≥250) vs `LoadCredential` (<250); `server.js` stays credential-only
- [ ] `Tasks/0003-TASK.md` — **Final phase**: E2E verify + deploy — build vorch, manually redeploy `vorch-service` on the host, run `test_secret_injection.py` on ubuntu-server (both cases PASS, exit 0), confirm no plaintext in `/run/cm-seed`; write Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs above before starting Task 0000
2. For each unchecked task in order: read the task file, do the work, run the matching Test file;
   on failure write `Retro/FIX-NNN.md`, fix, re-run; on pass check the box
3. When every box is checked, the workflow is complete

## Autonomous execution

Call the following MCP tool to begin execution:
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "serpentish-shuttlecock"
```

Or type in the prompt:

  exec "workflow"

The agent will call `hv_orchestrate_run` automatically.

## Improvisation policy
- One FIX file per distinct failure, numbered sequentially (FIX-001, FIX-002, ...)
- Never silently retry — write the FIX file first, then apply the fix
- Two failed recoveries on the same failure → stop and surface to operator

## Constraints (do NOT violate — from PLAN.md)
- Do **not** edit `test_secret_injection.py` assertions/gate to force a pass.
- The app gets the DB password **only** from the binding-injected systemd credential — no
  hardcoding, no `PGPASSWORD` env, no extra Vault fetch in the app/playbook, nothing baked into the image.
- Secret values never logged, never in run output, never world-readable; the 22.04 fallback file is `0600 root`.
- `vorch-lib` + the consuming sample only — no API/web/DB-schema changes unless strictly required.
- VMs are never Vault clients (the controller injects).

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
