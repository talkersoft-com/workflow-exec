# Deck: cloud-manager

## Deck name
`cloud-manager`

## Repos with code changes
| Repo | What changes |
|------|-------------|
| `vm-infra/secret-management` | The `cm-seed-secrets` script (reuse-or-generate → apply → Vault → `systemd-creds encrypt`); systemd unit drop-ins with `LoadCredentialEncrypted=` for api/vorch/porch; the live-cutover runbook. |
| `cloud-manager-api` | One change: `AddKeyPerFile($CREDENTIALS_DIRECTORY)` layered config provider (`env → cred`) so `VAULT_TOKEN` / `DBPASSWD` / `RABBITMQ_PASSWORD` resolve from decrypted cred files. |
| `vorch-service` | RabbitMQ cred wiring — read the rotated RabbitMQ password from a cred (or seeder-updated `config-server.yaml`). |

Repos not listed will be on the feature branch but skipped by hv ship.
`vorch-lib` needs no code change (Go vault client already honours `VAULT_TOKEN_FILE`).

## Branch
Orchestration codename: `roadside-lark`
Git execution branch (Task 0000, minted via hv_next from origin/hive on 2026-06-23): `socky-chickencoop`

## Initialize (Task 0000)
```
hv_status  deck: "cloud-manager"
hv_init    deck: "cloud-manager"
```
If already provisioned on a prior feature branch with all PRs merged:
```
hv_status  deck: "cloud-manager"
hv_next    deck: "cloud-manager"
```

## RabbitMQ three-touch-point decision (Task 0003)
Chose **Option A (systemd cred for all three consumers)**:
- API reads `RABBITMQ_PASSWORD` via `AddKeyPerFile` (cred file).
- vorch + porch read it from `RABBITMQ_PASSWORD_FILE=%d/RABBITMQ_PASSWORD` via the new
  `internal/credenv` helper (the `vorch-service` code change).
- Cutover removes `amqp.password` from `config-server.yaml`.
Result: no plaintext RabbitMQ password in any file (satisfies Test 0003 TC-003).

Note (plan correction): the plan assumed "the Go vault client already honours
`VAULT_TOKEN_FILE`". It does not — `vorch-lib/serverconfig` only honours the
`VAULT_TOKEN` *env* var and reads `amqp.password` straight from `config-server.yaml`.
The minimal fix lives in the `vorch-service` repo (`internal/credenv` + two `main.go`
call sites), so `vorch-lib` is still untouched, consistent with the deck scope.

## Ship
```
hv_integrate  deck: "cloud-manager"
              message: "feat: systemd-creds secret injection for the controller"
              title:   "feat: systemd-creds secret injection for the controller"
              stage:   "exec"
```
