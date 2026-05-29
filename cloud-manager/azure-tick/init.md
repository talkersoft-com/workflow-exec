# porch follow-ups — execution workflow

## What this workflow does
Closes every gap captured in jazzy-herring/Retro/LESSONS.md. Headline outcome: the `pg` playbook actually installs PostgreSQL on `postgres-test` and writes the generated password to `cloudmanager/playbook-secrets/<runId>/postgres-test/postgres` in Vault. Along the way: porch redaction extends to subprocess streams, ansible-runner is system-installed so porch's `ProtectHome` returns to `yes`, materialize emits under `project/` so porch drops its `--project-dir` workaround, and 7 stale Queued runs get retired.

## Read before starting
- `deck.md` — repos in scope + hv MCP calls
- `Orchestrate/ORCH.md` — task list + `/loop` directive
- `../../../workflow-plans/cloud-manager/nutty-cobbler/PLAN.md` — source plan (authoritative)
- `../../cloud-manager/jazzy-herring/Retro/FIX-002.md` — full diagnosis of the inventory gap
- `../../cloud-manager/jazzy-herring/Retro/LESSONS.md` — the five follow-up tags
- `../../../workflow-plans/cloud-manager/charmed-panda/PLAN.md` — the carve-out this builds on
- `../../../workflow-plans/cloud-manager/nimble-orangutan/PLAN.md` — per-run scoped Vault token model

## Constraints
- camelCase JSON on the wire; public_ids only; internal UUIDs never leaked.
- Vault tokens NEVER in logs (existing porch redactor stays; Gap 3 extends it to subprocess streams).
- Vault tokens NEVER in run output streams.
- `ip_address` column on `vm.virtual_machines` is nullable so existing rows aren't broken.
- One feature, one PR per affected repo. Expected: `cloud-manager-api`, `vorch-service`, `workflow-exec`.
- After ship: `ProtectHome=yes` on porch.service; no pipx-symlink chain; no `--project-dir` flag in porch's exec.
- No web UI changes — `ipAddress` is additive on the wire.
