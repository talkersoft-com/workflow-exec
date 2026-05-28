# Lessons: cloud-manager/funky-vole

## What would have helped

### Plan should have verified Vault policy capabilities, not assumed them
The nimble-orangutan PLAN (line 20) asserted "The deployed cloudmanager token already has full read/write on `cloudmanager/*` — it can mint child tokens via `auth/token/create`." This was wrong — sudo on `cloudmanager/*` doesn't extend to `auth/*` or `sys/policies/*`. Discovered during A.6 verification when the live trigger returned 403 from Vault. A 30-second `curl` to `/v1/sys/capabilities-self` during planning would have caught this. Going forward: **for any plan that mints/writes against Vault paths outside the mount, capture an explicit "verified capabilities" line citing the response from sys/capabilities-self.**

### Vault TLS cert / SAN handling
Vault is served at `https://ubuntu-server.talkersoft.com:8200` but the cert is a real talkersoft.com cert whose SAN list doesn't cover the subdomain. Go (vorch) and curl-with-`-k` both worked; .NET strict TLS did not. Fixed by accepting any cert in the HttpClient handler (FIX-001) — the tradeoff is acceptable because the API and Vault are co-located on the same host, but the right long-term fix is to enroll `ubuntu-server.talkersoft.com` in the cert SAN list.

### Publish + copy, not just build
`dotnet build` puts assemblies in `src/CloudManager.API/bin/Release/net8.0/`, but the systemd unit runs `/opt/cloud-manager-api/CloudManager.API`. The first restart appeared to pick up my change because the journal stack frames still pointed at workspace source paths (Release PDB), but the runtime DLL was old. **For service-restart verification, always do `dotnet publish -o <tmp>` and `sudo cp -r <tmp>/* /opt/<service>/` before restarting.** Build is not enough.

### Trigger flow's "first SaveChanges → mint → second SaveChanges → publish" is correct
The order matters: the run row needs to be inserted first so `run.PublicId` is generated and can be used as the unique policy name. The mint then fills `VaultRunToken` and `VaultSecretsPrefix`. Only after the second SaveChanges does AMQP publish — that means a Vault mint failure leaves a row in Queued state with empty vault columns and **no orphan run on the vorch side.** This is the desired failure mode. Sweeper correctly ignores it (no token to revoke), CancelAsync correctly no-ops the Vault revoke (token empty), and the operator can cancel or delete the row at leisure. Don't tempt yourself into "cleaning up" the Queued row on mint failure — leaving it visible to the operator is the right call.

### Redaction needs a wrapper, not a decorator (Scrutor wasn't in deps)
The initial sketch used `builder.Logging.Services.Decorate<>(...)` which requires Scrutor. The shipped impl iterates `builder.Services` and replaces each `ServiceDescriptor` whose `ServiceType == typeof(ILoggerProvider)` with a wrapper that constructs a `RedactingLoggerProvider` around the original. Slightly more code but no new NuGet dep.

### Variable shadowing in TriggerMultiVmAsync
The new `EnforceRequirementsAsync` returns a `missing` collection (unmet requirements), while the existing code already had a local `missing` (unknown VM ids in the request). Renamed the existing one to `missingVms` to avoid the shadow. Worth a grep on `missing` whenever adding new enforcement steps.

## Fix files written
- `FIX-001.md` — Vault TLS handler accepts the talkersoft.com cert SAN mismatch. Code shipped.
- `FIX-002.md` — cloudmanager policy must be extended with `auth/token/create`, `auth/token/revoke`, `sys/policies/acl/playbook-run-*`. `cloudmanager.py` updated in this PR; **operator must run `./vault-cli cloudmanager` (or `vault policy write cloudmanager -`) with a root token to re-apply the live policy.** Until that step is done, all run triggers will return 503 "Vault unavailable, try again" — which is intentional graceful degradation.

## Deviations from plan
- **Plan said `playbook_runs.host_id` is separate** — followed; no host_id column added. Confirmed in DB.
- **Plan implied root-equivalent Vault creds** — they don't exist. Filed FIX-002 with the gap; `cloudmanager.py` updated to be correct going forward. Verification matrix in A6-vault-audit.md shows everything we can verify without a root token.
- **TLS handler accepts any cert** rather than pinning the talkersoft.com cert thumbprint. Lower-effort, acceptable for co-located deployment, noted as a follow-up. See [[follow-up-vault-san]].

## Follow-ups for the next plan
- `[[follow-up-vault-policy-rollout]]` — operator step + automation to re-apply the cloudmanager policy across all environments.
- `[[follow-up-vault-san]]` — fix the talkersoft.com cert SAN list so we can drop the "accept any cert" handler.
- `[[follow-up-playbook-runs-host-id]]` — add the host_id column to `playbook_runs` (was deliberately deferred per plan; should be its own short plan).
- `[[follow-up-trigger-row-cleanup]]` — cron or sweeper variant that hard-deletes Queued runs older than 24h with no vault token + no AMQP publish.
- `[[follow-up-multi-controller-real]]` — currently `ResolveControllerHostIdAsync` returns the first controller deterministically. When >1 controller exists, plan needs (a) a "controller selector" honored at trigger time (today's selectedController slice is UI-only) or (b) per-VM controller routing.
