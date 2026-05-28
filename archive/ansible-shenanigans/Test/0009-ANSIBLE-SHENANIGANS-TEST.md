# Test: Schema diffing

## Test ID
`0009-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0009-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
Set up a deliberate drift test repo first — fork a tiny playbook and create two branches:
- `drift-test-stable` — current schema
- `drift-test-broken` — adds a required field with no default

## Test Cases

### TC-001: no drift, runs as before
- **Setup**: assignment on `drift-test-stable`
- **Check**: Apply
- **Pass**: run Succeeded; no drift toast
- **Fail**: false positive — classifier flagged identity as drift

### TC-002: breaking drift returns 422
- **Setup**: change assignment's playbook `git_ref` to `drift-test-broken`; Apply
- **Check**: API response
- **Pass**: 422 with `breaking` list containing the new required field
- **Fail**: 5xx, or 202 (drift not detected) → fix classifier

### TC-003: 422 modal renders
- **Check**: UI when clicking Apply against the broken ref
- **Pass**: modal with field-level diff; both buttons present
- **Fail**: silent failure → handle 422 in apply flow

### TC-004: adopt new schema flow
- **Check**: click "Adopt new schema", fill in the new required field, save assignment, click Apply
- **Pass**: run Succeeded
- **Fail**: PATCH didn't save the new vars_override → debug

### TC-005: non-breaking drift proceeds
- **Setup**: create `drift-test-additive` branch that adds an optional field with default; switch assignment to it; Apply
- **Check**: run + worker log
- **Pass**: run Succeeded; worker log has "non-breaking drift" warning
- **Fail**: 422 → over-eager classifier; or no warning → log path broken

### TC-006: schema cache freshens
- **Check**: after each Apply, `playbooks.argument_schema` and `last_refreshed_at` reflect the latest fetched schema
- **Pass**: both updated
- **Fail**: stale cache means future form rendering uses outdated fields

## Scoring
All six. Clean up drift-test branches when done if they're in a real repo.

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
