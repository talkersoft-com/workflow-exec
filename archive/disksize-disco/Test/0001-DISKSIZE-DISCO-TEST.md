# Test: API entity + migration

## Test ID
`0001-DISKSIZE-DISCO-TEST`

## Task Reference
`Tasks/0001-DISKSIZE-DISCO-TASK.md`

## Test Cases

### TC-001: column exists with expected type + default
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -U cloudmanager -d clouddb \
  -c "\d vm.virtual_machines" | grep disk_size_gb
```
**Pass**: shows `disk_size_gb | integer | | not null | 10`.

### TC-002: no NULL or zero rows
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -U cloudmanager -d clouddb \
  -tc "SELECT count(*) FROM vm.virtual_machines WHERE disk_size_gb IS NULL OR disk_size_gb = 0;"
```
**Pass**: `0`.

### TC-003: API still builds
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
dotnet build src/CloudManager.API/CloudManager.API.csproj 2>&1 | grep -E "Error|Build succ" | tail
```
**Pass**: `Build succeeded` and `0 Error(s)`.

## Scoring
All three. On pass, check the box for Task 0001 and advance to Task 0002.
