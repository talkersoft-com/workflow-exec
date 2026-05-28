# hv_ship: await_merge infinite loop + open_browser flag

## What this workflow does

Implements two changes to `hive-deck-pro`:
1. Removes the two-phase `continue_transaction` approach and restores `await_merge` to a single infinite polling loop that blocks until all PRs are merged
2. Adds an `open_browser` config flag under `ship:` that opens a browser tab for each PR URL immediately after creation

## Read before starting
- `deck.md` — which deck and repos are in scope
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager/steamy-kangaroo/PLAN.md` — full design

## Constraints
- Do not add a timeout to the polling loop — it must run indefinitely
- `open_browser` is best-effort — errors opening the browser must be silently ignored
- After any Go or TypeScript change: rebuild binary (`go install`) and MCP dist (`npm run build`), then instruct user to restart MCP via `/mcp`
