# Improvise: Phase 0004 test repo + outputs.yml JSON encoding

## Phase
`0004-ANSIBLE-SHENANIGANS`

## What the plan said
Task: "use `https://github.com/geerlingguy/ansible-role-postgresql` as a known-good sample, ref `master`, path `tasks/main.yml`".

## What actually happened
1. **No argument_specs.yml**: cloning geerlingguy's repo shows `meta/main.yml` (role metadata) and `defaults/main.yml` (variable defaults), but no `meta/argument_specs.yml`. The newer argument_specs convention isn't adopted by most older community roles.
2. **PrivateTmp blocks file:// URLs in /tmp**: the systemd unit sets `PrivateTmp=yes`, so the cloudmanager service sees its own private /tmp — a host-side `/tmp/test-playbook` is invisible to it. First clone attempt failed with `repository '/tmp/test-playbook' does not exist`.
3. **outputs.yml stored raw**: original RefreshInternalAsync wrote the YAML file contents verbatim into the `output_schema` jsonb column, which Postgres rejected with `22P02 invalid input syntax for type json`.

## Decisions
1. **Built a hermetic test repo** at `/var/lib/cloud-manager/test-playbook` (inside `ReadWritePaths`, visible to the sandboxed service) with a proper `meta/argument_specs.yml` covering: str+choices+default, list[dict] with nested options + required, int with default, bool with default. This exercises the full translator without depending on an external repo's schema.
2. **Kept /var/lib/cloud-manager** as the test repo location — already in `ReadWritePaths`, owned by cloudmanager, no extra sandbox grants needed.
3. **Added `YamlToJson` helper** to PlaybookService — outputs.yml is parsed as YAML then re-emitted as JSON before storing. Same transform we already do for argument_specs (translator already returns JObject/JSON).

## Verification
All 7 TCs pass against the hermetic repo:
- TC-001 → 201 with `pb_0AMFV9SZFW`, schema with 4 properties, last_refreshed_at set
- TC-002 → schema contains `postgresql_version` (string, default "16", enum), `postgresql_databases` (array, items.required=[name]), `postgresql_port` (integer, default 5432), `postgresql_enabled` (boolean, default true)
- TC-003 → list returns the playbook
- TC-004 → PATCH advanced last_refreshed_at by ~21s
- TC-005 → DELETE 204
- TC-006 → scratch dir empty after delete
- TC-007 → flag off → 404

## Followup
- Phase 0010 (end-to-end smoke) can use this same hermetic repo if it stays generic enough. If we want a real postgres install during Phase 0010, we'd need to fork geerlingguy/ansible-role-postgresql and add `meta/argument_specs.yml` ourselves — note this in the workflow Wishlist.
- For the production registry: warn users via UI when adding a repo missing argument_specs (better than 400 on submit).
