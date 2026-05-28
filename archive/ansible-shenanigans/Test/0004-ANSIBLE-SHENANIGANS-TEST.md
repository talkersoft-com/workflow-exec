# Test: Playbook registry API

## Test ID
`0004-ANSIBLE-SHENANIGANS-TEST`

## Task Reference
`Tasks/0004-ANSIBLE-SHENANIGANS-TASK.md`

## When To Run
After API deploy. Flag stays `enabled=false, healthy=false` for all tests *except* the route exercises — for those, flip `enabled=true, healthy=true` temporarily, then reset.

## Test Cases

### TC-001: register postgres role
- **Setup**: flag = (true, true)
- **Check**: `POST /api/v1/playbook` body `{"name":"pg","git_url":"https://github.com/geerlingguy/ansible-role-postgresql","git_ref":"master","playbook_path":"tasks/main.yml"}`
- **Pass**: 201 with `public_id` starting `pb_`, `argument_schema` non-empty, `last_refreshed_at` set
- **Fail**: 500 / empty schema → translator failed, or git clone failed

### TC-002: schema includes expected fields
- **Check**: `GET /api/v1/playbook/{pid}` → `.argument_schema.properties` contains `postgresql_version` (or similar from the role)
- **Pass**: at least 3 of the role's documented variables show up as JSON Schema properties with `type` and `default`
- **Fail**: schema is empty or missing properties → fix translator helper

### TC-003: list returns it
- **Check**: `GET /api/v1/playbook` → array contains the registered row
- **Pass**: present
- **Fail**: list endpoint broken

### TC-004: PATCH triggers refresh
- **Check**: `PATCH /api/v1/playbook/{pid} {git_ref: "master"}` (no-op ref change) → `last_refreshed_at` advances
- **Pass**: timestamp newer than before
- **Fail**: refresh not triggered

### TC-005: DELETE with no assignments succeeds
- **Check**: `DELETE /api/v1/playbook/{pid}`
- **Pass**: 204
- **Fail**: 4xx/5xx → debug

### TC-006: scratch dir cleaned
- **Check**: `ls /var/lib/cloud-manager/playbook-meta/` after the delete
- **Pass**: empty (or no entry for the deleted pid)
- **Fail**: leaked clone → service didn't clean up

### TC-007: flag off ⇒ 404
- **Setup**: flag = (false, false)
- **Check**: `curl -sw '%{http_code}' http://localhost:5250/api/v1/playbook -o /dev/null`
- **Pass**: `404`
- **Fail**: flag bypass — block before advancing

## Scoring
All seven, then reset flag to (false, false).

## On Pass
Check the box for this task in `Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md` and continue to the next unchecked task. Workflow-level Result + Wishlist are written once at the very end, after every box is checked.
