# Lessons: cloud-manager/fearless-heron (indigo-torpedo workflow)

## What would have helped

The workflow itself ran cleanly — every test passed first try, no FIX files needed. The scaffold-vs-execution friction all happened *before* `/loop` ran, in three earlier scaffold-refining PRs (#18 initial, #19 refine, #20 from a parallel session, #22 consolidated). A few notes for future workflows:

1. **The task-file specs were detailed enough that planning execution was mostly transcription.** This is a feature, not a bug — but it means tasks like 0002 (API-PLAN) and 0003 (WEB-PLAN) had little real design work happening during `/loop`; that work was done at scaffold time. For future planning workflows, consider whether the task spec should be *less* prescriptive (let the executing agent make design choices) or *more* prescriptive (the executing agent is just a transcriber). The current scaffold leans heavily on the latter, which works but loses some of the "design happens during execution" value.

2. **The `secret_inputs[]` design wasn't in any prior plan.** It emerged during scaffold refinement when the API-PLAN's materialization endpoint had to specify *who* resolves `x-secret-path` inputs (worker, not API) and *how* the worker knows which paths to resolve. The bundle now has it explicit in both API-PLAN and VORCH-PLAN. Good outcome from the audit step; missing from the original `PLAYBOOK-INTEGRATION-PLAN.md`.

3. **The `playbook-run-cancel` queue is new shared infrastructure** that must be implemented on both sides together (API publisher + worker subscriber). Documented in both plans, but the implementation workflows for API and vorch should explicitly synchronize on this — if API ships cancel first, the endpoint exists but does nothing observable; if vorch ships cancel-handling first, it subscribes to a queue nothing publishes to. Suggest the API and vorch implementation workflows include a checkpoint task to verify the other side has landed before declaring done.

## Fix files written
None.
