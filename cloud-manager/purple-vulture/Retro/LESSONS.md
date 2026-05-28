# Lessons: cloud-manager/purple-vulture

## What would have helped

1. **Test files I scaffolded had an incorrect deployed-bundle path.** Test 0003 TC-003 used `/opt/cloud-manager-web/dist/assets/*.js` but the actual deployed layout is `/opt/cloud-manager-web/assets/*.js` (no `dist/` prefix — `install-web-app.py` flattens it). I corrected the test in-place during execution. Recommendation: before authoring a Test TC, verify the literal command runs on a current deployment, not just on the build output tree.

2. **Existing layout overlap between editor and AttachedRolesPanel.** With the editor's flex layout and a typical viewport, the AttachedRolesPanel `<h4>` element renders over the Save button's hit-test area — Playwright's click was repeatedly intercepted by `<h4>Attached roles</h4>`. The Save button is visually present and reachable via `Ctrl+S`, so this was a verification-only blocker, not a functional one for end users (a real user clicks the button position they see). Whether the overlap was pre-existing or introduced by the `calc(100vh - 120px)` → flex change is not fully isolated — it warrants a small follow-up plan to add explicit `position: relative; z-index: 1` on the editor pane, or to drop a `min-width: 0` somewhere in the chain so the Save toolbar sits cleanly above the sidebar. Not in scope for this PR.

3. **Playwright `browser_handle_dialog` is post-hoc.** Clicking Close fires `window.confirm` which suspends the page; Playwright surfaces the dialog in the next snapshot, and `browser_handle_dialog` resolves it. Works cleanly but is two tool calls per dialog — fine for a verification run, not great if we wanted to script dozens of these.

## Fix files written
None. The Test 0003 TC-003 path correction was an in-place edit of the test file (not a code FIX), so no `FIX-NNN.md` was opened.

## Deviations from plan
- **`window.confirm` chosen over a custom `ConfirmDialog`.** The plan recommended `window.confirm` unless a `ConfirmDialog` already existed. The grep confirmed none exists. Used `window.confirm` as recommended.
- **Save button click had to fall back to `Ctrl+S` during Flow A verification.** The functional path (save → revision created → close → land on referrer) works end-to-end; the Save *visual button* hit-test interception is a small follow-up. Documented in #2 above and in `Results/verify.md`.
- All other plan choices kept as recommended (text-label Close button, two entry points only, `stopPropagation` on the list pencil).
