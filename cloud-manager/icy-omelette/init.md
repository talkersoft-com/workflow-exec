# hv_workflow — plan_path to plan_paths array

## What this workflow does

Replaces the single `plan_path` string parameter on the `hv_workflow` MCP tool with `plan_paths`, an array of absolute file paths. All plans are read and prepended to the workflow output in order, each separated by a `---` divider, before the workflow instructions. Change is purely in the MCP TypeScript layer — Go CLI unchanged.

## Read before starting

- `deck.md` — repos in scope, branch, hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/thorny-catapult/PLAN.md` — full design

## Constraints

- The change is in `hive-deck-pro`, not `cloud-manager` repos — workflow-exec is the planning output, code changes go to `hive-deck-pro`
- Must rebuild MCP (`npm run build`) after TypeScript change
- Must reinstall `hv` binary after any Go changes (none expected here)
