# Test: End-to-end smoke (postgres)

## Test ID
`0010-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0010-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After the Apply call has been issued. Most checks are post-completion; one (TC-002) is during.

## Test Cases

### TC-001: VM provisioned
- **Check**: `mcp__cloud-manager-mcp__vm_get(smoke-postgres VM id)`
- **Pass**: orchestrationStatus=2, machineStatus=0
- **Fail**: Phase 0001-equivalent infrastructure regression; investigate

### TC-002: live log appears
- **Check**: open run page during run
- **Pass**: lines stream within 30s of starting
- **Fail**: Phase 0008 regression — debug there

### TC-003: run Succeeded
- **Check**: `GET /api/v1/run/{run_id}` after expected duration
- **Pass**: status=Succeeded, stats.failures=0
- **Fail**: read error_message + log file

### TC-004: secrets surfaced via API
- **Check**: `secrets_published` jsonb on the run row
- **Pass**: ≥3 entries with `path` matching `cloudmanager/vm/instances/{vm_pub}/{pb_pub}/postgres/...`
- **Fail**: Phase 0007 regression

### TC-005: secret reveal in UI works
- **Check**: click Reveal on `admin_password` in run page
- **Pass**: real-looking password rendered
- **Fail**: 4xx → check `/run/{id}/secret` endpoint

### TC-006: VM SSH + postgres present
- **Check**: 
  ```bash
  # use the cached SSH key OR pull from Vault directly
  ssh cloudmanager@<vm_ip> "sudo -u postgres psql -c '\l'"
  ```
- **Pass**: psql lists databases including the one specified in vars_override
- **Fail**: postgres not running, or DB not created — confirm playbook actually ran

### TC-007: connection_uri actually works
- **Check**: 
  ```bash
  URI=$(vault kv get -format=json cloudmanager/vm/instances/<vmpub>/<pbpub>/postgres/connection_uri | jq -r '.data.data.value')
  psql "$URI" -c 'SELECT version()'
  ```
- **Pass**: returns a postgres version string
- **Fail**: URI malformed, or password mismatch — the playbook + Vault writes diverged

### TC-008: child token revoked
- **Check**: `vault token lookup -accessor <accessor from logs>` (or grep worker log for the accessor)
- **Pass**: 403/404
- **Fail**: Phase 0007 regression

## Scoring
All eight. **Do not destroy the VM.**

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
