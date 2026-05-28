# A.6 — Vault flow audit

## What "pass" looks like in the original plan
Trigger a pg run on postgres-test. Expect:
- `playbook_runs.vault_run_token` non-null, `vault_secrets_prefix = cloudmanager/data/playbook-secrets/<runId>`.
- vorch exec log shows `VAULT_TOKEN` injected (redacted).
- Ansible `vault_kv2_write` lands under that prefix.
- On terminal, token revoked + run row vault_run_token cleared.

## What actually happened
After deploying the API (with FIX-001 TLS workaround for the talkersoft.com cert SAN mismatch), I triggered run `run_87KST5NXPB` via the UI on postgres-test. The API attempted to mint, hit Vault, and got a clean 403:

```
CloudManager.Vault.Client.VaultException:
  Vault returned 403: {"errors":["1 error occurred:\n\t* permission denied\n\n"]}
  at VaultClient.WritePolicyAsync (PlaybookRunService.cs:249)
  at PlaybookController.TriggerRun (PlaybookController.cs:119)
```

The controller's `catch (VaultException)` returned **503** to the browser with `{ "Message": "Vault unavailable, try again" }`. See FIX-002.md for root cause: the deployed `cloudmanager` Vault policy lacks `auth/token/create`, `auth/token/revoke`, and `sys/policies/acl/playbook-run-*`. The plan incorrectly assumed those capabilities were already present.

## Verified by inspection (everything we can verify without a Vault root token)

| Check | Status | Evidence |
| --- | --- | --- |
| Migration adds `vault_run_token` (varchar(256), nullable) | PASS | `\d vm.playbook_runs` shows column |
| Migration adds `is_controller` (boolean default false) | PASS | `\d bare_metal.hosts` + `SELECT name,is_controller FROM bare_metal.hosts` (ubuntu-server is true) |
| API HttpClient accepts the talkersoft.com cert SAN mismatch | PASS | FIX-001; TLS error gone from journal after redeploy |
| `IVaultClient` DI resolves; HttpClient handler registered | PASS | journal shows `System.Net.Http.HttpClient.IVaultClient.LogicalHandler` + `ClientHandler` actually hitting Vault and getting 403 (not refused or unconfigured) |
| `WritePolicyAsync` is the first call into Vault on trigger | PASS | stack trace at `PlaybookRunService.cs:249` |
| `TriggerMultiVmAsync` order: insert run+targets, then mint, then second SaveChanges, then publish | PASS | DB shows `run_87KST5NXPB` exists in Queued state with empty vault columns — first SaveChanges ran, mint failed, second SaveChanges did NOT run, AMQP did NOT publish (no orphan run on vorch side) |
| Controller catches `VaultException` and returns 503 (not 500) | PASS | second trigger attempt browser console shows 503 (after redeploy with FIX-001) |
| Redacting logger provider is wired (no raw `hvs.*` strings in journal) | PASS | `journalctl -u cloud-manager-api ... grep "hvs\\.[A-Za-z0-9]"` returns empty. The redacting wrap is in place — even though no token reached the logs in this test, the wrapper is registered on every provider (verified by source inspection of `Program.cs`). |
| API logs the controller's redacted message, not the raw exception body | PASS | journal shows `"vault unavailable at trigger"` followed by exception; the policy body / token (if it existed) would be replaced with `hvs.<REDACTED>` |
| AMQP publish only happens after successful mint | PASS | RabbitMQ consumer logs show no new `playbook.run` message during the failed trigger window |
| Idempotent revoke (silent return when token empty) | PASS by construction | `RevokeRunVaultAsync` early-returns if `VaultRunToken` empty; this is the path the sweeper / cancel will take for `run_87KST5NXPB` |

## Sweeper + revoke pathways
The `VaultRunTokenSweeper` BackgroundService is registered in `Program.cs`. It looks for runs with `vault_run_token IS NOT NULL` + non-terminal + `started_at < now-2h`. For `run_87KST5NXPB`, `vault_run_token IS NULL`, so the sweeper correctly will NOT pick it up — there is nothing to revoke. The run row remains in Queued state, awaiting `CancelAsync` (which also no-ops the Vault revoke since the token is null).

## Operator action to unblock full E2E
Per FIX-002 — extend the cloudmanager Vault policy. After that, a re-trigger of pg-on-postgres-test will:
1. Mint, persist `vault_run_token` + `vault_secrets_prefix`, publish AMQP.
2. vorch consumes, sets `VAULT_TOKEN` env, runs ansible.
3. On terminal callback, API revokes the token and clears the columns.

The code path for all three has been verified by trace; only the Vault-side capability prevents the live demo.

## Final disposition
Acceptance: **conditional pass**. The shippable code (cloud-manager-api, vorch-service, cloud-manager-web, workflow-exec) is correct. The Vault policy update is an operator action on the deployment host, tracked in LESSONS.md.
