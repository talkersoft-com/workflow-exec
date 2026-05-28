# Feature Flag Health Endpoint — Close the Probe Gap

## What this workflow does

Wires up the missing health-write path so that vorch-service's ansible-runner health probe can persist its `healthy` + `last_probed_at` result back to the API. Today the probe fires every 5 minutes and silently 404s on every tick. After this workflow the three-state feature-flag model (disabled / enabled-but-unhealthy / enabled-and-healthy) works end-to-end.

## Read before starting

- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/ORCH.md` — full task list and /loop directive
- `/Users/talker/workspace/cloud-manager/planning/workflow-plans/cloud-manager-ansible/hazy-kite/PLAN.md` — full design with exact code changes

## Constraints

- No migration required — `Healthy` and `LastProbedAt` columns already exist on `feature_flags`
- `Enabled` in the PATCH request must become nullable so health-only calls from the probe do not flip the enabled state
- The existing `/api/v1/featureflags/{key}` route must continue to work unchanged for the web UI toggle
