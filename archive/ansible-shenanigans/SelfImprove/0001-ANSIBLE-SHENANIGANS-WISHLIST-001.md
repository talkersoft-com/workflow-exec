# Wishlist: ansible-shenanigans (workflow-level reflection)

## Scope
Cross-phase reflection on what would have cut total wall-clock the most if it had existed before phase 0001. Per-phase improvises are in `../Improvise/`. This document is for the operator's tooling-investment decisions, not for re-litigating decisions.

## Top three (most-leverage)

### 1. **Vault MCP** for read/list/policy ops
I burned ~30 minutes on Phase 0001 + 0007 alone wrestling with vault policy capabilities (which token grants what; can this token mint? can it list?). Every check meant: ssh, source env, export VAULT_*, run vault command, parse output. A small MCP with `vault_token_capabilities(token, path, op)`, `vault_kv_list(mount, path)`, `vault_token_lookup(token | accessor)` would have made Phase 0007's two-token decision visible in seconds. Also valuable: `vault_policy_show(name)` so I could read the `cloudmanager` policy without needing sudo-on-acl.

### 2. **VM SSH helper MCP** (`vm_ssh(vm_public_id, command)`)
Phase 0010 needed me to repeatedly SSH into the test VM to: free disk, kill stuck apt, check service state, fix package state. Each iteration was: extract key from Vault, write tempfile, chmod, ssh, rm tempfile. A `vm_ssh` MCP method that resolves the key from Vault by `vm_public_id` would have collapsed each of those iterations to a single tool call. Bonus: a `vm_exec_root` that pre-handles sudo.

### 3. **Operational checklist for the playbook flow**
A "what good looks like" doc covering:
- Ansible service unit constraints: `NoNewPrivileges`, `ProtectHome`, `ProtectSystem` and what they break (especially `~/.ansible/tmp` and HOME)
- `become: no` is required on `delegate_to: localhost` tasks
- collections must live somewhere stable (`/opt/.../collections`) and `ANSIBLE_COLLECTIONS_PATH` set
- `vars_override` round-trip through jsonb deserializes to dict on Python side

I learned each of these the hard way. Phase 0006 + 0010 ate 60+ minutes on systemd-sandbox interactions that a one-pager would have prevented.

## Other things that compounded

### Test fixture VM size
2.4 GB root disk on the test VM is the wrong size. Phase 0010 had to switch from postgres to nginx-light because postgres install couldn't complete. Real ops work needs ≥10 GB.

### Hermetic test playbook lives outside git
The hermetic test playbook at `/var/lib/cloud-manager/test-playbook` is not in version control with the cloud-manager-api repo. It was created ad-hoc in Phase 0004. Should either be checked in under `cloud-manager-api/test-fixtures/playbooks/` or moved to a small dedicated `cloud-manager-test-playbooks` repo. Two reasons: re-doing this workflow tomorrow requires recreating the repo; and the test playbook's evolution (postgres → nginx) is invisible from git history.

### Permission-locked `dist/` directory
Every `npm run build` failed until I `sudo rm -rf dist` — the web install script runs as root and the resulting tree is root-owned. Either chown dist back to the dev user at install end, OR change the deploy to build under a temp dir and copy. Minor friction, but it bit me 3+ times.

### `add-migration.py` wraps `dotnet ef` without surfacing warnings
Phase 0003 had EF print "PlaybookRunStatus property 'Status' on entity type 'PlaybookRun' is configured with a database-generated default, but has no configured sentinel value" — this was the source of the integer-vs-text enum mapping. The wrapper script muted it as INFO output. Surface EF warnings as warnings.

### Cloud Manager MCP coverage
The MCP did not have: `playbook_list`, `playbook_register`, `assignment_list`, `run_get`, `run_apply`. The phases I built would benefit from MCP coverage in Phase 0010+: instead of curl-ing the API repeatedly to poll runs, an MCP method `wait_for_run(runId, timeout=600)` would replace the polling loops I wrote 5 times.

## What I would do differently
- Set `ANSIBLE_LOCAL_TEMP` + `HOME` env vars in the worker install script template, not as a Phase-0006 fix-up
- Build a "two-token" vault decision into the install script from day 1, not bolt it on in Phase 0007
- Make the test playbook a git submodule of cloud-manager-api so changes are auditable
- Add `ansible_become: no` at the play default level and lift to `yes` only on tasks that touch the target — would avoid the delegate_to localhost pitfall
- Pre-flight disk + apt state on any target VM before kicking off Apply

## Things that were just fine (don't change)
- The Stripe-style public_id pattern + AutoMapper Id↔PublicId swap
- The feature-flag state machine (404/503/200) is exactly right
- The /loop master ORCH with single-source-of-truth task checkboxes — much better than 11 separate ORCH files
- SignalR group-scoping by run_id (one hub, many groups) cleanly avoids per-run hub plumbing
- Hermetic test repo (once we accept the location wart) — beats relying on github.com being reachable
