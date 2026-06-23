# Orchestration: cloud-manager/roadside-lark — systemd-creds secret injection

## Objective
The controller's services (cloud-manager-api, vorch_service, porch) read VAULT_TOKEN,
DBPASSWD, and RABBITMQ_PASSWORD from systemd encrypted credentials instead of plaintext
unit/env files. An idempotent `cm-seed-secrets` script auto-generates/reuses the passwords,
applies them, stores them in Vault, and encrypts them into `.cred` files. Done = no secret in
any unit/env file, `systemctl cat` clean, and an end-to-end VM provision still succeeds.

## Inputs (read before Task 0000)
- `../init.md`
- `../deck.md`
- `../../../../workflow-plans/cloud-manager/roadside-lark/PLAN.md`

## Task list
Check the box when the task is implemented AND its test passes.

- [x] `Tasks/0000-TASK.md` — Phase 0: hv_status + hv_init/hv_next
- [x] `Tasks/0001-TASK.md` — Phase 1: cm-seed-secrets script (vm-infra/secret-management)
- [x] `Tasks/0002-TASK.md` — Phase 2: API AddKeyPerFile config provider
- [x] `Tasks/0003-TASK.md` — Phase 3: systemd unit wiring (+ vorch/porch RabbitMQ cred)
- [x] `Tasks/0004-TASK.md` — Phase 4: live cutover (backup -> seed -> redeploy -> restart -> strip plaintext)
- [x] `Tasks/0005-TASK.md` — Phase 5: verify (services up, systemctl cat clean, VM provisions); then Results + Retro/LESSONS, then hv_integrate

## Execution steps
1. Read all inputs before starting Task 0000.
2. For each unchecked task in order: read task -> do work -> run matching Test -> on fail write `Retro/FIX-NNN.md`, fix, re-run -> on pass check the box.
3. When every box is checked, write `Results/RESULT.md` + `Retro/LESSONS.md`, then `hv_integrate`.

## Autonomous execution
```
hv_orchestrate_run  deck: "cloud-manager"  branch: "roadside-lark"
```

## Improvisation policy
- One FIX file per distinct failure (FIX-001, FIX-002, ...). Never silently retry.
- If a failure cannot be recovered after two attempts: STOP and surface to the operator.

## Hard constraints
- Idempotent seeder: reuse-from-Vault if present, else generate; NEVER regenerate the Vault token; NEVER clobber a stored password.
- Back up BEFORE the live cutover: Vault snapshot (`snapshot-vault.py`) + `pg_dump clouddb`.
- Vault tokens never appear in logs or run output (use `internal/redact` for new vorch/porch log paths).
- RabbitMQ rotation must reach ALL three consumers (API, vorch, porch).

## End-of-workflow outputs (write BEFORE hv_integrate)
- `Results/RESULT.md`
- `Retro/LESSONS.md`
