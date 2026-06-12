# Lessons: cloud-manager/ansible-p2-decomposition

## FIX files
None — every task's test battery passed on its first complete run; no retries were needed.

## What would have helped
1. **The contracts file was missing from main.** `Exec.md` and `PLAN.md` both require
   `master-plans/ANSIBLE-EPIC-CONTRACTS.md`, but workflow-plans main only has
   `ANSIBLE-EPIC.md`. The restore commit (`73552d1`, "restore ansible-p2-decomposition plan
   from enjoyable-tegu") brought back the plan folders but not the contracts file they cite.
   It was recovered from `origin/enjoyable-tegu` (`3ab4e0e`). Lesson: when restoring plans
   from an unmerged branch, restore the shared master-plan files they reference in the same
   commit — and the P3–P9 executions will hit this same gap until the contracts file lands
   on main.
2. **Relative input paths in Exec.md are off by one.** `../../../workflow-plans/...` from
   `Execution/` resolves inside workflow-exec; the plans repo is a sibling repo
   (`../../../../workflow-plans/...`). Scaffold templates should compute these from the deck
   layout.
3. **Task 0004's "record events" conflicts with contracts §1/§6** (no new P2 entities, no
   playbook event aggregate). Resolved by recording failures on the revision row
   (state + error) — durable and API-visible — and documenting the deviation in RESULT.md.
   Task authors should name the target event table when one is expected.
4. **The API hardcodes its bind URL** (`UseUrls("http://ubuntu-server…:5250")`), so a second
   instance of the new build can't run beside the production service for HTTP-level testing.
   Verification used a service-layer harness against the live DB pre-deploy, with the
   HTTP-level diff done post-deploy (baseline captured from the old build first). Making the
   bind URL configurable would allow true side-by-side regression.
5. **Test semantics worth pinning in future task files:** "saving again replaces them" was
   implemented and verified as both (a) re-decomposing the same revision is idempotent and
   (b) the new head's record set matches the new content — old revisions keep their records
   by design (record-level history is a P2 goal).

## What went well
- The P1 inventory plan's conventions (entity/config/migration/controller/MCP patterns) made
  every layer of P2 mechanical to mirror.
- Delete-and-recreate inside one transaction, with edges keyed by internal source-revision
  anchors, kept reverse-index consistency trivial to reason about and test.
- The live decompose-all backfill processed the real seed data cleanly on the first run
  (2 playbook heads, 26 tasks, 63 edges, 0 failures).
