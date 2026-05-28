# pr_mode Enum — Replace auto_merge + require_merged_pr

## What this workflow does

Replaces `auto_merge: bool` and `require_merged_pr: bool` in the Ship config with a single `pr_mode` enum (`auto_merge` | `await_merge` | `manual`). Adds backward-compatible YAML unmarshalling so old configs keep working. Adds `RunAwaitMerge` polling op and a new `hv_await_merge` MCP tool with a background-task descriptor. Updates `config.yaml` on both Mac and server.

## Read before starting

- `deck.md` — repos in scope, branch, hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/toasty-brownie/PLAN.md` — full design

## Constraints

- `auto_merge: true` in existing configs must continue to work after the change (backward compat via UnmarshalYAML)
- `pr_mode: auto_merge` must produce identical output to current `auto_merge: true` behaviour
- No changes to `delete_branch_on_merge`, `teardown_on_ship`, or `auto_default_branch` semantics
- All existing tests must pass at every phase
