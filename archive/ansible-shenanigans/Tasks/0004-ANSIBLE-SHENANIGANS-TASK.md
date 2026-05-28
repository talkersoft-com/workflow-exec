# Task: Playbook registry API

## Task ID
`0004-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Five endpoints, all behind the playbooks flag, all reading public_ids and storing UUIDs:
- `GET /api/v1/playbook` — list (drop the temp 0002 no-op route; this replaces it)
- `POST /api/v1/playbook` — register, body `{ name, git_url, git_ref, playbook_path }`
- `GET /api/v1/playbook/{pid}` — full record incl. cached schemas
- `PATCH /api/v1/playbook/{pid}` — update name / git_ref (triggers refresh)
- `POST /api/v1/playbook/{pid}/refresh` — re-fetch + re-cache schemas
- `DELETE /api/v1/playbook/{pid}` — refuses 409 if any assignment references it

## Context
- Use `LibGit2Sharp` (already in the .NET ecosystem; check `dotnet list package` first to see if a sibling project already depends) for git clone. Fallback to shelling out to `git` if LibGit2Sharp is not desirable.
- `argument_specs.yml` is YAML; use `YamlDotNet` (already used by the AMQP serializer in this codebase — see `ConsumerService.cs`).
- Translate `argument_specs.yml` into JSON Schema following the mapping shown in `PLAYBOOK-INTEGRATION-PLAN.md` → "Schema Conventions". Keep the helper pure (testable).

## Steps

1. **Scratch dir**: `/var/lib/cloud-manager/playbook-meta/<pb_public_id>/`. Created by the API service user (`cloudmanager`). Owner + 0750. Cleared on `refresh`.

2. **`PlaybookService.RegisterAsync(name, gitUrl, gitRef, playbookPath)`**
   - Validate name unique (case-sensitive per plan's `varchar(255) unique`).
   - Insert row with `argument_schema = {}` and `output_schema = null` placeholder.
   - Call `RefreshAsync(publicId)` synchronously so the row is returned with cached schemas in one POST.

3. **`PlaybookService.RefreshAsync(publicId)`**
   - Clone shallow: `git clone --depth 1 --branch {git_ref} {git_url}` into the scratch dir.
   - Locate `meta/argument_specs.yml` relative to the cloned repo root (NOT relative to `playbook_path` — argument_specs lives at role level). If absent → reject with 400 "missing meta/argument_specs.yml".
   - Parse YAML → translate to JSON Schema via the helper.
   - Locate optional `meta/outputs.yml` — if present, store verbatim (it's already JSON-Schema-ish).
   - Update `argument_schema`, `output_schema`, `last_refreshed_at = NOW()`.
   - Delete scratch dir on success.

4. **`PlaybookController`** — six action methods matching the endpoints above. All decorated `[RequireFeatureFlag("playbooks")]`. DELETE checks `vm_playbook_assignments` and returns 409 + message if any rows reference this playbook.

5. **MCP support** (nice-to-have, not strict): if there's a `cloud-manager-mcp` to extend, add `playbook_register / playbook_list / playbook_get / playbook_refresh / playbook_delete`. Defer if MCP changes need their own restart cycle — note in Wishlist.

6. **Build + deploy** API.

## Acceptance Criteria
- Registering a real playbook (use `https://github.com/geerlingguy/ansible-role-postgresql` as a known-good sample, ref `master`, path `tasks/main.yml`) returns a row with `argument_schema` populated.
- `GET /api/v1/playbook/{pid}` returns the JSON Schema with `postgresql_version`, `postgresql_databases`, etc.
- DELETE refuses (409) when an assignment exists.
- DELETE succeeds (204) when none.

## Test
`Test/0004-ANSIBLE-SHENANIGANS-TEST.md`

## On Failure
The geerlingguy role uses Ansible's `argument_specs.yml` heavily — if the translator chokes, capture exact YAML node it failed on, write Improvise.
