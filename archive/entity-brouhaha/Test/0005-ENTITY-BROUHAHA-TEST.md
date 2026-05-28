# Test: EF migration applied to local and remote DB

## Test ID
`0005-ENTITY-BROUHAHA-TEST`

## Task Reference
`Tasks/0005-ENTITY-BROUHAHA-TASK.md`

## Test Cases

### TC-001: migration file exists
```bash
ls ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api/src/Models/CloudManager.Entities/Migrations/ \
  | grep AddPlaybookManagement
```
**Pass**: at least one file matching `AddPlaybookManagement` listed.

### TC-002: local DB — 9 new tables present (port 5433)
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5433 -U postgres -d clouddb -tc "
SELECT count(*) FROM information_schema.tables
WHERE table_schema = 'vm'
  AND table_name IN (
    'playbook_revisions','ansible_roles','role_files','role_file_revisions',
    'playbook_global_role_refs','playbook_roles','playbook_role_files',
    'playbook_role_file_revisions','playbook_run_targets'
  );
"
```
**Pass**: `9`.

### TC-003: local DB — playbooks schema correct
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5433 -U postgres -d clouddb \
  -c "\d vm.playbooks" | grep -E "content|git_url"
```
**Pass**: `content` row present, `git_url` row absent.

### TC-004: local DB — playbook_runs schema correct
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5433 -U postgres -d clouddb \
  -c "\d vm.playbook_runs" | grep -E "playbook_id|assignment_id"
```
**Pass**: `playbook_id` present, `assignment_id` absent.

### TC-005: remote DB — 9 new tables present (port 5432 via tunnel)
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5432 -U cloudmanager -d clouddb -tc "
SELECT count(*) FROM information_schema.tables
WHERE table_schema = 'vm'
  AND table_name IN (
    'playbook_revisions','ansible_roles','role_files','role_file_revisions',
    'playbook_global_role_refs','playbook_roles','playbook_role_files',
    'playbook_role_file_revisions','playbook_run_targets'
  );
"
```
**Pass**: `9`.

### TC-006: remote DB — schema matches local
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5432 -U cloudmanager -d clouddb \
  -c "\d vm.playbooks" | grep -E "content|git_url"
```
**Pass**: `content` present, `git_url` absent.

## Scoring
All 6 TCs must pass before advancing to Task 0006.
If tunnel is not up for TC-005/TC-006, start it first: `ssh -fNL 5432:localhost:5432 todd@ubuntu-server.talkersoft.com`
