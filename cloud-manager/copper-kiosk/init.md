# Marketplace UI — storefront, builder, provision visibility (copper-kiosk)

## What this workflow does
Implements plan 2 of the marketplace series in cloud-manager-web: a `/marketplace` storefront
(browse published blueprints, create a VM in three clicks → watch it come up on the VM detail
page), `/blueprints` builder pages (compose image + ordered playbooks + vars, publish/archive),
VM-detail blueprint provenance + live run-chain progress via SignalR, and one explicitly-scoped
assessment phase answering whether vorch/porch status writebacks suffice for the UI (deliverable:
findings doc; a drafted `vorch@cloud-manager` follow-up plan ONLY if gaps exist).

The backend it consumes is live (lucky-engineblock, shipped): `/api/v1/marketplace` +
`/api/v1/blueprint` behind the `marketplace` feature flag, seeds `jammy` and `postgres-jammy`,
BlueprintRunSequencer auto-running chains after provision.

## Read before starting
- `deck.md` — scope, orchestration id, pre-written hv MCP calls
- `Orchestrate/ORCH.md` — task list and execution instructions
- `../../../workflow-plans/cloud-manager/copper-kiosk/PLAN.md` — routes, page designs, plumbing,
  assessment criteria; open-question defaults are DECIDED (modal instantiate; count-on-card with
  names in expandable detail; up/down reorder buttons, no new dependencies)
- `../lucky-engineblock/Results/RESULT.md` — what the backend actually shipped (sequencer
  behavior, SSH-wait, event names)

## Constraints
- Code changes in **cloud-manager-web ONLY**. No api/mcp/vorch/porch changes; Phase 0005 produces
  findings + at most a follow-up plan draft, never inline changes.
- Existing Create VM wizard and all `/ansible/*` pages byte-identical (regression in Phase 0006).
- Everything new gated on the `marketplace` feature flag (flag off = zero visible change).
- Follow web conventions: api client in `src/services/api.ts` (camelCase), Redux Toolkit slices
  (`vmsSlice` exemplar), SignalR via the existing `useSignalR` hook, SASS + theme system.
- Build per phase: `python3 .cicd/build-cloud-manager.py --target web`; deploy web before the
  final ship per the deploy fragment.
- Keep task/test count proportional per phase.
