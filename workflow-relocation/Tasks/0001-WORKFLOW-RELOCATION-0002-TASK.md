# Task 0002 — Update `workflowCmd` to resolve repo from workflow definition

## Goal
Replace the `WorkflowFolder` deck-field lookup with a repo-based lookup: read `WorkflowDef.Repo`, find it in the resolved plan, use its on-disk path as `{{WorkflowFolder}}`. Fail clearly if the repo is not in the deck.

## Target repo(s)
`hive-deck-pro`

## Steps
1. Open `cmd/hv/main.go`, find `workflowCmd`
2. Remove the code that reads `l.WorkflowFolder` from the deck
3. After loading the workflow definition, read `wfDef.Repo` (e.g. `"talkersoft-com/execution-workflow"`)
4. Build the resolved plan via `resolve.Build(l)` (already done earlier in the function or replicate the call)
5. Search `plan.Nodes` (or `plan.Repos`) for the repo whose `Name` matches `wfDef.Repo`
6. If not found: print `workflow "<name>" requires repo "<repo>" which is not in this deck` and exit non-zero
7. Use the found repo's on-disk path as the value for `{{WorkflowFolder}}` in token replacement
8. Run `go build ./...` — fix any compile errors

## Definition of done
`go build ./...` passes. Running `hv workflow cloud-manager` resolves `{{WorkflowFolder}}` to the on-disk path of `talkersoft-com/execution-workflow`. Running `hv workflow` on a deck that does not contain the workflow repo prints the expected error and exits non-zero.
