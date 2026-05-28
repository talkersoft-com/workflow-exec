# Test: New entity classes

## Test ID
`0001-ENTITY-BROUHAHA-TEST`

## Task Reference
`Tasks/0001-ENTITY-BROUHAHA-TASK.md`

## Test Cases

### TC-001: all 9 entity files exist
```bash
BASE=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.Entities/Models
for f in \
  PlaybookRevision.cs \
  AnsibleRole.cs \
  RoleFile.cs \
  RoleFileRevision.cs \
  PlaybookGlobalRoleRef.cs \
  PlaybookRole.cs \
  PlaybookRoleFile.cs \
  PlaybookRoleFileRevision.cs \
  PlaybookRunTarget.cs; do
  [ -f "$BASE/$f" ] && echo "OK $f" || echo "FAIL $f missing"
done
```
**Pass**: all 9 lines start with `OK`.

### TC-002: entities project builds clean
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
dotnet build src/Models/CloudManager.Entities/CloudManager.Entities.csproj 2>&1 \
  | grep -E "Error\(s\)|Build succ" | tail -3
```
**Pass**: `Build succeeded` and `0 Error(s)`.

## Scoring
Both TCs must pass before advancing to Task 0002.
