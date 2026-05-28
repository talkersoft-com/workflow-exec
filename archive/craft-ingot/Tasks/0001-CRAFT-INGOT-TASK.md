# Task: Craft Ingot

> EXAMPLE FILE — This shows the pattern for a task file. When you are assigned work, create a new task file in this folder following this structure. Do not modify this file.

## Task ID
`0001-CRAFT-INGOT-TASK`

## Parent Orchestration
`Orchestrate/0001-CRAFT-INGOT-ORCH.md`

## Status
- [ ] In progress
- [ ] Complete
- [ ] Failed

## Objective
Smelt raw iron ore into a usable iron ingot using the furnace.

## Context
You have access to the source code and resources relevant to this task outside this repo. Use them. The task description tells you what to accomplish — it is your responsibility to figure out how, using whatever tools and code are available to you.

Do not ask for clarification. Read the codebase, understand the system, and implement.

## Steps
1. Locate the furnace in the workspace
2. Verify raw iron ore is available in inventory
3. Load ore into the furnace input slot
4. Provide fuel (coal preferred; wood acceptable)
5. Wait for smelting cycle to complete
6. Collect iron ingot from the output slot
7. Verify inventory reflects the ingot

## Acceptance Criteria
- Iron ingot is present in inventory
- No raw ore was lost without a corresponding ingot
- Furnace is empty after completion

## Test
After completing the steps above, run `Test/0001-CRAFT-INGOT-TEST.md` to validate. Do not mark this task complete until the test passes.

## On Failure
Write an improvisation in `Improvise/` before retrying. Check `Improvise/` first — if a prior improvisation covers this failure mode, reference it instead of writing a new one.

---

> PATTERN NOTE — Tasks are atomic. One task does one thing. If the work is too large for one task, break it into multiple numbered tasks and reference them from the orchestration. Keep task files focused — the steps should be specific enough that another agent could pick up mid-task and continue.
