# Task: Schema diffing

## Task ID
`0009-ANSIBLE-SHENANIGANS-TASK`

## Parent Orchestration
`Orchestrate/0001-ANSIBLE-SHENANIGANS-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Steps

1. **Refresh-on-apply** (worker side): just before building `extravars`, re-clone the playbook and re-translate `meta/argument_specs.yml`. Compute diff vs. `playbooks.argument_schema` (the cached one). If different, persist the fresh schema back to `playbooks.argument_schema` AND `last_refreshed_at = NOW`.

2. **Classifier**: pure function `classifyDiff(old, new) → { breaking: [...], nonBreaking: [...] }`. Breaking includes: removed required field that's not in `vars_override`, type change incompatible with stored value, choices narrowing that excludes stored value. Non-breaking includes: added optional fields with defaults, expanded enums, loosened types.

3. **API behavior**:
   - Breaking: `422 Unprocessable Entity`, body `{ message: "schema drift", breaking: [...], nonBreaking: [...] }`. Do not mark the run anything; the run row stays Queued briefly then transitions to Cancelled with `error_message = "schema drift"`.
   - Non-breaking only: log warning, proceed.

4. **UI for 422**:
   - Toast with summary.
   - Modal showing diff (added / removed / changed fields).
   - Two buttons:
     - **Adopt new schema** — `POST /api/v1/playbook/{pid}/refresh` to ensure cache is fresh, then `PATCH /api/v1/virtualmachine/{vmId}/playbook/{asgnId} {vars_override: <updated>}`. User edits the form with the new schema before resubmitting.
     - **Pin to old git_ref** — open the playbook's `PATCH` view to lock `git_ref` to a SHA so the schema is frozen.

## Acceptance Criteria
- Apply against an unchanged playbook → runs (no diff).
- Apply when the playbook git_ref now has a new required field → 422 with diff body; assignment form shows the diff.
- Apply when the playbook git_ref added an optional field with default → succeeds; warning logged in worker.
- `argument_schema` cache updated after each apply (even successful ones), so the form always renders the latest.

## Test
`Test/0009-ANSIBLE-SHENANIGANS-TEST.md`
