# workflow-configuration Repo — HV_HOME Source Control

## What this workflow does

Creates a `workflow-configuration` GitHub repo inside the cloud-manager deck that mirrors `~/.hv/` (config.yaml, decks/, mcps.yaml, modules.yaml). Adjusts `config.LoadSetup()` so `$HV_HOME` is checked directly (not with `.hv/` appended). Adds `make configure` which writes `~/.hv/.env` with `HV_HOME` pointing at the repo. After this, all hive-deck config lives in source control.

## Read before starting

- `deck.md` — repos in scope, branch, hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/silver-pigeon/PLAN.md` — full design

## Constraints

- The `$HOME/.hv/` fallback must continue to work for machines without `HV_HOME`
- The `workflow-configuration` repo name must be readable (not `.hv`)
- `make install` behavior unchanged — `make configure` is a separate optional target
- Migration is additive: existing `~/.hv/` contents are copied, not deleted
