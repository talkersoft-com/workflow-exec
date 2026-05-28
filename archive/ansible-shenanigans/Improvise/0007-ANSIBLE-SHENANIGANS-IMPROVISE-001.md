# Improvise: Phase 0007 token split + collections path

## Phase
`0007-ANSIBLE-SHENANIGANS`

## Issues encountered

### 1. cloudmanager operational token can't mint via playbook-run role
The cloudmanager policy has `read/list` on `cloudmanager/*` but not `create` on `auth/token/create/playbook-run`. The infra-toolkit policy has the inverse. **Solution: worker now holds two tokens.**
- `VAULT_TOKEN` (cloudmanager operational) — SSH key reads, listing/reading published secrets after the run
- `VAULT_ORCHESTRATOR_TOKEN` (infra-toolkit) — minting + revoking child tokens via playbook-run role

This is a temporary shape — the cleaner long-term answer is a dedicated `cloudmanager-orchestrator` policy attached to the cloudmanager token granting both. Logged for the workflow Wishlist.

### 2. community.general lookup plugin not found
The test playbook uses `community.general.random_string` for password generation, but the worker's ansible couldn't find it. Cause: `HOME=/tmp/.cloudmanager-home` env override (Phase 0006 workaround) made ansible look for collections under that new HOME, not `/home/cloudmanager/.ansible`. **Solution:** install collections to `/opt/cloud-manager-worker/collections` (cloudmanager-owned, stable path) and set `ANSIBLE_COLLECTIONS_PATH=/opt/cloud-manager-worker/collections` in worker env.

### 3. API needs its own VAULT_TOKEN for /run/{id}/secret
The Reveal endpoint reads from Vault. Added `/etc/cloud-manager/api.env` (operational token) and wired `EnvironmentFile=-/etc/cloud-manager/api.env` into the install script.

## Verification (all 8 TCs)
- TC-001 ✅ mint response shows `metadata.{vm,playbook}_public_id` populated
- TC-002 ✅ token_policies = `["cloudmanager-playbook-write", "default"]` (no orchestrator policy leakage)
- TC-003 ✅ lease_duration = 3600 (1h)
- TC-004 ✅ `vault kv list cloudmanager/vm/instances/vm_ZNCMJE89SR/pb_EMEJCH08GG` → `postgres`
- TC-005 ✅ post-run accessor lookup returns 403/permission denied (vault doesn't differentiate "revoked" from "never existed" for non-existent accessors)
- TC-006 ✅ `secrets_published = [{path, output_key=postgres, type=secret}]`
- TC-007 ✅ Backend: `/api/v1/run/{id}/secret?path=...` returns `{admin_password, connection_uri}`. UI: `RunDetailPage` with Reveal button + 30s auto-hide modal built and deployed (user must visually confirm)
- TC-008 ✅ Path traversal: `?path=cloudmanager/data/vm/instances/OTHER/OTHER/secret` → HTTP 403 "Path outside run prefix"

## Real run for verification
`run_NHBAK5GQ2Z` → Succeeded → published `postgres` secret with `admin_password=i5TDOfonnRAimECes7O1Sidc` and a postgres connection URI. Child token accessor `pSZ18sMYjwZFbXwptvkT9pPy` revoked post-run.

## Followup / Wishlist
- The two-token shape works but is a wart. Should be one token w/ combined policies, OR a hidden init step that mints the orchestrator child at API/worker startup and rotates.
- Path traversal validation is string-prefix-based. If we add KV v1 mounts later, validate against the actual prefix shape.
- Reveal modal auto-hide is 30s — picked from the test doc. Consider letting users dismiss earlier (already supported via Close button).
