# Task: hv_ship + verify DB deployed on remote

## Task ID
`0006-ENTITY-BROUHAHA-TASK`

## Objective
Commit all changes, push, open and merge PRs via hive-deck, then verify the remote database
reflects the deployed schema. This is the completion gate for the workflow.

## Step 1 — Final build check before shipping
```bash
cd ~/workspace/cloud-manager/vm-infra/cloud-manager/cloud-manager-api
dotnet build src/CloudManager.API/CloudManager.API.csproj 2>&1 | grep -E "Error\(s\)|Build succ" | tail -5
```
**Must pass** before calling `hv_ship`. Do not ship a broken build.

## Step 2 — Ship
```
hv_ship
  deck: "cloud-manager"
  message: "feat: playbook management data model — EF entities, migration, remove git-backed code"
  title: "entity-brouhaha: playbook management data model"
  body: "Implements the playbook management schema from DATA-MODEL.md. Adds 9 new EF entities, updates Playbook and PlaybookRun, removes git-backed service code, and applies the AddPlaybookManagement migration."
```

`hv_ship` will:
- Commit any uncommitted changes in `cloud-manager-api`
- Push to the remote feature branch
- Open a PR
- Auto-merge (if `auto_merge: true` in config)
- Skip repos with no changes

Record the PR URL and merge commit SHA in the Result file.

## Step 3 — Verify remote DB schema

After the PR merges and CI (if any) completes, verify the remote DB has the correct schema:

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
**Pass**: 9 rows returned.

```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5432 -U cloudmanager -d clouddb \
  -c "\d vm.playbooks" | grep -E "content|git_url"
```
**Pass**: `content` present, `git_url` absent.

```bash
PGPASSWORD='P@ssw0rd!' psql -h localhost -p 5432 -U cloudmanager -d clouddb \
  -c "\d vm.playbook_runs" | grep -E "playbook_id|assignment_id"
```
**Pass**: `playbook_id` present, `assignment_id` absent.

## Step 4 — Record branch name + PR in deck.md
Update `deck.md`:
- Confirm the generated branch name is recorded
- Add the PR URL and merge commit SHA

## Acceptance Criteria
- `hv_ship` completes without error
- PR opened (and merged if auto_merge is on) for `cloud-manager-api`
- Remote DB has all 9 new tables
- `vm.playbooks` has `content`, no git columns
- `vm.playbook_runs` has `playbook_id`, no `assignment_id`

## Test
`Test/0006-ENTITY-BROUHAHA-TEST.md`

## On Failure
- `hv_ship` refuses because a repo is on the default branch: run `hv_status` to diagnose — the workspace should be on the feature branch. If not, investigate before retrying.
- PR auto-merge blocked by a required review: note the PR URL in the Result and inform the operator. The DB migration was already applied in Phase 5.
- Remote DB verification fails after merge: the migration was applied in Phase 5 before shipping — the remote DB should already have the schema. If not, re-run `deploy-database.py` targeting the remote.

## When complete
Write:
- `Results/0001-ENTITY-BROUHAHA-RESULT.md`
- `SelfImprove/0001-ENTITY-BROUHAHA-WISHLIST-001.md`
