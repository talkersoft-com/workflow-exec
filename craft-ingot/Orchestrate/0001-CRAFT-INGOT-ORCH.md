# Orchestration: Craft Ingot

> EXAMPLE FILE — This shows the pattern for an orchestration workflow. When you implement a real task, create a new file in this folder following this structure. Do not modify this file.

## Workflow ID
`0001-CRAFT-INGOT`

## Objective
Smelt raw iron ore into a usable iron ingot using the furnace.

## Tasks
- [ ] `Tasks/0001-CRAFT-INGOT-TASK.md` — Prepare furnace and smelt the ore

## Orchestration Steps

1. Read all files referenced below before starting
2. Execute each task in order, top to bottom
3. After each task, run the corresponding test
4. If a task fails, write an improvisation in `Improvise/` and adapt before retrying
5. Write a result in `Results/` when the workflow completes — success or failure

## Autonomous Execution

If running this workflow autonomously, embed the loop directive below. The agent runs `/loop` and continues until all task checkboxes are checked or an unrecoverable error occurs.

```
/loop Continue executing tasks in Orchestrate/0001-CRAFT-INGOT-ORCH.md. After each task, run the corresponding test. If a test fails, write an improvisation and retry. Check off tasks as they complete. Stop when all tasks are complete or a failure cannot be recovered. Write a result when done.
```

## Improvisation Policy

If a step fails, do not silently retry. Write an improvisation in `Improvise/` first, document the adaptation, then continue. If a prior improvisation exists that covers this failure mode, reference it instead of writing a new one.

## References

- Tasks: `Tasks/0001-CRAFT-INGOT-TASK.md`
- Tests: `Test/0001-CRAFT-INGOT-TEST.md`
- Improvise: `Improvise/` (create here on failure)
- Results: `Results/` (write here when done)

---

> PATTERN NOTE — An orchestration can reference multiple tasks. Tasks do not have to be sequential; some can run in parallel if the orchestration says so. You can have multiple orchestration files for different workflows — they can share tasks and improvisations. The orchestration is the high-level plan; the task files are the implementation detail.
