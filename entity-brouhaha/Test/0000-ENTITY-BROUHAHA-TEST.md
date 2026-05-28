# Test: Workspace initialized with hive-deck

## Test ID
`0000-ENTITY-BROUHAHA-TEST`

## Task Reference
`Tasks/0000-ENTITY-BROUHAHA-TASK.md`

## Test Cases

### TC-001: hv_status shows all repos clean on a feature branch
```
hv_status deck: "cloud-manager"
```
**Pass**: every repo reports clean status AND is on a feature branch (not `main` / default branch).
**Fail**: any repo is dirty, on `main`, or has an upstream not configured → re-run Task 0000.

### TC-002: branch name recorded in deck.md
Open `deck.md` and confirm the "Generated branch name" field is filled in with the branch name
returned by `hv_init` or `hv_next`.

**Pass**: field is not empty and matches what `hv_status` reports.
**Fail**: field is blank → fill it in before advancing.

## Scoring
Both TCs must pass before advancing to Task 0001.

## On Pass
Check the box for Task 0000 in `Orchestrate/0001-ENTITY-BROUHAHA-ORCH.md` and proceed to Task 0001.
