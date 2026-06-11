# Lessons: cloud-manager/lucky-engineblock

## What would have helped
1. **A documented "guest readiness" signal.** vorch acks `create-vm` Success when the libvirt
   domain is up, not when sshd accepts connections. Anything that auto-runs playbooks needs the
   latter. The sequencer now polls TCP :22 itself (FIX-003); a vorch-emitted "guest-ready" event
   (cloud-init phone-home) would be the clean long-term signal and would also remove the known
   limitation that an API restart during the wait window drops a pending chain start.
2. **The direct apply path (`PlaybookAssignmentController.Apply` → `EnqueueAsync`) creates
   zero-target runs that porch no-ops** ("zero targets; marking succeeded (no-op)") and the run
   row stays Queued in the DB forever (rollup is target-driven). Pre-existing, not caused by this
   plan — every working run in the DB went through `TriggerMultiVmAsync`. Worth a follow-up plan:
   either make `EnqueueAsync` create the target (one-liner family) or retire the endpoint in
   favor of `cloud_playbook_run`.
3. **The deploy script's api smoke test could never pass** (`/api/v1/Host` has no route; only
   `/list` does). Either it was never exercised, or deploys were "failing" green by hand. Smoke
   URLs belong next to the routes they test, or better: hit `/api/v1/health/check`.
4. **`Program.cs` hardcodes `UseUrls(...:5250)`**, which defeats `ASPNETCORE_URLS`/`--urls` and
   makes it impossible to run a second instance for testing without deploying. A follow-up
   one-liner (respect env override when present) would make future workflow testing much safer.
5. **Orchestration scaffold paths drifted** — the ORCH.md path the operator passed didn't exist
   because plan approval (hv_plan_approve) had not been run yet; the scaffold had to be created
   from the approved plan first. The `hv_workflow_*`→`hv_orchestrate*` rename context in memory
   helped diagnose this quickly.

## Fix files written
- FIX-001: second API instance can't bind (hardcoded 5250) → early flag-gated deploy; tests ran against deployed API
- FIX-002: deploy api smoke test polls a route that doesn't exist → corrected to /api/v1/Host/list
- FIX-003: sequencer raced guest SSH readiness → background :22 poll (5 min cap) before chain start
