# Improvisation: Craft Ingot — Furnace Unavailable

> EXAMPLE FILE — This shows the pattern for an improvisation. When a task or test fails, write here before retrying. Do not modify this file.

## Improvisation ID
`0001-CRAFT-INGOT-IMPROVISE-001`

## Source
- Task: `Tasks/0001-CRAFT-INGOT-TASK.md`
- Orchestration: `Orchestrate/0001-CRAFT-INGOT-ORCH.md`
- Failed Test Case: `Test/0001-CRAFT-INGOT-TEST.md` — TC-001 (Ingot Present)

## What Failed
The primary furnace was occupied by another process. Loading ore caused a slot conflict. Smelting did not begin. TC-001 failed — no ingot produced.

## What Was Tried
1. Waited 30 seconds and retried loading — furnace still occupied
2. Attempted to locate a secondary furnace on level 2 — not found

## Adaptation
Switched to the blast furnace on level 3, east wall. It accepts the same ore input, same fuel, and produces the same ingot output at 2x speed. No changes to the task steps were required — only the target furnace changed.

## Outcome
Task completed successfully using blast furnace. TC-001, TC-002, and TC-003 all passed. Result written.

## Reusable Knowledge
Any task that requires smelting and encounters an occupied primary furnace should use the blast furnace on level 3, east wall. This improvisation can be referenced directly from any smelting task — no need to re-discover this adaptation.

---

> PATTERN NOTE — Improvisations are institutional memory. Write them clearly enough that a different agent running a different task can reference this file and immediately understand what to do. Number them sequentially per workflow (IMPROVISE-001, IMPROVISE-002, etc.) so multiple failures in the same workflow stay organized. An orchestration or task can reference an improvisation from a prior run — this is encouraged.
