# Test: Verify install + Vault config

## Test ID
`0001-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0001-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After `install`, `configure`, and `verify` have each exited 0. The test below confirms the post-conditions independently of `verify`'s self-reporting.

## Test Cases

### TC-001: ansible-core callable as install user
- **Check**: `sudo -u cloudmanager ansible --version` exits 0
- **Pass**: version line ≥ 2.16
- **Fail**: pipx install didn't land in cloudmanager's PATH → improvise around shell init

### TC-002: hvac importable from ansible-core venv
- **Check**: `sudo -u cloudmanager ~/.local/pipx/venvs/ansible-core/bin/python -c "import hvac; print(hvac.__version__)"`
- **Pass**: exits 0 with version string
- **Fail**: `pipx inject` was not run or pipx path is non-default

### TC-003: collection installed
- **Check**: `sudo -u cloudmanager ansible-galaxy collection list community.hashi_vault`
- **Pass**: `community.hashi_vault` is in output
- **Fail**: galaxy install failed silently — re-run with `-vvv`

### TC-004: runner policy exists
- **Check**: `vault policy read cloudmanager-playbook-runner` exits 0
- **Pass**: policy body contains `auth/token/create/playbook-run`
- **Fail**: configure step never ran or token lacked perms

### TC-005: scoped policy exists
- **Check**: `vault policy read cloudmanager-playbook-write` exits 0
- **Pass**: policy body contains `cloudmanager/data/vm/instances/+/+/*`
- **Fail**: same as TC-004

### TC-006: token role mints orphan, capped TTL
- **Check**: `vault read auth/token/roles/playbook-run -format=json`
- **Pass**: `orphan: true`, `renewable: false`, `explicit_max_ttl: 7200` (2h in seconds), `allowed_policies: [cloudmanager-playbook-write]`
- **Fail**: configure wrote the role with wrong params — re-run configure (idempotent)

## Scoring
All six must pass. Partial is failure. Do not proceed to Phase 1 until all six pass.

## On Failure
1. Write `Improvise/0001-ANSIBLE-SHENANIGANS-IMPROVISE-NNN.md`
2. Document the failing TC and exact error
3. Adapt and retry the relevant ansible-cli command
4. Re-run this test

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
