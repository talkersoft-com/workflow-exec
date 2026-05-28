# Result: Craft Ingot

> EXAMPLE FILE — This shows the pattern for a result file. Write a result at the conclusion of every task or orchestration — success or failure. Do not modify this file.

## Result ID
`0001-CRAFT-INGOT-RESULT`

## Status
✓ SUCCESS

## References
- Orchestration: `Orchestrate/0001-CRAFT-INGOT-ORCH.md`
- Task: `Tasks/0001-CRAFT-INGOT-TASK.md`
- Test: `Test/0001-CRAFT-INGOT-TEST.md`
- Improvisation: `Improvise/0001-CRAFT-INGOT-IMPROVISE-001.md`

## Summary
Iron ingot successfully crafted. Primary furnace was unavailable — blast furnace used instead per improvisation 001. All test cases passed on second attempt.

## Test Results
- TC-001: Ingot Present ✓
- TC-002: No Ore Lost ✓
- TC-003: Furnace Empty ✓

## Improvisation Used
`Improvise/0001-CRAFT-INGOT-IMPROVISE-001.md` — Furnace unavailable, switched to blast furnace on level 3.

## Duration
~6 minutes including one failed attempt and adaptation.

## Notes
Blast furnace is now the documented fallback for all smelting workflows. Consider updating the orchestration to prefer blast furnace when the primary is occupied rather than failing first.

---

> PATTERN NOTE — Results are always written. A result with status FAILURE is not a bad outcome — it is an honest record. Include what failed, what was tried, and what the next step should be. A result with no improvisation reference means the task succeeded on the first attempt. A result that references an improvisation shows the system working as intended — failure was handled, documented, and adapted.
