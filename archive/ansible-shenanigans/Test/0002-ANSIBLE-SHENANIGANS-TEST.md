# Test: Feature flag scaffolding

## Test ID
`0002-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0002-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After the API + web deploy completes.

## Test Cases

### TC-001: `/feature-flags` returns the map
- **Check**: `curl -s http://localhost:5250/api/v1/feature-flags | jq '.playbooks'`
- **Pass**: returns `{ "enabled": false, "healthy": false, "last_probed_at": null, ... }`
- **Fail**: 404 → controller not registered; missing key → seed didn't run

### TC-002: disabled flag returns 404
- **Setup**: ensure `playbooks.enabled=false`
- **Check**: `curl -sw '%{http_code}' http://localhost:5250/api/v1/playbook -o /dev/null`
- **Pass**: `404`
- **Fail**: 200/503 → filter not applied or wrong precedence

### TC-003: enabled-but-unhealthy returns 503
- **Setup**: `PATCH /admin/feature-flags/playbooks {enabled: true}`; healthy stays false
- **Check**: `curl -sw '%{http_code}' http://localhost:5250/api/v1/playbook -o /dev/null`
- **Pass**: `503`
- **Fail**: 200 → filter only checks `enabled`, missing the healthy branch

### TC-004: enabled+healthy returns 200
- **Setup**: update DB row `set healthy = true where key = 'playbooks'`
- **Check**: `curl -sw '%{http_code}' http://localhost:5250/api/v1/playbook -o /dev/null`
- **Pass**: `200`
- **Fail**: filter still blocking → recheck precedence

### TC-005: web UI gates correctly
- **Setup**: leave `enabled=false`
- **Check**: load the web app; inspect Redux DevTools or page source for a "Playbooks" section
- **Pass**: section is not in the DOM
- **Fail**: section visible → `FeatureGate` not wired

### TC-006: reset to off
- **Check**: DB row `playbooks` has `enabled=false, healthy=false` at the end of the test
- **Pass**: confirmed
- **Fail**: tests left state dirty → reset manually before advancing

## Scoring
All six. Reset to off (TC-006) is mandatory.

## On Failure
Improvise, retry, re-test.

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
