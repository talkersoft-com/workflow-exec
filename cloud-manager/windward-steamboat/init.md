# Ansible Epic planning factory (windward-steamboat)

## What this workflow does
Authors the nine-plan Ansible Epic series: ANSIBLE-EPIC-CONTRACTS.md first, then plans
ansible-p1-inventory … ansible-p9-library (dependency-ordered, each independently reviewable and
executable under its named workflow; P6 under vorch@cloud-manager), ANSIBLE-EPIC-LEDGER.md, and a
mandatory cross-consistency pass with an epic-coverage map. Ships everything as ONE plan-stage PR
and STOPS — the human reviews and approves the nine; this run never starts any of them.

## Read before starting
- `deck.md`, `../../../workflow-plans/cloud-manager/windward-steamboat/PLAN.md`
- THE EPIC: /home/todd/workspace/hive-deck-pro/planning/workflow-plans/master-plans/ANSIBLE-EPIC.md
- Mandatory current-state research BEFORE writing anything: entities
  (cloud-manager-api/src/Models/CloudManager.Entities), the MCP tool surface (cloud-manager-mcp),
  web pages (cloud-manager-web), porch capabilities (vorch-service/cmd/porch) — the plans must
  build on main as it exists (marketplace/blueprints shipped) and may not contradict it.

## Constraints
- DOCUMENTS ONLY — the only repo that changes is workflow-plans (plus this exec folder runtime).
  Zero diffs in any code repo.
- Contracts before plans; every plan cites contracts consumed/exposed; the consistency pass fixes
  drift in place before shipping.
- Each plan: standard shape (Objective/Background/Design with before-after contract examples/
  proportional phases/own open questions), names its workflow ref, names its dependencies.
- Ship stage "plan" → PR stays open (await_merge). Announce the PR link and STOP.
