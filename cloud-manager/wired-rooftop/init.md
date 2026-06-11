# Blueprint composer navigation (wired-rooftop)

## What this workflow does
Two navigation affordances in cloud-manager-web: an Edit action on every /blueprints list row
opening the composer (/blueprints/{id}), and create-modal success navigating directly into the
new blueprint's composer. Nothing else changes.

## Read before starting
- `deck.md` — scope and hv MCP calls
- `Execution/Exec.md` — task list
- `../../../workflow-plans/cloud-manager/wired-rooftop/PLAN.md`

## Constraints
- cloud-manager-web ONLY; follow the existing VMs/Playbooks list→detail pattern; no new
  dependencies; marketplace flag gating untouched; scope strictly the two affordances.
- Build (--target web) per phase; deploy web before ship; playwright verification on the
  deployed app; proportional tests (this is a small fix — keep it small).
