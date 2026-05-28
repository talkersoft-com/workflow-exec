# Finish the Ansible run path — Vault scoped tokens, requirement enforcement, multi-controller surface

## What this workflow does
Three features bundled into one PR set: (A) cloud-manager-api mints a short-lived Vault token scoped to a single playbook run, vorch forwards it to ansible-runner as `VAULT_TOKEN`, and the token is revoked on terminal status; (B) the trigger refuses to dispatch a run if its `playbook_collection_requirements` aren't met on the chosen controller, surfaced as a 400 the UI renders inline; (C) the Collections UI honors an explicit controller-host selector backed by `bare_metal.hosts.is_controller`. End state: `pg` against `postgres-test` writes its generated password to `cloudmanager/data/playbook-secrets/<runId>/postgres-test/postgres`, requirement mismatches are caught pre-dispatch, and the Collections page exposes a host dropdown instead of silently picking `hosts[0]`.

## Read before starting
- `deck.md` — repos in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list + /loop directive
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns
- `../../../workflow-plans/cloud-manager/nimble-orangutan/PLAN.md` — source plan with the full design
- `../../../workflow-plans/DATA-MODEL.md` — for the `ansible.*` tables shipped in workflow-plans PR #12

## Constraints
- **camelCase JSON on the wire**, public_ids on the wire, internal UUIDs never leaked.
- **Vault tokens NEVER appear in API logs** — Phase 0002's redaction enricher matches `hvs\.[A-Za-z0-9_\-]+` and replaces with `hvs.<REDACTED>`. Belt-and-suspenders.
- **Vault tokens NEVER appear in run output streams** — vorch must not log the value of `VAULT_TOKEN`, even on env-injection.
- **`PlaybookRun.VaultRunToken` is scrubbed from list endpoints** — only `GetByIdAsync` returns the real value.
- **Revoke on terminal must be idempotent** — the worker may PATCH `status=Succeeded` more than once; the second call is a no-op.
- **No silent fallback to the long-lived cloudmanager token in vorch** — when `vaultRunToken` is empty, env is not set, and vault tasks fail loudly.
- **`playbook_runs` does NOT get a `host_id` column** — multi-controller dispatch is a separate plan.
- **Refuse-only for B** — no auto-install at trigger time.
- **One feature, one PR per affected repo. Five PRs total** (A+B+C land bundled).
- **All hv operations through MCP tools** — never raw git for repos in the deck.
