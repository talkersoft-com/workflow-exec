# Workflow Relocation

## What this workflow does
Refactors the hive-deck-pro workflow system so that the workflow definition owns the repo link rather than the deck yaml. Currently the deck yaml carries both `workflow:` and `workflow_folder:` fields; after this change the deck only declares `workflow: <name>` and the workflow definition in `workflows.yaml` carries the `repo:` field. The `workflowCmd` is updated to look up the repo from the workflow definition and validate it is present in the deck before resolving the on-disk path for `{{WorkflowFolder}}`.

## Read before starting
- `deck.md` — which deck and repos are in scope; pre-written hv MCP calls
- `Orchestrate/0001-WORKFLOW-RELOCATION-ORCH.md` — full task list and /loop directive
- `../../common/toolkit/hive-deck.md` — hv MCP call patterns

## Constraints
- Do not break `hv workflow` for any deck that already has a valid `workflow:` field
- All changes must compile cleanly — `go build ./...` must pass after every task
- The `workflow_folder` field must be removed cleanly with no backwards-compatibility shim
