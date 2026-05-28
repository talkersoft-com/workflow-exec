# Test: Service/DTO cleanup + clean build

## Test ID
`0004-ENTITY-BROUHAHA-TEST`

## Task Reference
`Tasks/0004-ENTITY-BROUHAHA-TASK.md`

## Test Cases

### TC-001: DTO Playbook has Content, no git fields
```bash
BASE=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.DTO/Models/Playbook.cs
grep -q "Content"        "$BASE" && echo "OK Content present"          || echo "FAIL Content missing"
grep -q "GitUrl"         "$BASE" && echo "FAIL GitUrl still present"   || echo "OK GitUrl removed"
grep -q "GitRef"         "$BASE" && echo "FAIL GitRef still present"   || echo "OK GitRef removed"
grep -q "PlaybookPath"   "$BASE" && echo "FAIL PlaybookPath present"   || echo "OK PlaybookPath removed"
grep -q "LastRefreshedAt" "$BASE" && echo "FAIL LastRefreshedAt present" || echo "OK LastRefreshedAt removed"
```
**Pass**: all 5 lines start with `OK`.

### TC-002: DTO PlaybookRun has PlaybookId, no AssignmentId
```bash
BASE=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.DTO/Models/PlaybookRun.cs
grep -q "PlaybookId"   "$BASE" && echo "OK PlaybookId present"       || echo "FAIL PlaybookId missing"
grep -q "AssignmentId" "$BASE" && echo "FAIL AssignmentId present"   || echo "OK AssignmentId removed"
```
**Pass**: both lines start with `OK`.

### TC-003: IPlaybookService no longer declares git-era methods
```bash
BASE=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Services/CloudManager.Data.Services/Interface/IPlaybookService.cs
grep -q "RefreshAsync"              "$BASE" && echo "FAIL RefreshAsync still declared"             || echo "OK RefreshAsync removed"
grep -q "DetectDriftForAssignment"  "$BASE" && echo "FAIL DetectDrift still declared"             || echo "OK DetectDrift removed"
```
**Pass**: both lines start with `OK`.

### TC-004: No references to removed fields across entire src tree
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
grep -r "AssignmentId\|GitUrl\|GitRef\|PlaybookPath\|LastRefreshedAt" src/ \
  --include="*.cs" \
  --exclude-dir=Migrations \
  -l
```
**Pass**: no output (no files reference these names outside migrations).
**Fail**: listed files still have references — remove them and rerun.

### TC-005: Full API build is clean
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
dotnet build src/CloudManager.API/CloudManager.API.csproj 2>&1 | grep -E "Error\(s\)|Build succ" | tail -5
```
**Pass**: `Build succeeded` with `0 Error(s)`.

## Scoring
All 5 TCs must pass before advancing to Task 0005.
