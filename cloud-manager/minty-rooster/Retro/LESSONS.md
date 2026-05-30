# Retro: cloud-manager/minty-rooster — mcp coverage Tier 1

## FIX files written

None.

## What went smoothly

- The shared API client wrapper (`src/runner.ts` / `src/config.ts`) was already
  correct — camelCase enforcement and 4xx surfacing were both in place from
  prior PRs. Phase 0001 was effectively a confirmation, not a change.
- The three tools are nearly mechanical wrappers over existing endpoints — the
  PATCH/GET shapes matched the API exactly, so no schema translation was needed.
- All three tiers (api, web, mcp) built clean on first attempt.

## What was friction

- The "build but do not reload" constraint for cloud-manager-mcp is subtle and
  must be flagged in RESULT.md, because the running MCP server in the
  operator's Claude Code session is exactly the artifact whose dist we just
  rebuilt. Reloading mid-workflow would invalidate the `hv_ship` tool we still
  need to call. The operator `/mcp` reload step is a hard prerequisite for the
  new tools to actually appear in the next session.

## Non-obvious things for future tier-N MCP workflows

- The shared API client is already hardened — future tiers wrapping more
  endpoints can skip the "wrapper hardening" phase unless they encounter a
  specific new failure mode (e.g. response-streaming, multipart uploads).
- `dist/` is tracked in the cloud-manager-mcp repo; committing built JS is the
  convention. Future workflows should `npm run build` + `git add dist/` as
  part of the implementation phases, not as a separate deploy step.
- The deploy fragment's api/web steps reduce to smoke-test-only when those
  tiers have no code changes. Don't redeploy them — it churns environments
  for no reason.
- `cloud_vm_retry_teardown` and `cloud_run_get_secret` are intentionally
  excluded per the audit-log PLAN (`flaring-balloon`) and Vault-token
  redaction rules respectively. Don't accidentally surface either in a
  future tier without re-reading those constraints.
