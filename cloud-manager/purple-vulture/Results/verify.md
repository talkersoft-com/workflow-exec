# Browser verification — purple-vulture

All three close paths exercised against `https://ubuntu-server.talkersoft.com` after deploy.

## Flow A — list → editor → save → close → list ✅
- Start: `/ansible/playbooks`
- Clicked the new ✎ pencil icon on the `pg` row → editor opened at `/ansible/playbooks/pb_EMEJCH08GG/edit`.
- Typed a probe edit into the YAML textarea (`--- # probe — purple-vulture Flow A …`).
- **Note**: clicking the Save button was intercepted by the AttachedRolesPanel `<h4>` (existing layout quirk unrelated to the close/referrer changes — see LESSONS).
- Used the editor's existing `Ctrl+S` keybinding to save instead. Server returned success.
- Clicked Close → landed at `/ansible/playbooks` (referrer from `location.state.from` honored).
- Screenshot: `Results/flow-a-on-list.png`.

## Flow B — detail → editor → dirty close → cancel keeps user, accept lands on detail ✅
- Start: `/ansible/playbooks/pb_EMEJCH08GG`
- Detail page showed the new `✎ Edit` button in the header — clicked it.
- Editor opened at `/ansible/playbooks/pb_EMEJCH08GG/edit`.
- Typed a new edit (`--- # probe — purple-vulture Flow B (unsaved) …`) WITHOUT saving.
- Clicked Close → browser confirm appeared with message **"Discard unsaved changes?"**.
- Chose Cancel → URL stayed at `/edit`. Confirmed dirty buffer preserved.
- Clicked Close again → same confirm appeared.
- Chose OK → landed at `/ansible/playbooks/pb_EMEJCH08GG` (the referrer).
- Screenshot: `Results/flow-b-on-detail.png`.

## Flow C — cold-load `/edit` → close → detail (fallback) ✅
- `browser_navigate` directly to `/ansible/playbooks/pb_EMEJCH08GG/edit` (no `location.state`).
- Clicked Close (no dirty state, no confirm).
- Landed at `/ansible/playbooks/pb_EMEJCH08GG` — the detail-page fallback because `location.state.from` was undefined.
- Screenshot: `Results/flow-c-on-detail-fallback.png`.

## Sub-nav sanity ✅
Throughout all three flows, the Ansible sub-nav (`Playbooks / Roles / Run / History`) remained visible above the editor with the **Playbooks** tab active (the existing prefix-matching active-tab logic in `AnsibleSubNav` handles `/ansible/playbooks/:pid/edit` correctly).
- Screenshot: `Results/sub-nav-during-edit.png` (captured during Flow A).

## Revision-count delta ✅
Pre-Flow-A: `SELECT count(*) FROM vm.playbook_revisions WHERE playbook_id = (SELECT id FROM vm.playbooks WHERE public_id = 'pb_EMEJCH08GG')` → **0**.
Post-Flow-A save (via Ctrl+S): same query → **1**. Delta = **+1**. The PATCH-creates-revision behavior on the API side is unchanged by this workflow.

## Source-edit scope ✅
End-of-Phase-4 `git status --porcelain` for the four code repos matches end-of-Phase-2 — only `cloud-manager-web` has dirty files, and only the four expected files (`RoleFileEditor.tsx`, `AnsiblePlaybookEditorPage.tsx`, `PlaybookDetailPage.tsx`, `PlaybooksPage.tsx`). No backend, no vorch, no scope creep.
