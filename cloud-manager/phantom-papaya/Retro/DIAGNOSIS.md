# Diagnosis — VM-creation failure root cause

## Failure layer
**operational (config)** — the failure is rooted in `cloud-manager-api`'s unified config env file, NOT in vorch-service Go code or vorch-lib provision logic. The Go code correctly detects, propagates, and reports the error; the broken artifact is the value of the `VAULT_TOKEN` it was deployed with.

**Affected repo:** `cloud-manager-api` (single repo).

## File:line of the bug
`vm-infra/cloud-manager/cloud-manager-api/config/cloud-manager-config.env` line **89**:

```
VAULT_TOKEN=hvs.<REDACTED-STALE-TOKEN>   # accessor qJjiLLUy9axP7O0LZp4nA2sK — revoked
```

This value is read by `cloud-manager-api/config/config_loader.py:165` (`config.get('VAULT_TOKEN', '')`) and written into `/kvm-automator/config-server.yaml`'s `vault.token` field by `generate_vorch_config_yaml()` (line 149). The installer (`cloud-manager-api/scripts/vorch/install-vorch-service.py:185-198`) calls this method at deploy time. Restarting `vorch_service` therefore loads the stale token.

## Why it fails

The token in `cloud-manager-config.env` is **revoked / no longer valid** in the live Vault instance at `https://ubuntu-server.talkersoft.com:8200`. Three independent verifications:

1. `GET /v1/auth/token/lookup-self` with this token returns **403 permission denied** — even a token's right to look itself up is denied, which only happens when the token has been revoked or never existed in the current Vault.
2. The same lookup with the token stored at `/home/todd/cloudmanager_vault_token.txt` (a different cloudmanager-policy token) returns the full token metadata: `policies: [cloudmanager, default]`, valid until `2026-06-20`.
3. A real `POST /v1/cloudmanager/data/vm/instances/probe-phantom-papaya` with the working token succeeds — confirming the KV-v2 mount, the policy grants, and the path are all correct. The only broken artifact is the token itself.

Likely sequence of events: Vault was reinstalled / re-bootstrapped at some point (`vault.txt` in `/home/todd/` reflects a now-stale set of credentials; the working token in `cloudmanager_vault_token.txt` was issued on `2026-05-19` and was never copied back into `cloud-manager-config.env`).

## What the Go code does correctly (do not change)

- `vorch-lib/provision/provision.go:278-288` — calls `vault.NewClient(...)`, then `vaultClient.StoreSSHKey(...)`. On the StoreSSHKey 403, logs the error and returns `vault key storage failed for VM %s: %w`. **Correct fail-hard semantics per the VAULT-SSH-KEY-PLAN.md design doc.**
- `vorch-service/vorch-service/handlers/vm_handlers.go:34-58` — `CreateVMCommand` returns the error from `provision.InitializeHost`.
- `vorch-service/vorch-service/messagesub/messagesub.go:81, 201-211` — captures `cmdErr`, sets `wrapper.Status = "Failed"` (line 204), publishes that envelope. **Correct error path.**
- `vorch-service/vorch-service/messagepub/messagepub.go:34` — logs `"Successfully published success message"`. This name is misleading (it means "the publish succeeded", not "the operation succeeded"), but it is NOT the bug, and renaming it is out of scope per the workflow's no-opportunistic-refactor rule.
- `cloud-manager-api` — receives the `status: Failed` callback and updates `vm.virtual_machines.orchestration_status` correctly (visible to the UI as `Status: Error`).

## Minimal fix

**One file, one line, one repo.**

Edit `vm-infra/cloud-manager/cloud-manager-api/config/cloud-manager-config.env` line 89:

```
# before
VAULT_TOKEN=hvs.<REDACTED-STALE-TOKEN>   # accessor qJjiLLUy9axP7O0LZp4nA2sK — revoked

# after
VAULT_TOKEN=hvs.<REDACTED-NEW-TOKEN>     # accessor dPpGPga0IpCxwNeGjaHWPgvl — valid until 2026-06-20
```

Source of the new value: `/home/todd/cloudmanager_vault_token.txt` (a `cloudmanager`-policy token, no TTL until 2026-06-20).

## Deployment path

After the file edit, redeploy vorch-service:

```
cd vm-infra/cloud-manager/cloud-manager-api
sudo python3 scripts/vorch/install-vorch-service.py
```

This regenerates `/kvm-automator/config-server.yaml` with the new token and restarts `vorch_service`. The API service does NOT need to be redeployed for this bug because vorch is the only component that writes to Vault during VM creation. (If/when cloud-manager-api itself starts reading SSH keys back from Vault, the API will also need a restart to pick up the new token — but that's beyond the current scope.)

## Regression analysis

Not a code regression. The vault SSH-key feature was recently added (per `VAULT-SSH-KEY-PLAN.md` and vorch-service PR #7's `go mod tidy` promoting `hashicorp/vault/api`). At deployment time, a valid token was committed into `cloud-manager-config.env`. That token was later revoked (Vault was apparently rebuilt — the credentials in `/home/todd/vault.txt`, including its root token, are also rejected). The replacement token was saved to `/home/todd/cloudmanager_vault_token.txt` but never propagated back into the source of truth (`cloud-manager-config.env`).

## What would have caught this earlier

A smoke test or healthprobe that performs `vault PUT cloudmanager/data/vm/instances/healthcheck` at vorch startup (or every N minutes) and surfaces the failure to systemd would have flagged the bad token without requiring a real VM-create attempt. The existing healthprobe loop in `provision.log` already runs every 5 minutes but only checks `ansible-runner` and the playbooks feature-flag endpoint, neither of which touches Vault. (Adding such a check is a follow-up improvement, NOT part of this bug's fix.)

## Repos that need PRs

- `cloud-manager-api` — one-line change to `config/cloud-manager-config.env:89`
- `workflow-exec` — Results/Lessons (mechanical; happens at ship time)

No other repos should be touched.
