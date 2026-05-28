# Test: Updated Playbook and PlaybookRun entities

## Test ID
`0002-ENTITY-BROUHAHA-TEST`

## Task Reference
`Tasks/0002-ENTITY-BROUHAHA-TASK.md`

## Test Cases

### TC-001: Playbook.cs has Content, does NOT have git fields
```bash
BASE=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.Entities/Models/Playbook.cs
grep -q "Content" "$BASE" && echo "OK Content present" || echo "FAIL Content missing"
grep -q "GitUrl"  "$BASE" && echo "FAIL GitUrl still present" || echo "OK GitUrl removed"
grep -q "GitRef"  "$BASE" && echo "FAIL GitRef still present" || echo "OK GitRef removed"
grep -q "PlaybookPath"      "$BASE" && echo "FAIL PlaybookPath still present" || echo "OK PlaybookPath removed"
grep -q "LastRefreshedAt"   "$BASE" && echo "FAIL LastRefreshedAt still present" || echo "OK LastRefreshedAt removed"
```
**Pass**: all 5 lines start with `OK`.

### TC-002: PlaybookRun.cs has new fields, does NOT have AssignmentId
```bash
BASE=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.Entities/Models/PlaybookRun.cs
grep -q "PlaybookId"         "$BASE" && echo "OK PlaybookId present"         || echo "FAIL PlaybookId missing"
grep -q "PlaybookRevisionId" "$BASE" && echo "OK PlaybookRevisionId present"  || echo "FAIL PlaybookRevisionId missing"
grep -q "VarsSnapshot"       "$BASE" && echo "OK VarsSnapshot present"        || echo "FAIL VarsSnapshot missing"
grep -q "Output"             "$BASE" && echo "OK Output present"              || echo "FAIL Output missing"
grep -q "AssignmentId"       "$BASE" && echo "FAIL AssignmentId still present" || echo "OK AssignmentId removed"
```
**Pass**: all 5 lines start with `OK`.

### TC-003: entities project still builds clean
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
dotnet build src/Models/CloudManager.Entities/CloudManager.Entities.csproj 2>&1 \
  | grep -E "Error\(s\)|Build succ" | tail -3
```
**Pass**: `Build succeeded` and `0 Error(s)`.

## Scoring
All three TCs must pass before advancing to Task 0003.
