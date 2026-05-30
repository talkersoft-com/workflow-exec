# Lessons: cloud-manager/magical-canary

## Carry-forward (READ THIS FIRST)
- **Operator must `/mcp` reload to expose `cloud_playbook_run_list`.** The deploy script intentionally does not restart MCP — only the operator can. Until that reload happens, the new MCP tool is built into `dist/` but not yet available to Claude Code.
- `playbookId` and `status` filters are wired in the API + MCP layer but the web UI only exposes `vmId` for v1. Future phases can surface them in the same URL/Redux pattern (`useSearchParams` → Redux mirror → `api.runs.list({...})`).

## What would have helped
1. The task body for Phase 5 included a psql snippet to pick "the top VM" by playbook-run count, but it did NOT filter `deleted_at IS NULL`. The API's `VirtualMachines` DbSet has a global query filter on `DeletedAt`, so the top-by-count VM (`vm_GY68JPPKDN`) is soft-deleted and correctly returned `total=0` — looked like a bug at first glance. Future plans that hand-roll psql snippets for cross-checks against the API should include the same soft-delete filter the API applies, or call it out explicitly so the verifier doesn't double-take.
2. The deploy script's built-in smoke test against `/api/v1/Host` 404s because `HostController` only exposes `/api/v1/Host/list`. This is pre-existing and unrelated to the magical-canary work, but it produces a scary red line in the deploy output during Phase 2. Worth flagging in a future fix: either point the smoke test at `/api/v1/Host/list` or pick a different always-on liveness endpoint.
3. The `FIX-001` stub-style breadcrumbs left in `AnsibleHistoryPage` from prior workflows were genuinely useful — they made Phase 4 a "fill in this seam" exercise rather than an archaeology dig. Future plans should keep that habit.

## Fix files written
- none

## Notes for the next workflow
- All 15 repos started clean on `magical-canary` except `planning/workflow-exec`, whose only dirty path was the expected `?? cloud-manager/magical-canary/` scaffold. Phase 0 verification of that exception is a good gate to keep.
- Phase 2 + Phase 5 both deployed (api, then web). Phase 6 only needed `--target mcp` because the others were already live. If a future workflow can lean on `--target all` at the very end instead of incremental deploys, it would centralize the deploy log in one place.
- VaultRunToken scrubbing was verified end-to-end in TC-003 (`tokensLeaked: 0` across all 50 items). Keep this check in any future endpoint that surfaces PlaybookRun records.
