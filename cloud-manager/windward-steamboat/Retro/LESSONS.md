# Lessons: windward-steamboat

1. **Parallel research before contracts paid off.** Five concurrent read-only research passes
   (entities/prefixes, MCP surface, web stack, porch pipeline, house plan style) produced the
   raw facts the contracts file needed; grep-verifying the prefix registry afterward caught
   nothing — but it is the right cheap guard, since one collision would invalidate all nine
   plans at once.

2. **Contracts-first genuinely prevented drift.** Writing C-VAR/C-RES/C-ED before any plan
   meant P5 and P8 could cite §7/§8/§9 verbatim instead of paraphrasing; the consistency pass
   found only 4 drift points, all wording-level (no structural contradictions across ~1,400
   lines of plans).

3. **The drift that did occur was in the contracts file, not the plans.** Three of four fixes
   loosened over-tight contracts wording ("API-read surfaces only", "new fields are optional")
   that the more carefully reasoned plans had legitimately outgrown. Lesson: write contracts
   permissive about *shape of additions* and strict about *existing-surface invariants*.

4. **Read-model vs write-path framing resolved the P2/P3 tension cleanly.** Epic §2 demands
   both decomposition (P2) and round-trip fidelity (P3) — explicitly stating "P2 = lossy
   derived index, P3 = fidelity write path" in both plans removed the contradiction reviewers
   would otherwise flag, and produced the natural deferral of structured-edit endpoints from
   P2 to P3.

5. **Per-plan open questions with defaults keep the series reviewable.** The series-level plan
   carried zero open questions; pushing each decision (sidecar-vs-porch, flag naming, lint
   gating defaults) to the owning plan with a recorded default lets the human approve
   plan-by-plan without re-litigating the series.

6. **Existing patterns covered every new need.** Nothing in the nine plans required a new
   architectural pattern: events copy VmEvent, the lint queue copies collection-installs,
   secret refs ride SecretInputs, the editor swap reuses the documented lazy-boundary upgrade
   path. When a plan seems to need a new pattern, the research pass probably missed an
   existing one.
