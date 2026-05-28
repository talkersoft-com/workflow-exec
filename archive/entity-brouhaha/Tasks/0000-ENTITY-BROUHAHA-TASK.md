# Task: Initialize workspace with hive-deck

## Task ID
`0000-ENTITY-BROUHAHA-TASK`

## Parent Orchestration
`Orchestrate/0001-ENTITY-BROUHAHA-ORCH.md`

## Objective
Verify the workspace is clean, then provision all repos and create a feature branch across the
`cloud-manager` deck using hive-deck. Record the auto-generated branch name. Do not write any
code until this task is complete.

## Steps

### 1. Check workspace state
```
hv_status deck: "cloud-manager"
```
All repos must be clean (committed, pushed, no stash, upstream configured) before proceeding.
If any repo is dirty, investigate and resolve before continuing — do NOT proceed with dirty repos.

### 2. Provision and branch
If workspace is clean and repos are on the default branch:
```
hv_init deck: "cloud-manager"
```

If workspace is already provisioned on a prior feature branch (all PRs merged and clean):
```
hv_next deck: "cloud-manager"
```

### 3. Record the branch name
After `hv_init` (or `hv_next`) completes, note the auto-generated branch name from the output.
Record it in `deck.md` under "Generated branch name".

### 4. Confirm with hv_status
```
hv_status deck: "cloud-manager"
```
All repos should now show as clean on the new feature branch (not on `main`).

## Acceptance Criteria
- All repos in the `cloud-manager` deck are on the same auto-generated feature branch
- No repo is on `main`
- Branch name is recorded in `deck.md`
- No code changes yet

## Test
`Test/0000-ENTITY-BROUHAHA-TEST.md`

## On Failure
- `hv_status` shows dirty repos: commit or stash the changes in the affected repo before retrying. If stashing is needed due to a merged PR, use `hv_stash` → `hv_next` → `hv_unstash`.
- `hv_init` refuses because repos are already on a feature branch: check if PRs are merged. If yes, use `hv_next`. If no, ship first with `hv_ship`.
- Network errors cloning a repo: retry `hv_init` — it is idempotent and skips already-cloned repos.
