# Test: hv_ship + remote DB verification

## Test ID
`0006-ENTITY-BROUHAHA-TEST`

## Task Reference
`Tasks/0006-ENTITY-BROUHAHA-TASK.md`

## Test Cases

### TC-001: hv_ship completed without error
```
hv_status deck: "cloud-manager"
```
**Pass**: `cloud-manager-api` shows as clean with all commits pushed (nothing ahead of upstream unpushed). All other repos also clean.

### TC-002: PR opened (and merged if auto_merge is on)
```
hv_list_pulls deck: "cloud-manager"
```
**Pass (auto_merge on)**: no open PRs — all merged.
**Pass (auto_merge off)**: PR URL is visible and recorded in `deck.md` and the Result file.

### TC-003: remote DB — final schema verification
```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5432 -U cloudmanager -d clouddb -tc "
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'vm'
  AND table_name IN (
    'playbook_revisions','ansible_roles','role_files','role_file_revisions',
    'playbook_global_role_refs','playbook_roles','playbook_role_files',
    'playbook_role_file_revisions','playbook_run_targets'
  )
ORDER BY table_name;
"
```
**Pass**: 9 rows.

### TC-004: deck.md has branch name and PR reference recorded
Open `deck.md` and confirm:
- "Generated branch name" is filled in
- PR URL (or merge commit SHA) is noted

**Pass**: both fields present.

## Scoring
All 4 TCs must pass. On pass, write Results and Wishlist — workflow complete.
