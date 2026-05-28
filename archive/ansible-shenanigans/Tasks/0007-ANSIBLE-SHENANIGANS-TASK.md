# Task: Vault scoped tokens + outputs

## Task ID
`0007-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Per-run child tokens. Per-run published secrets surfaced via the run row + UI.

## Context
- The `playbook-run` role accepts `metadata` at mint time (`vm_public_id`, `playbook_public_id`) — they're not used by the broad fallback policy but go in now so a future templated policy can use them without a worker change.
- The child token TTL: 1 hour (per plan), capped at 2h by the role.
- Listing the prefix uses the ORCHESTRATOR token (which has read+list under `cloudmanager/metadata/vm/instances/+/+/*` per the runner policy), not the child token.

## Steps

1. **Worker: mint child token before runner.run**
   ```python
   child = orchestrator.auth.token.create(
       role_name="playbook-run",
       ttl="1h",
       meta={"vm_public_id": vm_pub, "playbook_public_id": pb_pub},
       orphan=True,
   )
   env["VAULT_TOKEN"] = child["auth"]["client_token"]
   accessor = child["auth"]["accessor"]
   ```
   Put the child token in `env/envvars` instead of the orchestrator token.

2. **Worker: revoke child token on completion**
   ```python
   finally:
       orchestrator.auth.token.revoke_accessor(accessor)
   ```
   Inside a `try/finally` around the runner call.

3. **Worker: list prefix and populate `secrets_published`**
   ```python
   listing = orchestrator.secrets.kv.v2.list_secrets(
       mount_point="cloudmanager",
       path=f"vm/instances/{vm_pub}/{pb_pub}",
   )
   # walk recursively, build [{path, output_key, type, description}]
   # cross-reference against playbook.output_schema for labels/types
   ```
   Save to `playbook_runs.secrets_published` jsonb.

4. **API: per-secret read endpoint**
   - `GET /api/v1/run/{runId}/secret?path=<full vault path>` — orchestrator reads, returns `{ value }`. Validates the path is under the run's `vault_secrets_prefix` (no path traversal).

5. **Web: secrets table on run page**
   - Render `secrets_published` as a table: name, type (from output_schema if matched), Reveal button, Copy Path button.
   - Reveal: one call to `GET /run/{runId}/secret?path=...` → display in modal, auto-hide after 30s.
   - Copy Path: clipboard with the full Vault path.

## Acceptance Criteria
- After a postgres playbook run, `playbook_runs.secrets_published` contains entries for at least `admin_password` and `connection_uri` (assuming the role writes them; if it doesn't, write a tiny test playbook that does).
- `vault kv list cloudmanager/vm/instances/{vm_pub}/{pb_pub}/postgres/` shows the same keys.
- During the run, `vault token lookup -accessor <accessor>` shows TTL counting down from ~1h.
- After completion, `vault token lookup -accessor <accessor>` returns 403/404 (token revoked).
- Reveal button on the UI returns the actual password value once, masked thereafter.

## Test
`Test/0007-ANSIBLE-SHENANIGANS-TEST.md`

## On Failure
The token's metadata-at-mint syntax differs between hvac versions. If `meta=` doesn't work try `meta_data=` or pass via raw API.
