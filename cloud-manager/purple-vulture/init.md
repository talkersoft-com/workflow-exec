# Integrate Ansible Playbook YAML editor into the UI

## What this workflow does
Adds the missing entry points and close affordance so the existing `/ansible/playbooks/:pid/edit` route is discoverable from the UI and returns the user to the page they came from on close. UI-only change in `cloud-manager-web` — no backend, DB, or vorch work. End state: operators can open a playbook from the list or detail page, click Edit, modify the YAML, save (which creates a `playbook_revisions` row server-side), and click Close to land back on the page they started from.

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns
- `../../../workflow-plans/cloud-manager/gingery-kingfisher/PLAN.md` — source plan with referrer-pattern design, layout fix, and unsaved-changes guard recommendation

## Constraints
- **UI-only.** No changes to `cloud-manager-api`, `vorch-service`, `vorch-lib`, or any database migration.
- **No `RoleFileEditor` internals.** Only add the new `onDirtyChange?: (dirty: boolean) => void` prop and the matching `useEffect` dirty-detection wiring. Internals of the editor stay as they are.
- **Editor must survive cold-load.** If `/ansible/playbooks/:pid/edit` is loaded directly (refresh, bookmark, deep-link, typed URL), Close must still work — falls back to the detail page when `location.state.from` is undefined.
- **Close stays inside `/ansible/*`.** Defensive guard against malicious `from` state — if `from.pathname` doesn't start with `/ansible/`, use the detail-page fallback.
- **Don't add a custom `ConfirmDialog` for v1.** `window.confirm` is acceptable. Only use a styled dialog if a `ConfirmDialog` already exists in the repo (Phase 1 verifies with a grep).
- **Two entry points only.** Detail page (header Edit button) + list rows (pencil icon). No edit shortcut on Run page, History page, or role-detail page.
- **All hv operations go through MCP tools.** Never raw `git` to commit / push / open PRs for repos in the deck.
