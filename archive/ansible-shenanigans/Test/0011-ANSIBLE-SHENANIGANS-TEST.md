# Test: Flip the flag

## Test ID
`0011-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0011-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After the PATCH to `/admin/feature-flags/playbooks`.

## Test Cases

### TC-001: enabled persisted
- **Check**: `psql -c "SELECT enabled FROM membership.feature_flags WHERE key='playbooks'"`
- **Pass**: `t`
- **Fail**: PATCH endpoint returned 200 but DB didn't update — debug

### TC-002: healthy becomes true within probe interval
- **Check**: poll every 30s for up to 6 min: `psql -c "SELECT healthy, last_probed_at FROM membership.feature_flags WHERE key='playbooks'"`
- **Pass**: `healthy=t`, `last_probed_at` within the last 6 min
- **Fail**: probe broken; this means Phase 0006 regressed — Improvise, fix, retry

### TC-003: /playbook returns 200
- **Check**: `curl -sw '%{http_code}' http://localhost:5250/api/v1/playbook -o /dev/null`
- **Pass**: `200`
- **Fail**: filter logic regression

### TC-004: UI shows playbooks
- **Check**: fresh browser load
- **Pass**: `/playbooks` link visible in nav; VmDetailPage shows Playbooks section
- **Fail**: web bundle stale or FeatureGate misreading; hard-reload first

### TC-005: smoke VM still inspectable
- **Check**: `vm_get(smoke-postgres VM id)` via MCP
- **Pass**: still present, still running
- **Fail**: VM was destroyed prematurely → operator can't inspect; this is a process failure not a code one, note in Result

## Scoring
All five.

## On Pass
This is the last task. Check the final box in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`, then write `Results/0001-ANSIBLE-SHENANIGANS-RESULT.md` (workflow-level summary across all 11 phases) and `SelfImprove/0001-ANSIBLE-SHENANIGANS-WISHLIST-001.md` (workflow-level reflection). The workflow is complete.
