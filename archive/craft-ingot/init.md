# Execution Workflow — Entry Point

You are an agent. You have been given access to these files and a task to perform. This file is your entry point. Read it fully before doing anything else.

## What This Is

This repo is a workflow scaffold. The operator will explain a task to you and instruct you to implement it using this system. You will have access to the relevant source code and other resources outside this repo. These files define the patterns you must follow — not the specific work, just the structure for doing work.

Your job is to determine what the init steps are for the workflow you have been assigned. They are not prescribed here. You decide, based on the task and the patterns you see.

## How The Operator Will Use This

The operator will point you at this repo, explain the task, and say something like "implement this using the workflow." You then:

1. Read all files in this repo to understand the pattern
2. Determine the init steps for the assigned task
3. Create the appropriate files (orchestration, tasks, tests) following the examples
4. Execute the workflow
5. Write a result when finished — success or failure

You are not given a prescriptive checklist. You are given a pattern. Follow it.

## Folder Structure

**Files at the workflow root** (not folders):

| File | Purpose |
|---|---|
| `init.md` | The agent entry point — what the workflow is, what to read before starting |
| `branches.md` | The repo manifest. Every repo this workflow modifies, with the branch name. **The orchestrator MUST branch every repo listed here before starting Task 0001.** |

| Folder | Purpose |
|---|---|
| `Orchestrate/` | High-level orchestration workflows. One file per workflow. Can embed `/loop` for autonomous execution. References task IDs. Multiple orchestrations can exist. |
| `Tasks/` | Discrete implementation units. Numbered files. Each task is one atomic piece of work. The agent is responsible for implementing it. The very first task (`0000-...`) is always "branch every repo per branches.md". |
| `Test/` | Validation files keyed to tasks. Defines how to confirm a task succeeded. Run after each task. |
| `Improvise/` | When something fails, write an improvisation here before retrying. Document what failed, what was adapted, and what the outcome was. Tasks and orchestrations can reference improvise files directly. |
| `Results/` | Written at the end of every task or orchestration — success or failure. Always write a result. This is the record of what happened. |
| `SelfImprove/` | Written *after* the Result. Captures what tools, MCPs, docs, or access would have made the run faster or more correct. Operator feedback, not a complaint log. Always write one. |

## Numbering Convention

Files are numbered with a four-digit ID and a descriptive slug:

```
0001-TASK-NAME-TASK.md
0001-TASK-NAME-TEST.md
0001-TASK-NAME-ORCH.md
0001-TASK-NAME-IMPROVISE-001.md
0001-TASK-NAME-RESULT.md
0001-TASK-NAME-WISHLIST-001.md
```

The same ID links the task, test, orchestration, improvisation, and result together. When you create a new workflow, pick the next available number.

## Your Responsibilities

- Read all existing files before creating new ones — do not duplicate
- Follow the patterns in the example files exactly
- **Branch every repo in `branches.md` BEFORE any code work** (Task 0000). Skipping this is a process failure.
- If a task fails, write an improvisation before retrying — do not silently retry
- Always run the test file after completing a task
- Always write a result at the end, regardless of outcome
- After the result, always write a SelfImprove wishlist — name the MCPs / docs / access you wished you had
- If an improvisation from a prior run is relevant, reference it — do not re-learn what is already documented

## What The Example Files Are For

The `CRAFT-INGOT` examples in each folder are not real tasks. They are instructional. They show you the structure, the language, the cross-references, and the level of detail expected. When you implement a real task, your files should mirror this pattern.
