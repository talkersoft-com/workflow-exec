# Task: Branch every repo

## Task ID
`0000-DISKSIZE-DISCO-TASK`

## Parent Orchestration
`Orchestrate/0001-DISKSIZE-DISCO-ORCH.md`

## Objective
Create a `disksize-disco` branch in every repo listed in `branches.md` BEFORE any code work. Run this once at workflow start.

## Steps

1. Open `branches.md`. Read the table of repos.
2. For each repo path in the table, run:
   ```bash
   git -C <repo> fetch origin
   git -C <repo> checkout main && git -C <repo> pull --ff-only
   git -C <repo> checkout -b disksize-disco
   ```
3. Confirm each branch exists locally: `git -C <repo> branch --show-current` returns `disksize-disco`.

## Acceptance Criteria
- Every repo in `branches.md` is on the `disksize-disco` branch locally.
- No code changes yet; the branch matches `main`'s HEAD on each repo.

## Test
`Test/0000-DISKSIZE-DISCO-TEST.md` — one-line bash assertion per repo.

## On Failure
- If `git pull --ff-only` reports "Not possible to fast-forward": main was updated since clone — `git status` to investigate, do NOT force.
- If `checkout -b` reports "branch already exists": you re-ran the task. Confirm the existing branch is at main's HEAD; if so, skip. If diverged, ask the operator.
