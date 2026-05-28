# Improvise: Phase 0005 cross-cutting fixes + visual verification gap

## Phase
`0005-ANSIBLE-SHENANIGANS`

## What happened during Phase 0005

### 1. ToastData/EmptyState API mismatch
The plan assumed `addToast({variant, message})` and `<EmptyState title="..." message="..."/>` shapes. Actual:
- `ToastData.title` is REQUIRED; `message` is optional
- `EmptyState` uses prop `description` not `message`

Fixed by sweeping all toast calls to include `title`, and all `EmptyState message=` to `description=`.

### 2. dist/ permission-locked from prior deploys
`npm run build` failed with `EACCES /home/todd/.../dist/assets`. The web install script runs as root and the resulting dist tree was root-owned. Cleared with `sudo rm -rf dist` then rebuilt as todd.

### 3. Backend prerequisites pulled forward
Test cases TC-004 + TC-006 require `POST /api/v1/virtualmachine/{vmId}/playbook` and `DELETE .../playbook/{asgnId}` — endpoints the plan slotted for Phase 0006 (worker). Built them now (PlaybookAssignmentController + PlaybookAssignmentService) so the UI test cases could exercise the full attach/detach flow. The worker glue in Phase 0006 only needs to add the "queue a run" surface, not the assignment CRUD.

## Visual verification gap (important)

I (the agent) cannot drive a browser. TCs that require visual inspection of the rendered UI were satisfied indirectly:
- **TC-001 gating off** → verified at the API level (both `/playbook/list` and `/virtualmachine/X/playbook` return 404 when flag is off). `<FeatureGate flag="playbooks">` wraps every UI surface (App.tsx nav guarded by `selectIsFeatureActive`; `PlaybooksPage`, `PlaybookDetailPage`, `VmPlaybooks` each wrapped).
- **TC-002/003 gating on** → bundle string-check confirms "Playbooks", "Attach playbook", "Register new", "rjsf" are present in `/opt/cloud-manager-web/assets/index-*.js`.
- **TC-004/006 attach/detach** → end-to-end via curl: attached `pg` → `asgn_10XJP0MX0F` (varsOverride preserved), detached → count=0.
- **TC-005 Apply stub** → confirmed by checking the bundle contains `Execution lands in Phase 0006`; no `/apply` route exists server-side so an accidental call would 404.

**User should still load the web at `https://ubuntu-server.talkersoft.com` (or `http://localhost:3000`), flip the flag in DB, and visually confirm the Playbooks nav link + VM detail Playbooks section render.** The bundle is the right shape; the gate logic is the right shape; but only the user can confirm the pixels.

## State
- Flag reset to `(enabled=false, healthy=false)` per the test doc requirement.
- One playbook still registered (`pg` → `pb_EMEJCH08GG`), no attachments.
