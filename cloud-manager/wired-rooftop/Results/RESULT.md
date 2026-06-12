# Result: cloud-manager/wired-rooftop

## Outcome
Complete. The blueprint composer is reachable without URL memory on the deployed app:
every `/blueprints` row has an explicit Edit action opening `/blueprints/{id}`, and creating a
blueprint lands the user directly in its composer.

- Execution branch: `brittle-peach` (created by Task 0000 via hv_next from origin/main)
- Code change: `cloud-manager-web/src/pages/BlueprintsPage.tsx` only — one extra `<th>` and a
  right-aligned ghost `Edit` button per row (Playbooks-list pattern, existing `Button` component,
  no new dependencies).
- Scope note: the clickable row name → `/blueprints/{id}` and the create-modal success
  navigation (`navigate('/blueprints/' + created.id)`) were already present on origin/main
  (shipped with the marketplace UI merge, PR #17 stringy-nymph). The only missing affordance was
  the explicit per-row Edit button; the diff is exactly that.

## Test evidence
### Test 0000
- TC-001 ✓ all 18 repos on `brittle-peach`, none on default
- TC-002 ✓ clean except scaffold's own runtime edits in workflow-exec (excepted)
- TC-003 ✓ deck.md Branch = `brittle-peach`, matches hv_status

### Test 0001
- TC-001 ✓ `python3 .cicd/build-cloud-manager.py --target web` green (812 modules, built in 3.8s)
- TC-002 ✓ diff touches only `BlueprintsPage.tsx`; no new dependencies; routes/slices/api
  client/flag gating diffs are zero

### Test 0002
- TC-001 ✓ deploy `--target web` green; smoke: service up after 3s, HTTP 200 on `/`
- TC-002 ✓ on `https://ubuntu-server.talkersoft.com` (the deployed origin — see Retro/FIX-001.md):
  - Edit on the `jammy` row → landed on `/blueprints/bp_f344a5b2e2`, full composer rendered
    (metadata form, playbook attach, Archive/Delete)
  - Created scratch blueprint `wired-rooftop-scratch` via the modal → landed directly on
    `/blueprints/bp_JV5BK04TZ8`, composer rendered with Draft badge
  - Scratch cleaned up via Delete (confirm dialog accepted); list back to the original 3 rows
- TC-003 ✓ regression: clickable row name still navigates to the composer; Publish/Archive/Delete
  render on the detail page (Draft shows Publish+Delete, Published shows Archive+Delete) and the
  Delete path was exercised end-to-end via the scratch blueprint. Note: publish/archive/delete
  never lived on the list rows — they are detail-page actions; list rows carry name + status
  badge + Edit. Flag end-state: `marketplace` was already enabled+healthy before verification and
  was not toggled (left as found).
- TC-004: PR links recorded below; merge status reported by the operator-facing summary.

## Pull requests
- cloud-manager-web PR #18 — https://github.com/talkersoft-com/cloud-manager-web/pull/18 (merged)
- workflow-exec PR #47 — https://github.com/talkersoft-com/workflow-exec/pull/47 (merged)

## Artifacts
- Screenshot: `.playwright-mcp` output `wired-rooftop-blueprints-list-edit-affordance.png`
  (deployed /blueprints with Edit column)
- `Retro/FIX-001.md` — verification-origin diagnosis (localhost:3000 has no API proxy; the
  deployed origin is https://ubuntu-server.talkersoft.com)
