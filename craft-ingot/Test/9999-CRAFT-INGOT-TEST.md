# Test: Open PRs and merge

## Test ID
`9999-CRAFT-INGOT-TEST`

## Task Reference
`Tasks/9999-CRAFT-INGOT-TASK.md`

## Test Cases

### TC-001: every PR is MERGED
```bash
for d in <repos from branches.md>; do
  cd $d
  state=$(gh pr view <workflow-name> --json state -q .state 2>/dev/null)
  [ "$state" = "MERGED" ] && echo "OK $d" || echo "FAIL $d (state=$state)"
done
```
**Pass**: every line `OK`.

### TC-002: default branch has the workflow's commits
```bash
for d in <repos from branches.md>; do
  cd $d && git checkout main && git pull --ff-only > /dev/null
  if git log --oneline -10 | grep -q "<workflow-name>"; then
    echo "OK $d"
  else
    echo "FAIL $d — no workflow commits on main"
  fi
done
```

### TC-003: deployed services run the merged code
For each service the workflow touched, verify a new symbol/endpoint is reachable. Example:
```bash
# API: grep deployed config or hit a new endpoint
curl -sw "%{http_code}" http://localhost:5250/api/v1/<new-endpoint> -o /dev/null
# Web: grep the served bundle for a new identifier
grep -o "<NewComponentName>" /opt/cloud-manager-web/assets/index-*.js | head -1
# MCP: list tools and check for the new one
```

### TC-004: Result file lists merge commit SHAs
```bash
grep -E "merge|commit" Results/0001-<workflow-name>-RESULT.md | head
```
**Pass**: at least one short SHA per merged repo.

## Scoring
TC-001 and TC-002 are mandatory. TC-003 should be run per affected service. TC-004 is documentation hygiene — strongly recommended.

## On Pass
This is the last task. Workflow is done.
