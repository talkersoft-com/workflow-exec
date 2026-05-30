# Lessons: cloud-manager/snowy-omelette

## What worked
Soft-delete (PR #19) being already in place meant FK ON DELETE RESTRICT semantics held without rework — every `vm_events` row will always find its parent because the parent is never hard-deleted. The hive-deck workflow scaffold made phase boundaries crisp: 12 tasks, 12 tests, one box per phase, easy to track. The append-only event table + projection-only timeline endpoint were trivially safe to add behind the existing service layer; no schema changes to existing tables. Parallel fan-out across API/MCP/web for phases 7-10 cut wall-clock roughly in half because none of those phases shared files. The best-effort `try/catch` around `RecordEventAsync` calls kept the audit log strictly additive — a logging failure can never break a VM operation.

## What would have helped
1. The workflow scaffold was first authored on `raging-numbat`, then the deck advanced to `snowy-omelette` — a "lift scaffold" step (commit 3cad789) was needed because `hv_workflow` names the workflow path by current branch and there's no auto-rename on deck advance. A first-class rename/lift command would have removed that manual step.
2. Phase 11 task prompt was authored against assumed REST conventions, but the actual controllers don't follow them: VM create lives at `POST /api/v1/virtualmachine/create/{HostPublicId}` with a `{virtualMachine, vmi}` body, list is `/list`, and destroy is on a different controller entirely (`DELETE /api/v1/orchestration/destroy/{publicId}`). Capturing the real routes during PLAN.md authoring (e.g. by grepping `[HttpPost]`/`[HttpDelete]` attributes in the API repo) would have avoided wasted E2E iterations.
3. `EventType` is stored as `varchar(64)` rather than a PG enum or check constraint; if a typo'd value ever appears the DB won't catch it — application-layer `ToSnakeCase`/string-literal callsites are the only guard. Worth a follow-up.
4. `destroy_requested` is written inside the consumer success branch alongside the terminal event, so the timeline never shows "destroy was requested but we don't know the outcome yet". For an audit log, intent should be recorded at the point of intent (in the destroy endpoint, before publishing AMQP). This came up in the E2E run and is captured as a follow-up.

## Fix files written
- FIX-001.md — Phase 11 E2E task prompt referenced REST routes that don't match the actual controllers; tests passed once real routes were used. Also noted the `destroy_requested`-at-ack-time timing surprise. Audit feature itself was not broken.
