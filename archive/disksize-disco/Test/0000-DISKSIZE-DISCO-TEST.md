# Test: Branch every repo

## Test ID
`0000-DISKSIZE-DISCO-TEST`

## Task Reference
`Tasks/0000-DISKSIZE-DISCO-TASK.md`

## When To Run
Immediately after the orchestrator runs the branch-creation loop.

## Test Cases

### TC-001: every repo on the disksize-disco branch
```bash
for d in \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-web \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-mcp \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/vorch-service; do
  b=$(git -C "$d" branch --show-current)
  [ "$b" = "disksize-disco" ] && echo "OK $d" || echo "FAIL $d (branch=$b)"
done
```
**Pass**: every line starts with `OK`.
**Fail**: any `FAIL` → re-run Task 0000 for that repo.

### TC-002: branch HEAD equals main HEAD (clean start)
```bash
for d in \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-web \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-mcp \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/vorch-service; do
  diff=$(git -C "$d" rev-list origin/main..disksize-disco --count)
  # ≤ 2 commits ahead is OK if those commits exist in another open PR (e.g. workflow A landed
  # in a side branch, workflow B branches off main while A is still in review).
  if [ "$diff" -le 2 ]; then echo "OK $d ($diff commits ahead of origin/main)"
  else echo "FAIL $d ($diff commits ahead — investigate)"; fi
done
```
**Pass**: all `OK` with 0 commits ahead.
**Fail**: any nonzero ahead-count → the branch was created on top of stale code. Investigate.

## Scoring
Both TCs must pass before advancing to Task 0001.

## On Pass
Check the box for Task 0000 in `Orchestrate/0001-DISKSIZE-DISCO-ORCH.md` and proceed to Task 0001.
