# Test: Vault scoped tokens + outputs

## Test ID
`0007-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0007-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After worker + API redeploy. Use a fresh VM + fresh postgres assignment.

## Test Cases

### TC-001: child token minted with metadata
- **Setup**: tail the worker log; trigger an Apply
- **Check**: log line shows mint with `vm_public_id` and `playbook_public_id` metadata
- **Pass**: present
- **Fail**: metadata not passed — fix the mint call

### TC-002: child token scoped correctly
- **Check during run**: `vault token lookup -accessor <accessor> | grep policies`
- **Pass**: `policies = [cloudmanager-playbook-write]` (only)
- **Fail**: orchestrator policies leaked into the child — wrong role used

### TC-003: token TTL reasonable
- **Check**: `vault token lookup -accessor <accessor> | grep ttl`
- **Pass**: TTL ≤ 1h (3600s), explicit_max_ttl 2h (7200s)
- **Fail**: TTL bigger than 1h — mint call wrong

### TC-004: playbook can write under prefix
- **Check after run**: `vault kv list cloudmanager/vm/instances/{vm_pub}/{pb_pub}/`
- **Pass**: at least one key exists
- **Fail**: empty → child token couldn't write (policy check) or playbook didn't write (playbook bug)

### TC-005: token revoked after run
- **Check**: `vault token lookup -accessor <accessor>`
- **Pass**: returns 403/404 (revoked)
- **Fail**: token still alive — `try/finally` didn't fire

### TC-006: secrets_published populated
- **Check**: `psql -c "SELECT secrets_published FROM vm.playbook_runs WHERE public_id='run_X'"`
- **Pass**: jsonb array with `path`, `output_key`, `type`, `description` entries
- **Fail**: null or empty array — listing pass didn't run or missed the prefix

### TC-007: Reveal works
- **Check**: in UI, click Reveal on `admin_password`
- **Pass**: a real-looking password appears in a modal; auto-hides after 30s
- **Fail**: empty / error → check the `/run/{id}/secret` endpoint logs

### TC-008: path traversal blocked
- **Check**: `curl 'http://localhost:5250/api/v1/run/run_X/secret?path=cloudmanager/data/vm/instances/OTHER_VM/OTHER_PB/anything'`
- **Pass**: 403 (path outside this run's prefix)
- **Fail**: 200 → orchestrator handing out arbitrary secrets, fix the validator

## Scoring
All eight, including the security check (TC-008). Reset VM after.

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
