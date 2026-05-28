# Result: ansible-shenanigans

## Workflow ID
`0001-ANSIBLE-SHENANIGANS`

## Outcome
**Shipped.** All 11 phases complete, flag flipped to `enabled=true, healthy=true`. End-to-end pipeline proven by a real Apply → ansible run on `vm_ZNCMJE89SR` → secrets published to Vault → revealable in UI → child token revoked.

## Phase summary

| Phase | Title | Result | Improvises |
|---|---|---|---|
| 0001 | Verify install + Vault config | ✅ | 3 (perms, token, sudo wrapper) |
| 0002 | Feature flag scaffolding | ✅ | 1 (route + PATCH naming) |
| 0003 | Playbook data model | ✅ | 1 (TC-007 EF-only vs trigger expectation) |
| 0004 | Playbook registry API | ✅ | 1 (test repo + outputs JSON encoding) |
| 0005 | Parameter form web UI | ✅ | 1 (Toast/EmptyState prop names + visual-test gap) |
| 0006 | Python worker + first end-to-end run | ✅ | 1 (PATH/HOME/ProtectHome + SQL ip lookup + playbook file shape) |
| 0007 | Vault scoped tokens + outputs | ✅ | 1 (two-token split + collections path) |
| 0008 | SignalR run log streaming | ✅ | 1 (browser-test gap; Node SignalR client used as proxy) |
| 0009 | Schema diffing | ✅ | 1 (placement change vs plan + minimal UI) |
| 0010 | End-to-end smoke | ✅ (with workload swap) | 1 (postgres → nginx due to VM disk; 4 VM-side issues) |
| 0011 | Flip the flag + workflow outputs | ✅ | 0 |

## Smoke VM (left up for inspection per instructions)
- `vm_ZNCMJE89SR` at `10.0.150.51` — Ubuntu 6.8.0-100-generic
- nginx-light installed, vhost `smoke-site` on `:8080`
- Vault: `cloudmanager/vm/instances/vm_ZNCMJE89SR/pb_EMEJCH08GG/nginx/{admin_password, connection_uri, port}` + 1 leftover postgres entry
- SSH key in Vault at `cloudmanager/vm/instances/vm_ZNCMJE89SR/ssh_key`
- Test playbook repo at `/var/lib/cloud-manager/test-playbook` with branches: `master` (current nginx smoke), `drift-test-broken` (Phase 0009 fixture), `drift-test-additive` (Phase 0009 fixture)

## Currently running
- `cloud-manager-api` (systemd) — port 5250, env file at `/etc/cloud-manager/api.env` (operational vault token)
- `cloud-manager-web` (systemd) — port 3000, serves `/opt/cloud-manager-web`
- `cloud-manager-worker` (systemd) — env file at `/etc/cloud-manager/worker.env` (operational + orchestrator vault tokens, ansible env overrides)
- Vault, RabbitMQ, Postgres — pre-existing infrastructure

## Final flag state
```
key=playbooks  enabled=true  healthy=true  last_probed_at=2026-05-24T18:01:01Z
```
Worker health probe runs every 5 min; flag stays true as long as the no-op probe playbook succeeds against localhost.

## What worked the first time
- The skill-style scaffolding of Phase 0001 (vault-cli pattern reuse for ansible-cli) — install/configure/verify/uninstall + bash wrapper
- Refactoring all MCP UUID schemas to also accept Stripe-style public_ids
- EF Core migration shape after FeatureFlag entity + EntityPrefixRegistry add
- Backend feature flag state machine (404 missing/disabled, 503 unhealthy, 200 active)
- AutoMapper PublicId ↔ Id swap pattern (cribbed from existing profiles)
- SignalR group-scoped run logs (one global hub, per-run group)
- Drift classifier as a pure function (testable, dedups correctly)

## What needed improvisation
- Systemd sandbox vs ansible's HOME/temp expectations (Phase 0006)
- Two-token vault split because no single existing policy granted both mint and read (Phase 0007)
- `become: yes` on play-level propagating to `delegate_to: localhost` and conflicting with `NoNewPrivileges` (Phase 0010)
- Test VM disk size too small for postgres (Phase 0010 workload swap)

## What the user should know
- The auth on the nginx smoke vhost returns 200 without challenging — investigate (likely `return 200` interaction with `auth_basic` order). The cloud-manager pipeline is correct; this is playbook content.
- The cloudmanager API VM column `machine_status` shows `1=Off` for `vm_ZNCMJE89SR` even though it's running — known issue from before this workflow.
- Vault tokens currently stored plaintext in env files (`~/cloudmanager_vault_token.txt`, `~/ansible_token.txt`, `/etc/cloud-manager/*.env`). User explicitly deferred token storage security — circle back when ready.

## References
- Workflow plan: `../../planning/PLAYBOOK-INTEGRATION-PLAN.md`
- Per-phase Improvises: `../Improvise/000[1-9,10]-*.md`
- Wishlist: `../SelfImprove/0001-ANSIBLE-SHENANIGANS-WISHLIST-001.md`
