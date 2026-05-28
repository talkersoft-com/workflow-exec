# Task: Run install, configure, verify

## Task ID
`0001-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Run the three `ansible-cli` commands against the production Vault and confirm each completes without error.

## Context
The scripts at `cloud-manager-api/scripts/ansible/` were built in a prior session but never run. The `cloudmanager` user (uid 700) exists locally. Vault is reachable at `https://ubuntu-server.talkersoft.com:8200` and the install-time token sits at `~/ansible-token.txt` (0600). Per the plan, the broad fallback policy `cloudmanager-playbook-write` is the starting point; templated scoped policy is deferred.

## Steps

1. `cd cloud-manager-api/scripts/ansible`
2. `sudo ./ansible-cli install` — answer `y` at the confirmation prompt. Watch for: apt installs, pipx installs of `ansible-core` and `ansible-runner`, `hvac` injected into ansible-core's venv, `community.hashi_vault` collection install, runtime dirs created at `/var/lib/cloud-manager/playbook-runs` and `/var/log/cloud-manager/playbook-runs`.
3. `export VAULT_ADDR=https://ubuntu-server.talkersoft.com:8200`
4. `export VAULT_TOKEN=$(cat ~/ansible-token.txt)`
5. `./ansible-cli configure` — writes the runner policy, the scoped broad policy, the `playbook-run` token role, then mints + revokes a 60s test child token.
6. `./ansible-cli verify` — runs all six smoke checks.

## Acceptance Criteria
- `install` exits 0 and prints "Install complete."
- `configure` exits 0 and prints "Vault configure complete." plus "Role round-trip OK"
- `verify` exits 0 and prints `[PASS]` for all six checks
- Vault confirms manually: `vault policy list | grep -E 'cloudmanager-playbook-(runner|write)'` returns both
- Vault confirms manually: `vault read auth/token/roles/playbook-run` returns the role

## Test
Run `Test/0001-ANSIBLE-SHENANIGANS-TEST.md` after the steps above. Do not mark this task complete until the test passes.

## On Failure
Write `Improvise/0001-ANSIBLE-SHENANIGANS-IMPROVISE-NNN.md` first. Most likely problems:
- pipx venv path not at `~/.local/pipx/venvs/ansible-core/` — verify check 4 (hvac) and 5 (runner) will fail; check `sudo -u cloudmanager pipx environment` for the actual `PIPX_HOME` and adjust either the script or the user's env.
- Vault token lacks `sudo` on `sys/policies/acl/*` — `configure` fails at `vault policy write`. Use a root token for first install.
- Vault policy already exists with different content — `vault policy write` is upsert, so this is fine, but worth noting in improvise if the prior content was hand-edited.
