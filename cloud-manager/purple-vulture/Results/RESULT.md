# Result: cloud-manager/purple-vulture

## Outcome
**SHIPPED** (pending final hv_ship in this task)

## Branch
`oaty-polliwog` (execution; planning was on `purple-vulture`)

## Pull Requests
| Repo | PR | Status |
|------|----|--------|
| `cloud-manager-web` | TBD — filled from hv_ship output | — |
| `workflow-exec` | TBD — filled from hv_ship output | — |

## Phase summary

### Phase 0 — Initialize
Deck `cloud-manager` already on execution branch `oaty-polliwog` (auto-transitioned from prior planning ship). All 15 repos clean. Recorded branch in `deck.md`.

### Phase 1 — Editor close + referrer + dirty-guard + layout
Files: `RoleFileEditor.tsx`, `AnsiblePlaybookEditorPage.tsx`. Added `onDirtyChange?: (dirty: boolean) => void` prop to `RoleFileEditor`; wired it from a `useEffect` watching the existing `dirty` boolean. In the editor page: imported `useLocation` and `useNavigate`, added `[dirty, setDirty]` state, page-level `beforeunload` guard, `close()` handler with `window.confirm` on dirty and `/ansible/`-prefix defensive guard with detail-page fallback, Close button next to Revisions, and replaced `height: calc(100vh - 120px)` with `display: flex; flexDirection: column; height: 100%; minHeight: 0`. See `Retro/CHANGE-0001.md`. Build clean (0 errors).

### Phase 2 — Entry points
Files: `PlaybookDetailPage.tsx`, `PlaybooksPage.tsx`. Detail page got a primary `✎ Edit` button in its header; list rows got a `✎` icon button before the existing Delete button. Both call `navigate(..., { state: { from: { pathname, search } } })`. List-row button uses `stopPropagation()` (the row's name `<a>` was already the click target; the pencil sits in the trailing cell). See `Retro/CHANGE-0002.md`. Build clean.

### Phase 3 — Build + deploy
`sudo rm -rf dist && npm run build` clean; `sudo python3 cloud-manager-api/scripts/web/install-web-app.py` finished with health-check passing. `cloud-manager-web` service `active`. Deployed bundle (`/opt/cloud-manager-web/assets/index-BmT04mKa.js`) contains the new `"Discard unsaved changes"` literal. Note: Test 0003 TC-003 had the wrong path (`/opt/cloud-manager-web/dist/assets/*.js`) — the deployed layout drops the `dist/` prefix; corrected in-place.

### Phase 4 — Browser verification (Playwright)
All three close paths verified against `https://ubuntu-server.talkersoft.com`:
- **Flow A** (list → ✎ → save via Ctrl+S → close → list): URL ended at `/ansible/playbooks`. Revision count `0 → 1`. `Results/flow-a-on-list.png`.
- **Flow B** (detail → Edit → dirty → close → Cancel → still on editor → close → OK → detail): browser confirm appeared with `"Discard unsaved changes?"`; both Cancel and Accept behaved correctly. `Results/flow-b-on-detail.png`.
- **Flow C** (cold-load `/edit` → close → detail fallback): no `location.state`; landed at `/ansible/playbooks/pb_EMEJCH08GG`. `Results/flow-c-on-detail-fallback.png`.
- Sub-nav stayed on Playbooks throughout. `Results/sub-nav-during-edit.png`.
- Full notes in `Results/verify.md`.

### Phase 5 — Ship
Final RESULT.md and LESSONS.md written; `hv_ship` invoked from this task. PR URLs filled in from ship output.

## Revision-count delta
Before Flow A: `count = 0`. After Flow A's Ctrl+S save: `count = 1`. Delta = **+1**. Backend revision-on-PATCH unchanged.

## Service status post-fix
- `cloud-manager-web` — active
- (API, vorch, RMQ unchanged — not part of this workflow's scope)
