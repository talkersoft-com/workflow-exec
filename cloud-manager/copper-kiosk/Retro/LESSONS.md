# Lessons — cloud-manager/copper-kiosk

1. **Assignment `lastRunStatus` is a writeback mirror, not a live status.** It is written only
   when porch PATCHes the run target — never at enqueue. Any UI needing the Queued phase must
   read the run rows (`GET /run?vmId=`). Cost us FIX-001; the plan's "assignment lastRunStatus
   for state" wording was subtly wrong and only a live watch of a real chain exposed it.

2. **The SignalR surface is narrower than "SignalR live updates" suggests.** The hub broadcasts
   exactly two things: `OrchestrationStatusUpdated` on create-vm acks and `LogLine` to per-run
   groups. There is no event at sequencer enqueue or target terminal writeback, so a purely
   event-driven chain panel is impossible today; RunDetailPage itself quietly polls while a run
   is active. We matched that pattern (bounded poll that stops at terminal state) and recorded
   the gap as a non-blocking observation in the Phase 0005 findings rather than drafting a vorch
   plan — the UI meets its spec without backend changes.

3. **VM provenance lives in events, not the VM DTO.** `blueprint_id` exists in the DB but is
   deliberately absent from the VirtualMachine response (lucky-engineblock kept response shapes
   unchanged). The `blueprint_instantiated` VmEvent carries blueprintId, blueprintName AND the
   instantiate-time runPlan — that snapshot is the right source anyway, since the blueprint can
   be edited after instantiate.

4. **Soft-deleted VMs orphan their assignments and block playbook deletion.** After destroying
   the TC-003 VM, its assignment row kept `ck-always-fail` undeletable (409 "has assignments")
   while the assignment endpoint 404s for the soft-deleted VM — an API dead-end worth a tiny
   backend follow-up someday. Test fixtures that attach playbooks should detach BEFORE
   destroying the VM (we did exactly that in the Phase 6 regression).

5. **Test the deployed app through its real front door.** `localhost:3000` serves the SPA but
   does not proxy `/notificationHub`, producing scary-but-irrelevant negotiation errors; the
   public URL proxies everything. Phase-6 e2e should always target the URL users actually hit.

6. **In-browser transition recording beats screenshot-polling for "live" claims.** A small
   injected sampler (1.5s interval, record-on-change) produced exact Pending→Queued→Succeeded
   timelines with timestamps and doubled as the no-manual-reload proof, since it runs inside the
   page session itself.

7. **The vite dev proxy made pre-deploy testing safe.** Phases 1–4 ran against the live API
   through `npm run dev` (port auto-bumped to 3001 — something else owns 3000; it turned out to
   be the deployed web service itself), so the deployed app was touched only once, in Phase 6.
