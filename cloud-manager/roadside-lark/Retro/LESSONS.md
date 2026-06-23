# LESSONS — roadside-lark (systemd-creds secret injection)

## What worked
- **Render-vs-rotate separation in the seeder.** Giving `cm-seed-secrets` a
  `--dry-run` and `--skip-apply` plus path/dir overrides meant Task 0001 could be
  proven non-destructively (idempotent reuse on isolated Vault paths, creds at
  0600, no leak) without touching the live controller before the cutover.
- **Pre-building every artifact** (API publish + both Go release binaries) before
  the cutover kept the actual rotation→restart window to a few seconds.
- **`systemd-run` as a credential test harness.** A transient unit with
  `LoadCredentialEncrypted=` proved the whole inject→decrypt→read path (and the
  Go `credenv` resolver) against real `.cred` files without restarting any
  production service.
- **`--with-key=host`** made the credentials decrypt deterministically despite the
  box's partial TPM2 — no boot-time surprises.

## Plan assumptions that were wrong (verify against code, not the plan)
- The plan/init said "the Go vault client already honours `VAULT_TOKEN_FILE`."
  It does **not**: `vorch-lib/serverconfig` only honours the `VAULT_TOKEN` *env*
  var and reads `amqp.password` straight from `config-server.yaml`. The fix was a
  small `internal/credenv` helper in `vorch-service` (two `main.go` call sites),
  keeping `vorch-lib` untouched as the deck intended. Lesson: confirm the actual
  read path in code before designing the injection.
- The API's RabbitMQ password does **not** come from the `RABBITMQ_PASSWORD` env
  (that env was only used by a startup validation print); the broker connection
  binds `AMQPConfiguration.Password` from appsettings `AMQP:password`. So the
  `AddKeyPerFile` change alone wasn't enough — the canonical `RABBITMQ_PASSWORD`
  key had to be wired into `amqpConfig.Password` explicitly.

## Failures hit and fixed (one FIX file each)
- **FIX-001** `gen_password` SIGPIPE (`… | head` under `pipefail`) — bound the
  random input instead.
- **FIX-002** `%d` is **not** expanded inside `Environment=` values — derive the
  cred path from `$CREDENTIALS_DIRECTORY` (which systemd always exports) instead.
- **FIX-003** No operator/root Vault token on the box → raft snapshot impossible;
  substituted a recoverable logical KV export of the `cloudmanager` mount +
  `pg_dump`, with a documented command-level rollback.
- **FIX-004** `psql -c` does **not** interpolate `:'var'` — feed the statement on
  stdin with `:"role"` / `:'pw'` quoting (injection-safe, off-argv).
- **FIX-005** The strip regex missed the **quoted** `Environment="VAULT_TOKEN=…"`
  in porch.service; `credenv` then preferred the stale env over the cred. Strip
  both quoted/unquoted forms.

## Operational notes for next time
- A drop-in cannot delete a base-unit `Environment=` line — plaintext removal is a
  base-unit edit, done in the cutover with `.cutover-bak` copies for rollback.
- The startup env-check in the API prints `DBPASSWD=✅ <value>` to the journal —
  a pre-existing plaintext-in-logs smell (redaction only covers `hvs.` tokens).
  Out of scope here; worth masking in a follow-up.
- Rotation has a brief window where live PG/RabbitMQ have the new password but the
  not-yet-restarted services hold the old one in memory. Existing connections
  survive (neither drops on password change); the back-to-back restart closes it.
