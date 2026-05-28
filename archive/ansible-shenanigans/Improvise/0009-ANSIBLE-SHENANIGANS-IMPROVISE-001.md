# Improvise: Phase 0009 drift detection placement + minimal UI

## Phase
`0009-ANSIBLE-SHENANIGANS`

## Placement change vs plan
The task spec said the worker does the re-clone + classify, then somehow the API returns 422. That's contradictory: the API has already returned 202 by the time the worker runs.

I put the drift detection on the **API** `/apply` endpoint instead — it calls `IPlaybookService.DetectDriftForAssignmentAsync` (which does a clean fresh-clone + re-translate + classify against the cached `argument_schema`). Breaking → 422 synchronously; otherwise enqueue the run as before. This matches "API returns 422" semantics cleanly without needing a Cancelled state on the run row.

## Built
- `SchemaDiff.cs` — pure classifier function `Classify(old, new, varsOverride) → DriftResult`. Categories: newly-required-without-override (breaking), removed-but-in-vars (breaking), type change with override (breaking), choices narrowing that excludes current (breaking). Everything else (newly-added optional, expanded enums, removed unused) is non-breaking. Dedupes when a field is in both categories.
- `IPlaybookService.DetectDriftForAssignmentAsync` — fresh-clone, re-translate, compare to cached `argument_schema`, then write the fresh schema back (cache freshening per TC-006).
- `PlaybookAssignmentController.Apply` — calls drift detection first; 422 with `{message, breaking, nonBreaking}` body on breaking; log warning on non-breaking; enqueue otherwise.
- Web `VmPlaybooks.handleApply` — parses 422 body and toasts a structured breaking-fields list. Full adopt-modal flow not built (left for the user to specify).

## Test fixtures
Created two branches on the hermetic test repo at `/var/lib/cloud-manager/test-playbook`:
- `drift-test-broken` — adds `postgresql_admin_email` as a required field with no default
- `drift-test-additive` — adds optional `log_level` with default

## Verification (passes I can drive)
- TC-001 ✅ master ref → 202 → run Succeeded
- TC-002 ✅ drift-test-broken → HTTP 422 with `breaking: [{field: postgresql_admin_email, reason: "now required, not provided in vars_override"}]`
- TC-005 ✅ drift-test-additive → 202 (run enqueued) + API log: `non-breaking drift on assignment asgn_X: 2 entries`
- TC-006 ✅ After each apply, `last_refreshed_at` advances and the new field (e.g. `postgresql_admin_email`) appears in `argument_schema->'properties'`

## Visual-only TCs (deferred)
- TC-003 modal renders — I added a structured toast for breaking drift rather than a full modal. The user can build the modal if the toast UX is insufficient.
- TC-004 "Adopt new schema" flow — not built. Requires per-field editing UI to fill the new required field. Toast tells the user what's needed; they manually PATCH the assignment for now (or extend the modal later).

## State
- Test playbook reset to `git_ref=master` so subsequent runs work without intervention.
- Drift-test branches kept for re-testing.
