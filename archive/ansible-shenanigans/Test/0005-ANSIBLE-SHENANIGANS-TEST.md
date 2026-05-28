# Test: Parameter form web UI

## Test ID
`0005-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0005-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After web deploy. Re-register the postgres playbook first if Phase 0004 cleanup removed it.

## Test Cases

### TC-001: gating off
- **Setup**: flag = (false, false)
- **Check**: load `/`, look at navigation; load `/virtualmachine/{any}`, look at sections
- **Pass**: no Playbooks link in nav, no Playbooks section on VM page
- **Fail**: leak → `<FeatureGate>` missing somewhere

### TC-002: gating on, page renders
- **Setup**: flag = (true, true)
- **Check**: load `/playbooks`
- **Pass**: table with the postgres playbook row
- **Fail**: fetch error → check `/feature-flags` returned the right map at app load

### TC-003: detail page renders schema
- **Check**: click into the playbook
- **Pass**: form fields visible matching the role's variables
- **Fail**: empty form → schema wasn't saved by Phase 0004, or rjsf rejected the schema (open dev console)

### TC-004: attach flow saves vars_override
- **Check**: on a VM, Attach → pick postgres → leave defaults → Submit
- **Pass**: row appears in assignments list; `GET /api/v1/virtualmachine/{vmId}/playbook` returns it with the populated `vars_override`
- **Fail**: 4xx — check request body shape against API expectation

### TC-005: Apply is a stub
- **Check**: click Apply
- **Pass**: toast "Execution lands in Phase 0006"; no `/apply` request in network tab
- **Fail**: backend call made → revert; this is Phase 0006's job

### TC-006: detach removes the row
- **Check**: click Detach → confirm
- **Pass**: row gone; `GET .../playbook` returns empty
- **Fail**: stale slice — check that the slice removes by id on success

## Scoring
All six. Reset flag to (false, false).

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
