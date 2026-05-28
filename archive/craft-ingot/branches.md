# Branches: craft-ingot

This file declares every repo this workflow will modify. **The orchestrator MUST create a feature branch in every repo listed here before starting Task 0001.** Doing this up front prevents the "we did 11 phases on main" cleanup that bit a prior workflow.

## Convention

- One row per repo
- The **Branch name = this workflow's folder name** (e.g. `ansible-shenanigans`, `craft-ingot`). Same name across every repo.
- Branch from the repo's main/default branch (use `main` unless your repo uses something else)
- "Purpose" is one short phrase — why this repo is in the list

## Repos for this workflow

| Repo path (relative to user's workspace) | Branch | Purpose |
|---|---|---|
| _example/example-api_ | craft-ingot | Where the backend changes land |
| _example/example-web_ | craft-ingot | Where the frontend changes land |

## How to set up (run this from the agent before Task 0001)

For each row:

```bash
git -C <repo-path> fetch origin
git -C <repo-path> checkout main && git -C <repo-path> pull --ff-only
git -C <repo-path> checkout -b <branch-name>
```

Skipping this step is a process failure, not a code one — and it gets caught at "did we break off a branch?" review time when it's already painful to clean up. Don't skip it.

## Optional: protected paths

If a repo has paths that should NOT be modified by this workflow even though the repo is in scope, list them here. Future automation may use this as a guardrail.

(none)

## Notes

- If a repo doesn't end up needing changes after all, leave the branch dormant or delete it before opening a PR
- If a NEW repo gets pulled in mid-workflow, add it here AND create the branch before touching it
- This file is the source of truth for "what was supposed to be touched" — keep it accurate
- The **last task of every workflow** is `Tasks/9999-...` — opens PRs against every repo here and merges them. See `Tasks/9999-CRAFT-INGOT-TASK.md`.
