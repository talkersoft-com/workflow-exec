# Lessons: cloud-manager/wired-rooftop

1. **Verify against the real deployed origin, not the backend service port.**
   `localhost:3000` is the raw `serve -s` static host behind nginx; it has no `/api` proxy, so
   the SPA fallback feeds HTML to every API call and FeatureGate silently degrades to
   "disabled". The deployed origin is `https://ubuntu-server.talkersoft.com`, which serves the
   same bundle and proxies `/api/v1/*` to the API on `localhost:5250`. Future workflows should
   point playwright there from the start (FIX-001).

2. **Check what main already has before implementing a plan's delta.** The plan's background
   ("list renders no link, create stays on the list") was stale: origin/main already shipped the
   clickable row name and create-to-composer navigation in the marketplace UI merge (PR #17).
   The true remaining gap was only the explicit per-row Edit button. Reading the current page
   source before coding kept the diff to 8 lines instead of re-implementing existing behavior.

3. **Plan/task wording can misplace actions.** Task 0001/Test 0002 referenced
   "publish/archive/delete row actions" on the list; those actions live on the blueprint detail
   page, not the list rows. Regression was interpreted accordingly: list actions = name link +
   Edit; lifecycle actions verified on the detail page (Delete exercised end-to-end via the
   scratch blueprint).

4. **A "Marketplace disabled" gate is not proof the flag is off.** The flag API said
   enabled+healthy while the UI showed disabled — the gate's fallback also fires when the flags
   fetch fails. Cross-check the API directly before toggling flags; here no toggle was needed
   and the flag was left as found.
