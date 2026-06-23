# RESULT — roadside-lark: systemd-creds secret injection for the controller

**Date:** 2026-06-23 · **Deck:** cloud-manager · **Orchestration:** roadside-lark
**Git execution branch:** `socky-chickencoop` (minted from origin/hive in Task 0000)

## Outcome — SUCCESS
The controller's services (`cloud-manager-api`, `vorch_service`, `porch`) now read
`VAULT_TOKEN`, `DBPASSWD`, and `RABBITMQ_PASSWORD` from **systemd encrypted
credentials** instead of plaintext unit/env files. PG and RabbitMQ passwords were
rotated to fresh 32-char values during the cutover, stored in Vault, and injected
as `.cred` files. No secret remains in any unit, `api.env`, or `config-server.yaml`;
`systemctl cat` is clean; the API is Healthy; an end-to-end VM provision succeeded.

## Backups (taken before cutover)
- **pg_dump:** `/var/backups/cloud-manager/clouddb-20260623T222127Z.sql` (1,135,098 bytes)
- **Vault (logical export of the `cloudmanager` KV mount, 46 secrets):**
  `/var/backups/cloud-manager/vault-cloudmanager-export-20260623T222127Z.json` (14,946 bytes)
- A raft snapshot was not possible (no operator/root token on the box) — see
  `Retro/FIX-003.md` for the equivalent recoverable backup + command-level rollback.
- Pre-edit unit/config backups: `*.cutover-bak` beside each edited file
  (`cloud-manager-api.service`, `porch.service`, `api.env`, `config-server.yaml`).

## What changed (by repo)
| Repo | Change |
|------|--------|
| `vm-infra/secret-management` | `scripts/cm-seed-secrets` (reuse-or-generate → apply → Vault → `systemd-creds encrypt --with-key=host`); `scripts/cm-cutover.sh` (live runbook); `systemd/<unit>.service.d/10-credentials.conf` drop-ins + README |
| `cloud-manager-api` | `Program.cs`: `AddKeyPerFile($CREDENTIALS_DIRECTORY)` layered above env + bridge to env readers; `RABBITMQ_PASSWORD` wired into `amqpConfig.Password` |
| `vorch-service` | `internal/credenv` + two `main.go` call sites — hydrate `VAULT_TOKEN` and AMQP password from `$CREDENTIALS_DIRECTORY` (plan's "Go already honours VAULT_TOKEN_FILE" was wrong) |

Vault paths (new, additive): `cloudmanager/controller/postgres`, `cloudmanager/controller/rabbitmq`.
Cred files: `/etc/cloud-manager/creds/{vault-token,db-password,rabbitmq-password}.cred` (0600, root).

## Verification (Test 0004 + 0005)
- `systemctl is-active cloud-manager-api vorch_service porch` → **active, active, active**
- `systemctl cat` of all three → **no** `P@ssw0rd` / `DBPASSWD=` / `RABBITMQ_PASSWORD=` / `hvs.` values
- Decrypted creds present per-service at `$CREDENTIALS_DIRECTORY` (0400)
- `cloud_health_check` → **Healthy**; API `GET /api/v1/Host/list` → **200**
- Rotation real: old `P@ssw0rd!` → PG `auth failed`, RabbitMQ `401`; new Vault values → PG ok, RabbitMQ `200`
- Service journals since cutover → **0** auth-error lines
- **End-to-end provision:** `roadside-lark-verify` (Jammy) → `orchestrationStatus: 2`, IP `10.0.150.80`;
  SSH key written to `cloudmanager/vm/instances/vm_EVQ055J670/ssh` (546 chars) via the cred token; VM torn down (404).

## Improvisations (FIX files)
FIX-001 SIGPIPE in `gen_password` · FIX-002 `%d` not expanded in `Environment=` ·
FIX-003 raft snapshot impossible → logical export · FIX-004 `psql -c` no `:var`
interpolation · FIX-005 quoted `Environment="VAULT_TOKEN=…"` not stripped.

## Pull requests
_Filled after `hv_integrate`:_

| Repo | PR |
|------|----|
| _pending hv_integrate_ | |
