# Lessons: cloud-manager/splendid-mermaid

## What went well
- The root cause was already pinned in PLAN.md (resolve-after-create + new-VM's-id substitution), so
  P0/P1 were a precise reorder + signature change, not an investigation. P0 and P1 genuinely
  dovetailed: once the param comes from the operator, resolution no longer needs the new VM's id, so
  moving it before VM creation was clean.
- Resolving **before** the VM insert makes the no-orphan guarantee structural: the 422 path simply
  has no VM to orphan. The defense-in-depth failure-event guard covers only genuinely post-create
  failures, which is the right, narrow scope.

## What would have helped (surprises / friction)
- **The `.cicd/` build + deploy scripts live in `cloud-manager-mcp`, not `cloud-manager-api`.** Both
  task files say `python3 .cicd/build-cloud-manager.py` without a cwd; the scripts resolve siblings
  via `git rev-parse --show-toplevel`, so they must be run from the mcp repo. A one-line note in the
  task files ("run from cloud-manager-mcp") would have saved a wrong-directory miss.
- **The intended E2E source VM `pg-test123` is soft-deleted in the deployed DB**, but `VirtualMachine`
  carries a global soft-delete query filter — so the obvious `GuidByPublicIdAsync` source-VM
  validation rejected a valid source (FIX-001). The lesson generalizes: **a bound secret outlives its
  source VM**, so any lookup of a *source* VM reference for a secret path must `IgnoreQueryFilters()`.
  The already-shipping concrete-path binding (which hardcodes the same soft-deleted VM's path and
  resolved fine) was the tell that live-ness must not be required. Plan note: when a fixture is named
  as a "source", state whether it is expected live or soft-deleted.
- **MCP contract changes need a `/mcp` reload the agent cannot perform**, so the in-session MCP tool
  still advertised the old schema. Driving the param-carrying E2E via `curl` against the deployed
  Instantiate endpoint was the reliable path — worth doing for any workflow that changes an MCP tool
  the same session must then exercise.
- **`db_execute_sql_query` returns only a row count, never data**, and `orchestration_status` is
  stored as the enum *name* (text), not an int — a `= 2` predicate errors. Count-based before/after
  checks (e.g. "zero rows for the 422 attempt name") are the natural fit; use `cloud_vm_list` /
  `cloud_vm_get` for actual status values.

## FIX files
- `FIX-001.md` — soft-deleted source VM rejected by source-VM validation → `IgnoreQueryFilters()`.

## Carry-forward
- A follow-up could let `secretBindingParams` be keyed by per-attachment id (`bsb_…`) for the
  same binding attached twice (PLAN open question) — not needed for this workflow.
- The injection provider seals in-guest (systemd-creds); this workflow verified injection + a
  Completed orchestration. A deeper check could assert the sealed cred is readable inside the guest.
