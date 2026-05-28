# Fix VM-creation error in cloud-manager

## What this workflow does
Reproduces the failure operators see when creating a VM in cloud-manager, walks the captured evidence end-to-end (browser → cloud-manager-api → vorch_service → libvirt) to a single root cause with file/line precision, ships the minimal targeted fix to the responsible repo(s), and redeploys via the standard `install-*.py` scripts on `ubuntu-server.talkersoft.com`. End state: a user can submit the Create VM form and the VM enters provisioning successfully — verified by re-running the exact reproduction from Phase 1.

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns
- `../../../workflow-plans/cloud-manager/blizzard-sparrow/PLAN.md` — source plan (diagnose-then-fix structure)

## Constraints
- **No fix speculation in Phases 0–1.** Phase 2 is where commitments are made. Until reproduction artifacts are written to `Retro/REPRODUCTION.md`, no code may be edited.
- **One bug, one PR per affected repo.** No opportunistic refactors. If you spot adjacent bugs during reproduction, open a separate plan — do not bundle.
- **Do not disable feature flags or remove guards as a workaround.** Fix the cause, not the symptom.
- **Surface real errors to the user.** If the root cause is on the vorch side, the libvirt error must reach the API/UI. No silent fallbacks.
- **Scoped to VM-creation only.** Other VM operations (delete, snapshot, etc.) are out of scope even if a related bug surfaces.
- **All hive deck operations go through MCP tools** (`hv_status`, `hv_init`, `hv_next`, `hv_ship`). Never raw `git` to commit/push/PR repos in the deck.
