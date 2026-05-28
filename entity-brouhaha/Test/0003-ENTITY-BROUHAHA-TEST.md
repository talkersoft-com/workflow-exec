# Test: CloudManagerDbContext + public ID prefixes

## Test ID
`0003-ENTITY-BROUHAHA-TEST`

## Task Reference
`Tasks/0003-ENTITY-BROUHAHA-TASK.md`

## Test Cases

### TC-001: DbSets declared for all 9 new entities
```bash
CTX=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.Entities/CloudManagerDbContext.cs
for t in PlaybookRevisions AnsibleRoles RoleFiles RoleFileRevisions \
          PlaybookGlobalRoleRefs PlaybookRoles PlaybookRoleFiles \
          PlaybookRoleFileRevisions PlaybookRunTargets; do
  grep -q "DbSet.*$t\|$t.*DbSet" "$CTX" && echo "OK $t" || echo "FAIL $t missing"
done
```
**Pass**: all 9 lines start with `OK`.

### TC-002: config blocks present for new tables
```bash
CTX=~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.Entities/CloudManagerDbContext.cs
for t in playbook_revisions ansible_roles role_files role_file_revisions \
          playbook_global_role_refs playbook_roles playbook_role_files \
          playbook_role_file_revisions playbook_run_targets; do
  grep -q "\"$t\"" "$CTX" && echo "OK $t" || echo "FAIL $t not configured"
done
```
**Pass**: all 9 lines start with `OK`.

### TC-003: public ID prefixes registered
```bash
grep -r "pbrev\|ansr\|rfrev\|pgr\|plr\|plrf\|plrfr\|prt" \
  ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/ \
  | grep -v ".cs~" | head -20
```
**Pass**: output shows all 9 prefixes referenced in at least one file.

### TC-004: full API build succeeds
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
dotnet build src/CloudManager.API/CloudManager.API.csproj 2>&1 \
  | grep -E "Error\(s\)|Build succ" | tail -3
```
**Pass**: `Build succeeded` and `0 Error(s)`.

## Scoring
All four TCs must pass before advancing to Task 0004.
